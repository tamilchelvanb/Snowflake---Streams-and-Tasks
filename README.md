# Snowflake---Snowpipe, Streams and Tasks
This will capture the steps that will automate the injestion of cloudtrail log files from AWS S3 to a Snowflake relational table.

##Overview of what was done:
    1. The AWS log files from Cloud trail are stored in S3 in JSON Format
    2. Using Snowflake Snowpipe, the data from S3 are moved into a table with Variant datatype
    3. The json data was transformed from the array form and the required information are saved into a relational table. 
        - The new inserts from the snowpipe are captured through streams.
        - Using Tasks the data in streams are scheduled/automated to be loaded the target tables.

