# Serverless-Data-Lake
Serverless Data Lake
What is a Data Lake?
In today's data-driven world, organisations deal with massive volumes of structured and unstructured data coming from various sources. To manage, store, and analyse this diverse data efficiently, Data Lakes have emerged as a flexible and scalable solution.
A Data Lake is a centralised repository that allows you to store all your data - structured, semi-structured, and unstructured - at any scale. You can store your data as-is, without having to structure it first, and run different types of analytics - from dashboards and visualisations to big data processing, real-time analytics, and machine learning - to guide better decisions.
While traditional data lakes often require managing servers , storage and complex pipelines, a Serverless Data Lake takes this a step further by removing the need to manage infrastructure.
In this project I have implemented Serverless Data lake architecture using various AWS services through one of AWS's official workshops. The workflow involved data ingestion, transformation, and visualisation, all without provisioning or managing any servers. Below is an overview on how I built this end-to-end solution using tools like AWS Cloud Formation, S3, Glue, Athena, and QuickSight.
Step 1: Deploy a CloudFormation template (serverless-anayltics.yaml)
Click on Create Stack
Click on Choose an existing template
Under specify template → Upload a template file
Click on next
For stack name → sdl-<your-name>
Click next until review page and then click on submit

Wait for the resources to be provisioned
Step 2: Create a glue crawler
Click on create crawler
Specify the name of the crawler and click on next
Choose S3 as data source and then browse s3 for s3 path
serverlessanalytics-<account-number>-raw
Leave all other options as default and click on next
For IAM role choose → ServerlessAnalyticsRole
Under Output configurations click on add database and specify the database name. can specify raw as prefix for the tables.
Review all the options and click on create crawler
In the crawlers dashboard select the newly created crawler and click on run

Step 3: Review the metadata in glue catalog table
In the navigation menu, click Tables
On tables page, click on yellow_tripdata to review table metadata and schema information.

Next we will explore our data using Amazon Athena.
Step 4: Go to Amazon Athena console, click on workgroups and select primary.
Click on edit and then click on query result configuration
select location for query result as → s3://serverlessanalytics-<account-number>-athena/
Click on save changes

Step 5: Go to Athena's query editor
Select database as sql_db
In the tables section → click on 3 dots in front of raw_yellow_tripdata and select preview table option
You will see something like this:

You can perform similar steps for taxi_zone_lookup_table
You can now run various SQL queries to explore the data

Now we will transform data by enhancing existing data with additional data from other sources for more useful and insightful analysis
Step 6: AWS Glue Studio
First plan the data transformation steps 
Like we can join yellow trip data with taxi zone lookup to obtain pickup location information.
Remove records with NULL values (vendorid, payment_type, passenger count, ratecodeid).
Filter records within a time period
Join yellow trip data with taxi zone lookup to obtain drop-off location information
Save processed dataset to S3 in a query optimised format. etc….

Go to AWS Glue console and click on ETL jobs
Click on job details
specify the name → Transform NYC taxi drip data
Select ServerlessAnaylticsRole as IAM Role
click on save

Go to Visual now
select amazon S3
Under the data source properties
specify the name → Yellow Trip data
Select data catalog table
select the database and table name → raw_yellow_tripdata
click on save
Now to remove null records

Go to transform option
Select custom transforms
specify the name → Removing Null Records
select your previously selected s3 bucket as parent node
Paste the code block from nullRecords.py
click on save

Now to join raw_yellow_tripdata with raw_taxi_zone_lookup
repeat the same process which we did earlier with raw_yello_tripdata for raw_taxi_zone_lookup s3 bucket

Go to transform option
select join
specify the name → Yellow Trips Data + Pickup Zone Lookup
select node parents
yellow trip data
pickup zone lookup
join type → inner join
Specify the join condition that you want to have
click on save

You can try various transform methods in AWS Glue Studio based on your needs. After applying different transformations, save the final transformed data to Amazon S3.
Step 7: Saving the transformed state
go to targets option and click on S3
specify the name → Transformed Yellow Trip Data
select the s3 target location → s3://serverlessanalytics-<account-number>-transformed
click on save

In my case:
After specifying all the necessary transformations, click on RUN
You can also view the metrics:
Step 8: Now extracting the metadata and schema of transformed data using crawler
Go to aws glue
From the navigation menu select crawler, click on add crawler
Repeat the same steps which we did when creating previous crawler, but this time choose data source as S3 and S3 path → serverlessanalytics-<account-number>-transformed
click on run

Step 9: Again go to amazon Athena for querying transformed data
go to the query editor
now you will see one extra table → serverlessanalytics_<account-number>_transformed

click on three dots and select preview table

(Important: The results can be different based on your data transformation)
Step 10 (Optional) :You can enrich the table with additional data for eg: check view.sql file
Now to visualise the data which we have transformed , we can use QuickSight.
Amazon QuickSight is a scalable, serverless, embeddable, machine learning-powered business intelligence (BI) service built for the cloud. QuickSight lets you easily create and publish interactive BI dashboards that include Machine Learning-powered insights. QuickSight dashboards can be accessed from any device, and seamlessly embedded into your applications, portals, and websites.
Step 11: Using Amazon QuickSight
Go to QuickSight console, click Sign up for QuickSight
Specify the following:
region → us-east-1
account-name → sdl-<your-choice>
email address → <email-address>
Check Amazon S3 and click choose bucket
select your transformed bucket → serverlessanalytics_<account-number>_transformed
Click Finish

Go to the QuickSight, click datasets in the left navigation menu
Click on new datasets and select Athena
data source name → Athena-sdl
workgroup → primary
click create data source

In choose your table window
catalog → AwsDataCatalog
database →sdl_db
tables → v_yellow_tripdata
click select

In the finish dataset creation window → Import to SPICE for quicker analytics.
QuickSight will attempt to load your data to SPICE, please wait for a few minutes.
Great! Your dataset has now been imported into QuickSight dashboard. Now you can play with different visual types for interactive dashboards and insights
Example: You can select sankey visual type with source → <any-field> and destination → <any-field>
Or you can choose Bar chart for payment type:
Similarly, you choose other visual types, can apply filter and test different ways to visualise your data.
Great! In this project, you successfully built an end-to-end serverless data pipeline encompassing data ingestion, data transformation, and data visualisation. You leveraged various AWS serverless services to create a robust and scalable data lake architecture.
Data Ingestion: You ingested data from various sources into Amazon S3, the central data lake storage. This step involved capturing and storing raw data in its original format for later processing.
Data Transformation: Using AWS Glue, you defined and executed ETL (Extract, Transform, Load) jobs to process the raw data stored in S3. These jobs transformed the data into a structured format suitable for analysis and reporting.
Data Catalog: AWS Glue Data Catalog played a crucial role in organising and managing the metadata for your data lake. It provided a centralised repository for storing schema definitions, data lineage, and other metadata related to your datasets.
Data Analysis: With Amazon Athena, you were able to query and analyse the transformed data stored in S3 using standard SQL syntax. Athena's serverless architecture allowed you to run ad-hoc queries without provisioning or managing any infrastructure.
Data Visualisation: Finally, you connected Amazon QuickSight to your data sources, enabling you to create interactive visualisations and dashboards. QuickSight provided a user-friendly interface for exploring and presenting your data insights.
