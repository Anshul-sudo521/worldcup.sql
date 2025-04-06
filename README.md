# worldcup.sql
World Cup Database

(insert_data.sh)

#!/bin/bash

if [[ $1 == "test" ]]
then
  PSQL="psql --username=postgres --dbname=worldcuptest -t --no-align -c"
else
  PSQL="psql --username=freecodecamp --dbname=worldcup -t --no-align -c"
fi

# Do not change code above this line. Use the PSQL variable above to query your database.

# Clear existing data
$PSQL "TRUNCATE TABLE games, teams;"

# Insert unique teams from games.csv
echo "Inserting teams..."
tail -n +2 games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
  # Insert winner if not exists
  $PSQL "INSERT INTO teams(name) SELECT '$WINNER' WHERE NOT EXISTS (SELECT 1 FROM teams WHERE name='$WINNER');"
  # Insert opponent if not exists
  $PSQL "INSERT INTO teams(name) SELECT '$OPPONENT' WHERE NOT EXISTS (SELECT 1 FROM teams WHERE name='$OPPONENT');"
done

# Insert games
echo "Inserting games..."
tail -n +2 games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
  # Get winner_id and opponent_id
  WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER';")
  OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT';")
  # Insert game row
  $PSQL "INSERT INTO games(year, round, winner_id, opponent_id, winner_goals, opponent_goals) 
         VALUES($YEAR, '$ROUND', $WINNER_ID, $OPPONENT_ID, $WINNER_GOALS, $OPPONENT_GOALS);"
done

echo "Data insertion complete!"


(queries.sh)

#!/bin/bash

PSQL="psql --username=freecodecamp --dbname=worldcup --no-align --tuples-only -c"

# Do not change code above this line. Use the PSQL variable above to query your database.

echo -e "\nTotal number of goals in all games from winning teams:"
echo "$($PSQL "SELECT SUM(winner_goals) FROM games")"

echo -e "\nTotal number of goals in all games from both teams combined:"
echo "$($PSQL "SELECT SUM(winner_goals + opponent_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams:"
echo "$($PSQL "SELECT AVG(winner_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams rounded to two decimal places:"
echo "$($PSQL "SELECT ROUND(AVG(winner_goals), 2) FROM games")"

echo -e "\nAverage number of goals in all games from both teams:"
echo "$($PSQL "SELECT AVG(winner_goals + opponent_goals) FROM games")"

echo -e "\nMost goals scored in a single game by one team:"
echo "$($PSQL "SELECT MAX(winner_goals) FROM games")"

echo -e "\nNumber of games where the winning team scored more than two goals:"
echo "$($PSQL "SELECT COUNT(*) FROM games WHERE winner_goals > 2")"

echo -e "\nWinner of the 2018 tournament team name:"
echo "$($PSQL "SELECT name FROM teams WHERE team_id = (SELECT winner_id FROM games WHERE year=2018 AND round='Final')")"

echo -e "\nList of teams who played in the 2014 'Eighth-Final' round:"
echo "$($PSQL "SELECT DISTINCT name FROM teams JOIN games ON teams.team_id = games.winner_id OR teams.team_id = games.opponent_id WHERE year=2014 AND round='Eighth-Final' ORDER BY name")"

echo -e "\nList of unique winning team names in the whole data set:"
echo "$($PSQL "SELECT DISTINCT name FROM teams JOIN games ON teams.team_id = games.winner_id ORDER BY name")"

echo -e "\nYear and team name of all the champions:"
echo "$($PSQL "SELECT year, name FROM games JOIN teams ON games.winner_id = teams.team_id WHERE round='Final' ORDER BY year")"

echo -e "\nList of teams that start with 'Co':"
echo "$($PSQL "SELECT name FROM teams WHERE name LIKE 'Co%' ORDER BY name")"

(expected_output.txt)


Total number of goals in all games from winning teams:
68

Total number of goals in all games from both teams combined:
90

Average number of goals in all games from the winning teams:
2.1250000000000000

Average number of goals in all games from the winning teams rounded to two decimal places:
2.13

Average number of goals in all games from both teams:
2.8125000000000000

Most goals scored in a single game by one team:
7

Number of games where the winning team scored more than two goals:
6

Winner of the 2018 tournament team name:
France

List of teams who played in the 2014 'Eighth-Final' round:
Algeria
Argentina
Belgium
Brazil
Chile
Colombia
Costa Rica
France
Germany
Greece
Mexico
Netherlands
Nigeria
Switzerland
United States
Uruguay

List of unique winning team names in the whole data set:
Argentina
Belgium
Brazil
Colombia
Costa Rica
Croatia
England
France
Germany
Netherlands
Russia
Sweden
Uruguay

Year and team name of all the champions:
2014|Germany
2018|France

List of teams that start with 'Co':
Colombia
Costa Rica
