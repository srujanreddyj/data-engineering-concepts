Project: Data Pipelines with Airflow
---
## Project Description: 
A music streaming company wants to introduce more automation and monitoring to their data warehouse ETL pipelines and they have come to the conclusion that the best tool to achieve this is Apache Airflow. As their Data Engineer, I was tasked to create a reusable production-grade data pipeline that incorporates data quality checks and allows for easy backfills. Several analysts and Data Scientists rely on the output generated by this pipeline and it is expected that the pipeline runs daily on a schedule by pulling new data from the source and store the results to the destination.

### Data Description: 

The source data resides in S3 and needs to be processed in a data warehouse in Amazon Redshift. The source datasets consist of JSON logs that tell about user activity in the application and JSON metadata about the songs the users listen to.

### Data Pipeline design: 
The data pipeline does the following tasks.
1. Extract data from multiple S3 locations.
2. Load the data into Redshift cluster.
3. Transform the data into a star schema.
4. Perform data validation and data quality checks.
5. Calculate the most played songs for the specified time interval.
6. Load the result back into S3.


### Design Goals: 
Based on the requirements of our data consumers, our pipeline is required to adhere to the following guidelines:

* The DAG should not have any dependencies on past runs.
* On failure, the task is retried for 3 times.
* Retries happen every 5 minutes.
* Catchup is turned off.
* Do not email on retry.

Pipeline Implementation:

Apache Airflow is a Python framework for programmatically creating workflows in DAGs, e.g. ETL processes, generating reports, and retraining models on a daily basis. The Airflow UI automatically parses our DAG and creates a natural representation for the movement and transformation of data. A DAG simply is a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies. A DAG describes how you want to carry out your workflow, and Operators determine what actually gets done.

By default, airflow comes with some simple built-in operators like PythonOperator, BashOperator, DummyOperator etc., however, airflow lets you extend the features of a BaseOperator and create custom operators. For this project, I developed several custom operators.

The description of each of these operators follows:

StageToRedshiftOperator: Stages data to a specific redshift cluster from a specified S3 location. Operator uses templated fields to handle partitioned S3 locations.
LoadFactOperator: Loads data to the given fact table by running the provided sql statement. Supports delete-insert and append style loads.
LoadDimensionOperator: Loads data to the given dimension table by running the provided sql statement. Supports delete-insert and append style loads.
SubDagOperator: Two or more operators can be grouped into one task using the SubDagOperator. Here, I am grouping the tasks of checking if the given table has rows and then run a series of data quality sql commands.
HasRowsOperator: Data quality check to ensure that the specified table has rows.
DataQualityOperator: Performs data quality checks by running sql statements to validate the data.
SongPopularityOperator: Calculates the top ten most popular songs for a given interval. The interval is dictated by the DAG schedule.
UnloadToS3Operator: Stores the analysis result back to the given S3 location.
Code for each of these operators is located in the plugins/operators directory.