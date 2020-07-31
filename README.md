## ENEM 2017 Analysis

### Files



### Introduction

This project aims to analyze the data from 2017's ENEM, a national exam that serves as part of the admission process to brazilian universities, as well as explore different tools and approaches to organizing, storing and displaying data within the context of a data analyst job.

### Selected technology and tools

- **AWS** for creating the environment to analyze and store data
- **AWS S3** to store all the raw and processed data before being inserted into the database
- **AWS Redshift** as the RMDB, based on PostgreSQL
- **AWS SageMaker** for programming, data cleaning, calculations, running queries. Python distribution was 3.6 from Anaconda (Jupyter Notebook) and libraries used were Dask, Pandas, NumPy, GeoPandas, psycopg2
- **Power BI** for generating the dashboards
- **GitHub** to document and display the code and analysis
- **MarkDown** to write the documentation

### Execution

The summary of the steps taken to execute this project could be described as follows:

1. Analyze the tasks, overview the files and suggest a modeling approach
2. Create AWS account
3. Set up users, VPC, policies within the system
4. Create S3 bucket and upload raw files
5. Create Redshift cluster and the database
6. Create SageMaker instance
7. Analyze CSV file in SageMaker with Python
8. Through SageMaker execute all the SQL commands to create tables and load data into the database
9. Through SageMaker analyze the exported data again and perform another cleaning routine to consolidate the tables
10. With Power BI Desktop, use DirectQuery to access the Redshift database
11. Create all the dashboard visuals
12. Publish the dashboard

### Dimension modeling

For this dataset I wanted to guarantee that any information was easily accessible even if they were not going to be used for the final product. Initially I split the original file into several parts and selected the table that contained the columns relative to all the scores of each candidate as my fact table, and all the other tables as dimensional tables, connected to the scores table. This would translate into a star scheme. However, the fact table did not keep the residence city of the participants as a column (it stayed with the general information), therefore the table for cities and geolocation was better constructed if connected to this general information one. The scheme looks like this in the end:

![image](https://github.com/eduardoansi/enem2017/blob/master/dimensionalmodeling.png)

I generated a snowflake scheme, even if just for one single dimensional table not directly connected to the fact table.

#### AWS ####

Having decided the modeling I went to AWS to check what would be the most efficient way of executing the project. I decided on S3 for storing, Redshift for database and SageMaker for coding.

I decided to keep my own computer resources used at a minimum level, and even within the AWS cloud I did not use very powerful specifications. This was a bit challenging, specially with SageMaker and Jupyter Notebook.

All the part of setting up the Amazon environment is not hard to do, although it requires some knowledge of what each element of the cloud service means. It is important to grant the necessary permissions for the users (in this case was only myself) to have the whole process running smoothly. Most of these services have a free tier option, even though performance suffers from time to time if those are selected.

#### Python ####

The SageMaker instance had only 4GB of RAM and 5GB but was enough to do the job. Using Pandas to import the csv was not an option because it would use the entire memory to read the file, so I went with Dask, a package built on top of Pandas but that takes advantage of parallel computing and has a lazy execution. I performed some initial data processing with it to check general attributes of the dataset. However, the memory limitation was considerable and I later decided to continue the processing within the database created on Redshift.

The data for cities, states and regions were treated with SageMaker. The files were quite small and I used Pandas to merge two datasets: one containing the geolocation of each city and another containing population, IDH, GINI and other data (this was not provided originally, so I extracted it from Atlas Brasil). I used GeoPandas to convert the Shape file into a GeoJson format and then merged it with the other dataframe.

Having all the files inside the S3 bucket, I went on to create the database and all its tables. Some queries I performed with Redshifts' own query editor, but mostly I did them from Python, using the library psycopg2.

#### SQL ####

The first tries to load the csv into the database yielded errors because of mismatched data types. I decided to import all the columns as varchar(200) and work with them from there. Postgre, differently from other SQL dialects, does not possess a 'try-cast' command, and usually the conversion from a data type to another can be tricky. Because I was specially focused on the score columns, I noticed that several rows had blank values for them. Considering that my final goal was to analyze the performance of the students based on their scores, I decided to delete all the rows that had blank values on the score columns. After that I fixed some other data inconsistencies and finally was able to convert the datatype to from varchar to float.

Having the data ready I inserted all the columns into their respective new tables. I performed the same with the cities table that was separate from this file.

#### Power BI ####

With Power BI I could connect directly to the Redshift database, not having to download or store any data in my computer. For displaying the information I used mostly Power BI's native visuals, only importing one for the histogram and another for the heatmap.

I published it within the web app and concluded the project.

#### Coding ####

Because almost all of my coding was done inside Jupyter Notebook, all the commands, even for SQL, are well documented there. The file is in this repository as well, named dataprocessing.ipynb.

### Alternative ways ###

I tried other ways of obtaining the same result within the AWS environment. Having expecience with EC2, I created a virtual machine and allocated enough storage to execute everything in the same instance (which is the equivalent of doing everything locally, but in the cloud). I had Amazon Linux AMI with native support to Python, MySQL and other languages. Within the terminal I updated Python to a more recent version, installed iPython to write and execute the algorithms and installed and configured a MySQL server. However, I had a problem with the MySQL connection from outside the virtual instance (the instance was being connected with ssh and MySQL only with normal password through TCP/IP), and I could not easily manage to access the data from Power BI. In the end it worked out, but I had tested Redshift and decided to stick to it. I also used Amazon RDS to create a MySQL database, but Redshift performed better and connecting to it was never an issue.

### Conclusion

I appreciate the opportunity to have a deadline to work on this project during the last week. I believe the end result was satisfying and the whole process has been very rewarding. I hope you like it too, and I appreciate all kinds of feedback. I really enjoy learning and figuring things out.
