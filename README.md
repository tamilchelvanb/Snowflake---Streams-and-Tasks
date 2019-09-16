# Snowflake- Streams and Tasks
This will capture the steps that will automate the injestion of cloudtrail log files from AWS S3 to a Snowflake relational table.

## Steps Overview:
    1. Cloud trail logs from AWS, stored in S3 in JSON Format are moved to a table with variant data type using snowflake snowpipe
    2. The json data was transformed from the array form and the required information are saved into a relational table
        - The new inserts from the snowpipe are captured through streams
        - Using Tasks the data in streams are scheduled/automated to be loaded the target tables

## Prerequisites:
>You have a AWS account and cloud trail logs are setup 
