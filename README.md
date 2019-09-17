# Snowflake- Streams and Tasks
>This will capture the steps that will automate the injestion of cloudtrail log files from AWS S3 to a Snowflake relational table.


## Steps Overview:
    1. Cloud trail logs from AWS, stored in S3 in JSON Format are moved to a table with variant data type using snowflake snowpipe
    2. The json data was transformed from the array form and the required information are saved into a relational table
        - The new inserts from the snowpipe are captured through streams
        - Using Tasks the data in streams are scheduled/automated to be loaded the target tables


## Prerequisites:
>1. You have a AWS account and cloud trail logs of your account are setup to S3 Bucket.
>2. Configure an AWS IAM role with the required policies and permissions to access your external S3 bucket. This approach allows individual users to avoid providing and managing security credentials and access keys.
https://docs.snowflake.net/manuals/user-guide/data-load-s3-config.html
>3. Create and Automate the snowpipe to create event notifications.
https://docs.snowflake.net/manuals/user-guide/data-load-snowpipe-auto-s3.html#option-1-creating-a-new-s3-event-notification-to-automate-snowpipe

*At this stage, we should have the data logs created from Cloud Trail flowing in to the S3 bucket. The S3 data is consumed by Snowflake Snowpipe using the AWS role through SQS event notification. There should be a continuous flow of these json files into the variant table created as the logs are getting generated. We will now focus on pushing the log files to a structured format using Streams and automate this flow through Tasks.*


# Streams
A stream is a new Snowflake object type that provides change data capture (CDC) capabilities to track the delta of changes in a table, including inserts and data manipulation language (DML) changes, so action can be taken using the changed data. A table stream allows you to query a table and consume a set of changes to a table, at the row level, between two transactional points in time.


# Task
A task is a new Snowflake object type that defines a recurring schedule , including statements that call stored procedures. You can chain tasks together for successive execution to support more-complex, periodic processing.

In a continuous data pipeline, tasks may optionally use streams to provide a convenient way to continuously process new or changed data. A task can verify whether a stream contains changed data for a table and either consume the changed data or skip the current run if no changed data exists. In our use case, it will be new data coming from S3 and we push them to the relational table from json format saved in Variant data type.


## Create Streams from the table which hold json in variant data type
For this illustration, create a stream called CLOUDTRAIL_STREAM and CLOUDTRAIL_STREAM2 on the table CLOUDTRAIL_ROLE. CLOUDTRAIL_ROLE is the place where the json logs are saved in variant.

```
CREATE OR REPLACE STREAM CLOUDTRAIL_STREAM
    ON TABLE SNOWTABLE_ROLE;

CREATE OR REPLACE STREAM CLOUDTRAIL_STREAM2
    ON TABLE SNOWTABLE_ROLE;

```

CLOUDTRAIL_STREAM and CLOUDTRAIL_STREAM2 will capture every change in this case new data flowing into the table from snowpipe. As the data flows in streams will capture the inserts. 


## Create relational tables that will store the transformed json data
The following tables are created to store the data that will be generated from the transformation of the json file. 

```
CREATE OR REPLACE TABLE HASHMAP_TRAINING_DB.TRAINING.SNOWTABLE_ROLE_PROD1 (AWSREGION STRING,
                                             EVENTID STRING);


CREATE OR REPLACE TABLE HASHMAP_TRAINING_DB.TRAINING.SNOWTABLE_ROLE_PROD2 (AWSREGION STRING,
                                                                           RECEPIENT_ACCOUNTID NUMBER,
                                                                           REQUESTID STRING);

```

This transformation and load will now be automated using Task.


## Create Tasks to automate the cdc from the variant table stream to the relational tables
We will be using the following DML statements to get the data from json to the tables that we created.

Use the following statements to create tasks. To understand the different options of creating a task, we have the first task, SNOWTABLE_ROLE_PROD1 that will be based on a 5 minute schedule and the second task, SNOWTABLE_ROLE_PROD2 based on the first task. The Task SNOWTABLE_ROLE_PROD will be the predecessor of the task SNOWTABLE_ROLE_PROD2.

```
CREATE OR REPLACE TASK CLOUDTRAIL_TSK1
    WAREHOUSE = TRAINING_WH 
    SCHEDULE = '5 MINUTES'
WHEN
    SYSTEM$STREAM_HAS_DATA('CLOUDTRAIL_STREAM')
AS    
    INSERT INTO HASHMAP_TRAINING_DB.TRAINING.SNOWTABLE_ROLE_PROD1
    SELECT F.VALUE:awsRegion::STRING AS AWS_REGION
        , F.VALUE:eventID::STRING AS EVENT_ID 
    FROM HASHMAP_TRAINING_DB.TRAINING.CLOUDTRAIL_STREAM T
       , LATERAL FLATTEN (input => T.V) F
       , LATERAL FLATTEN (input => F.VALUE:resources) LF
    WHERE METADATA$ACTION = 'INSERT';


CREATE OR REPLACE TASK CLOUDTRAIL_TSK2
    WAREHOUSE = TRAINING_WH 
    AFTER CLOUDTRAIL_TSK1
AS
    INSERT INTO HASHMAP_TRAINING_DB.TRAINING.SNOWTABLE_ROLE_PROD2
    SELECT F.VALUE:awsRegion::STRING AS AWS_REGION
       , F.VALUE:recipientAccountId::NUMBER AS RECEPIENT_ACCOUNT_ID
       , F.VALUE:requestID::STRING AS REQUEST_ID
    FROM HASHMAP_TRAINING_DB.TRAINING.CLOUDTRAIL_STREAM2 T
       , LATERAL FLATTEN (input => T.V) F
       , LATERAL FLATTEN (input => F.VALUE:resources) LF
    WHERE METADATA$ACTION = 'INSERT';

```

Execute the following command to ensure the tasks are created. 

SHOW TASKS;


## Activate the Task
You will see the state of the tasks are listed as suspend. The Task by default will be created in suspended status. We have to Resume the task for them to execute based on the setting we made.


```
ALTER TASK CLOUDTRAIL_TSK1 RESUME;

ALTER TASK CLOUDTRAIL_TSK2 RESUME;

```

## Verify the results
To confirm the data are successfully transformed and loaded, execute the select statements below.

```
SELECT * FROM SNOWTABLE_ROLE_PROD1;

SELECT * FROM SNOWTABLE_ROLE_PROD2;

```
