# NBA Travel

### Introduction
Each year, teams in professional sports see scores of athletes suffer injuries both minor and major. After creating a predictive model to try and determine the leading causes of injuries in the NBA, I was fascinated by the effect travel schedules may have on player health. In this Tableau visualization, I display the key differences in travel between the bottom and top 5 teams in the league, in regards to overall travel schedule. 

The questions I wanted to address were as follows: 
1. Do coastal teams have more difficult travel schedules?
2. Does geography and travel play any role in team outcomes, based on record?
3. As my predictive model suggests, does the total amount of miles travelled per team affect the number of injuries a team suffers?

### Data Cleaning
The raw data for this project was taken from the airball R package which pulls data from __www.nba.com/data__ 

First, I used the following R script to pull data for every team in the league: 
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
