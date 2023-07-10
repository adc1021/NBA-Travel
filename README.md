# NBA Travel

## 1. Introduction
Each year, teams in professional sports see scores of athletes suffer injuries both minor and major. After creating a predictive model to try and determine the leading causes of injuries in the NBA, I was fascinated by the effect travel schedules may have on player health. In this Tableau visualization, I display the key differences in travel between the bottom and top 5 teams in the league, in regards to overall travel schedule. 

The questions I wanted to address were as follows: 
1. Do coastal teams have more difficult travel schedules?
2. Does geography and travel play any role in team outcomes, based on record?
3. As my predictive model suggests, does the total amount of miles travelled per team affect the number of injuries a team suffers?

## 2. Data Cleaning
The raw data for this project was taken from the airball R package which pulls data from __www.nba.com/data__ 

First, I used the following R script to pull data for every team in the league. The function below is written in R and pulls data with the help of the airball package and then immediately adds it directly to a <ins>PostgreSQL<ins> database: 

```R

addDelay <- function() {
  Sys.sleep(15)  # Adjust the delay time as needed
}

western_teams <- c("Dallas Mavericks", "Houston Rockets", "Memphis Grizzlies", "New Orleans Pelicans", "San Antonio Spurs", "Denver Nuggets", "Minnesota Timberwolves", "Oklahoma City Thunder", "Portland Trail Blazers", "Utah Jazz", "Golden State Warriors", "Los Angeles Clippers", "Los Angeles Lakers", "Phoenix Suns", "Sacramento Kings")
eastern_teams <- c("Boston Celtics", "Brooklyn Nets", "New York Knicks", "Philadelphia 76ers", "Toronto Raptors", "Chicago Bulls", "Cleveland Cavaliers", "Detroit Pistons", "Indiana Pacers", "Milwaukee Bucks", "Atlanta Hawks", "Charlotte Hornets", "Miami Heat", "Orlando Magic", "Washington Wizards")

teams <- c(western_teams, eastern_teams)

con <- dbConnect(PostgreSQL(),
                 host = "localhost",
                 port = 5432,
                 dbname = "NBA_Travel",
                 user = "antho",
                 password = "Himynameis15!")

team_data <- data.frame()
  # Add delay before each request
  addDelay()

  # Retrieve data for each team
  team_data_team <- nba_travel(start_season = 2022,
                               end_season = 2022,
                               team = teams,
                               return_home = 3,
                               phase = "RS",
                               flight_speed = 550)[, c("Team", "Latitude", "Longitude", "d.Latitude", "d.Longitude", "Distance", "Route", "Rest", "Flight Time", "W/L", "Return Home", "Shift (hrs)")]

  # Append the team data to the overall data frame
  team_data <- rbind(team_data, team_data_team)


dbWriteTable(con, "records", team_data)
dbDisconnect(con)

```

After the data was stored, the table in which it was stored required cleaning. 

To do so, I used the following queries: 

a. Fixing row names to remove quotation marks: 
```SQL
ALTER TABLE records
RENAME COLUMN "Latitude" to latitude;
ALTER TABLE records
RENAME COLUMN "Longitude" to longitude;
ALTER TABLE records
RENAME COLUMN "d.Latitude" to d_latitude;
ALTER TABLE records
RENAME COLUMN "d.Longitude" to d_longitude;
# etc...
```

b. The Flight Time field has string values instead of integers, and required the removal of extra characters
```SQL
UPDATE records
SET flight_time  =
  CASE
    WHEN flight_time = '-' THEN '0'
    WHEN flight_time LIKE '%hours' THEN REPLACE(REPLACE("Flight Time", '~', ''), ' hours', '')::numeric
	WHEN flight_time LIKE '%minutes' THEN REPLACE(REPLACE("Flight Time", '~', ''), ' minutes', '')::numeric
END
```

c. Flight Time was also not consist with it's values for flights that lasted a few hours or less than an hour. To resolve this I altered the field using this query: 
```SQL
UPDATE records
SET flight_time =
	CASE
		WHEN flight_time < 10 THEN ROUND((flight_time * 60)::numeric, 0)
		ELSE flight_time
	END
```
d. Then I had to ensure the entire row was the correct data type: 

```SQL
ALTER TABLE records
ALTER COLUMN flight_time TYPE FLOAT
USING flight_time::FLOAT;
```


e. After performing a few other queries I used the following query to move the data into a csv format for visualization: 
```SQL
COPY records (flight_id, team, latitude, longitude, d_latitude, d_longitude, distance, route, rest, flight_time)
TO '/Users/antho/Documents/knicks_2022.csv'
WITH (FORMAT CSV, HEADER);
```
