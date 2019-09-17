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
