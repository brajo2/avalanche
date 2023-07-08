# avalanche

## Data Engineer Applicant Project 2023 – Colorado Avalanche
The below questions are intended to give you a platform to stand out among a number of very qualified candidates applying for this position.

### Part 1
The first series of questions revolve around a sample data provider, Elite Prospects. Elite Prospects is one of the most useful and most utilized data providers in the hockey world, with player pages for just about any hockey player at any level you could want world-wide, amongst other functions. They have an API that can provide this data for team use. You can see the documentation for the API here.
For the first part of this project, we’d like you to imagine that you’re being tasked with reading data from this API into a Colorado Avalanche database. Diagrams, screenshots, and code snippets are encouraged where applicable. Please answer the following questions:
#### 1. What are some examples of tables and specific data from this API that you believe would be particularly important for the Avalanche’s analytics department to possess?
   - player id tables with player name, position, handedness, height, weight, birthdate, birthplace--with some universal identification would be great
   - If the player has been drafted already then draft year, draft round, draft pick, draft team, and current team would be great additional information to have
   - Other tables that are geared towards measuring
     - raw player stats: goals, assists, shots, etc.
     - advanced player stats: (perhaps RAPM (regularized adjusted plus-minus), if available for hockey), WAR, on/off ice stats, with or without you stats, etc.
     - team stats: goals for, goals against, record, etc.
     - stats that are geared towards measuring the quality of a player's competition & teammates
     - contract tables with player salary, contract length, etc.
     - tables with player's injury history (!) would be interesting because with the right data, you could build a model to predict injury risk (which is why it'd be important to have tables with player height, weight, handedness, etc.)
     - player tracking tables if available would be great, to be able to measure things like speed & acceleration.
     - Also hockey's played internationally, so it'd be great to have tables with international stats, perhaps U18, U20, Olympics, etc.
     - And if we can, multilingual information could be useful as well. I am not sure if Elite Prospects API has this but it'd be great to have tables with player names in different languages, and perhaps even tables with player bios in different languages. If there is play-by-play data returned by the API, then we could create procedures to parse out the proper text to create advanced boxscores that give insight into on/off stats and player events that aren't just goals.
     - It may also be important to have LIVE data for the sake of dashboards and perhaps real-time strategical decision support. 

#### 2. How would you go about the process of reading in and storing this information, as well as updating it on a regular basis? Schemas are encouraged.
   - Here's an idea of how I would store this information:
     - Depending upon the country that the prospect data is centric to, I would have different schemas like `country` and `league` etc.
   ```
    CREATE SCHEMA IF NOT EXISTS country;
    CREATE SCHEMA IF NOT EXISTS league;
    CREATE SCHEMA IF NOT EXISTS player;


   ```
     - I'd have a set of more central schemas for metadata like `player` to store metadata that is universal to all prospects
       - these tables can contain play-by-play data relevant to the league / circuit in which the games were played
       - `player` could contain metadata, but also stats that are universal to all prospects for easy querying
     - Leagues could have their own schemas, because certain data may need to be handled differently based on language, rule settings, etc.


#### 3. What are some logical ways you would go about formatting this data in tables to make it as efficient and easy as possible for a Data Scientist to query it?
   - I think denormalized tables would be best for querying, because it would be easier for data scientists to join tables together to get the data that they want.
   - During the data ingestion & transformation process, keeping tables smaller can help for the sake of faster automatic querying and resource management, but I think that end-users would prefer to have easily queryable tables / datamarts.
   - Conceptually even within the advertisement technology industry, it's common place to have datamarts that are denormalized, but with all the valuable secondary / derived metrics easily accessible.


#### 4. What are some of the ways that you can ensure that any data errors that stem from the ETL process are caught and fixed in a timely and effective fashion?
   - Running validations over data that's processed during the pipeline can help catch errors early on, and depending upon the infrastructure, it may even be possible to use Slack / email notifications to alert data engineers of errors (or end users).
     - For example, if a player's height is 0, then that's a clear error that can be caught early. We can write Python scripts to validate data ingested, or we can write SQL scripts to validate data ingested. And then execute the necessary logic to notify the appropriate parties and fix the data.
   -  Also writing testing (generally use pytest module) for the ETLs can help us catch errors during CICD, which can prevent us from productionizing code that would create those errors for downstream.
-----------------------
### Part 2
The following are more general data engineering questions:
#### 1. What is the difference between a transaction-based system and an analytics-based system within the ETL process?
  - A transaction-based system is more geared towards handling data from ingestion to storage, focused on processing speed and good data quality (i.e. no unforeseen duplicates, no unforeseen nulls, etc., no unexpected data loss, etc.)
  - An analytics-based system is more geared towards handling data from storage to consumption, ensuring we aren't creating SQL that is too slow, but also doesn't duplicate entries, doesn't create nulls where there shouldn't be so that we can create accurate reports.
    - For example, I've been creating analytics-based ETLs for interrupting play-by-play data for the purpose of teasing out possession data. I've largely been relying upon SQL query logic to interpret the "rules" for when a possession starts and ends, and I've been using Python to execute the SQL queries w/ Jinja templating, but I've come across some issues with the CTE logic allowing for ONE possession to include two offensive teams, which isn't possible. So I've had to go back and rework the SQL logic to ensure that I'm not creating duplicate entries for possessions. But this is an ETL that is interested in creating data digestible for a ridge regression model, so rather than being as concerned about data ingestion & storing, I'm more concerned about creating accurate data for the model.


#### 2. How would you go about deciding which to use when building infrastructure? Is one more appropriate than the other for handling the type of data and processes that a hockey analytics department requires?
    - As mentioned above, if the goal / current objective is to build out an infrastructure of housing a ton of prospect data, then I think a transaction-based system would be more appropriate. Questions like "what's the most adequate format to ingest and store this data, such that this ETL can be run timely, sustainably, and if it needs to be iterated upon, easily understandable to the next person?" would be important to ask here.
    - But if the goal is to build out data for analytics consumption & model creation, then the other would be more appropriate for focus upon. Making great ETLs that are templatizable, can calculate derived metrics appropriately such that they can be used for model creation becomes priority. Then storing that data in a way that is easily queryable by data analysts and data scientists is important.
  
#### 3. What is the difference between structured and unstructured data?
    - Structured data is much like a spreadsheet, where each row is a record and each column is a field. It's easy to read, ingest and store, like a CSV, TXT, or EXCEL file.
    - Unstructured data is much like a blob of text, where it's not easy to read, ingest and store, like a JSON file or XML file. There's methodology to parse JSON with Python, Postgres, Snowflake, etc., but it's not quite as easy as reading a CSV file.
    - For the sake of data scientists, analysts, end-users, if someone was to ask for a raw data dump, it'd be much easier to give them a CSV file than a JSON file, because they can easily read the CSV file in Excel. 

#### 4. What are the four/five Vs of big data and why are they important?
   - Volume
     - How much data is being processed? Because that will determine whether we may need to use a larger data warehouse like Snowflake, or if we can get away with using a smaller data warehouse like Postgres. It'd also determine whether we ought to write ETLs in SQL to create our derived data, or if we can use Python. And within Python, should we be using Pandas or PySpark, etc.? Should we be storing data within AWS S3?
   - Velocity
      - This is so important because it determines how quickly we need to process data. 
      - Are we processing real-time data? Do we need to update a live dashboard with data we're getting in a few seconds before?
      - Are we trying to run our analytics-based ETLs before a certain time in the morning for DS/DA consumption? 
      - Do we need to process data before the start of a game?
      - The more data we can process in a shorter amount of time, the more we can do with that data.
   - Variety
      - We discussed structured vs unstructured data above, but besides text there's possibility that we need to store video, images or audio. 
      - that gives more creative control to data scientists and software engineers to create more interesting models that could even provide competitive advantage. Play-by-play alone cannot completely tell the story with regards to a player's value above replacement or assessing a prospect's feel for the game, but storing video data could be helpful for that.
   - Veracity
       - This is exactly why validations and testing is so important. If downstream data is incorrect and that data is used for modeling, then the model loses some of its value, from a predictive and descriptive perspective.
   - Value
#### 5. What is a common mistake that rookie data engineers make that you’ve learned to avoid?
   - Being too one-dimensional with the way that an ETL is written.
     - Writing an ETL so that it only serves one purpose, even if it were a scalable concept, so that the code could be reusable for a different configuration of data.
   - Not writing enough tests for the ETLs.
     - I've been guilty of this, but I've learned to think about potential issues that can arise within ETLs, as it's easy to miss something that could cause an error downstream.
   - Writing code that is too obscure and not providing documentation.
     - It's cool to use a lot of Py libraries, or make something complicated, but it's better to be able to remember what you did and why you did it, and to be able to explain it to someone else.


### Part 3
The following are hockey and data science related questions:
#### 1. What would some of the issues be with using exclusively player point totals to determine who the best hockey players in the NHL are?
    - This is a question pervasive among all sports, I think. In basketball, raw boxscore stats and old school metrics like PER (player efficiency rating) had been used historically to evaluate, but those metrics are not engineered to tease out the individual value of a player, or the potential value for a player within a different team context.
    - In football, newer school metrics like EPA (expected points added) and CPOE (completion percentage over expected) have been used, but historically yards, touchdowns, interceptions have been strongly clung to.
    - Baseball might be the only sport that has been able to transition.
    - On / off stints become important to tease out player value, and point totals are variable & subject to randomness. So if we are to take a results-based approach to predictive modeling, then we'd struggle to create a reliable model for new situations.

#### 2. How would you go about creating an NHL aging curve for players? What would your critiques be of your own method and what are some of the challenges that come with this task?
   - An aging curve model may be somewhat out of my main wheelhouse, but a model that includes dependent variables like age, age^2, age^3, age^4, etc. could be used to predict a player's WAR (wins above replacement) for the next season.
   - A polynomial regression that is trained understanding the general rise and descent of a player's career could be used to predict a player's WAR for the next season.
   - Also more recently, in plus-minus statistics in the basketball community, exponential decay has been a concept important to future predictions as well. 
   - It'd also be important to remember that as player age, they leave the league, therefore some of the observations we'd need to be using would come from players that are no longer in the league. So we'd need to be careful about how we handle that. We'd probably need to pad the observational data with some sort of replacement level player.