# Rater8
**Overview** <br>
Ingest monthly rater8 data, update SQL database, provide various reports, handle errors for debugging. 

**Program Process:** <br>
1. Imports: python modules/libraries
2. Get Dates: 
3. Credentials: define variables for sensitive login info
4. Functions: define functions
5. Data Retrieval: retrieve data from SQL database and rater8 API
6. Data manipulation: format retrieved data to comply with SQL database tables 
7. Upload Review Data to raw tables
8. Create Reports: 
9. Update dbo tables: execute SQL stored procedures to update dbo tables from uploaded raw tables
10. Upload Reports: 

**Deployment** <br>
This program will be executed wihtin SQL Server Management Studio as a stored procedured. It requires the SSMS package for Machine Learning in order to run the SSMS stored procedure sp_execute_external_script. The stored procedure will require installing this program's required external libraries for python. This stored procedure will be given a time trigger in order to execute everyday at 11:00am. <br>
Note:<br>
  Rater8's API goes dark from 9:00pm to 10:00am. This program will be executed regularly at 11:00am to ensure successful HTTP interaction with the rater8 API. 

## Description
**Program Interfaces:**<br>
1. VistamarConsulting Azure SQL Database
2. rater8 API
3. email<br>

**Error Handling:**<br>
The function `email_error_report` is incorporated in all program functions for proper error handling and reporting. If any error is incurred the program will terminate and send an email with the `program_report`. The time of termination is recorded within the dataframe `program_report`. An email report is sent to the program's designated recipient. This allows the recipient to analyze the terminated program from the `program_report` and determine how to proceed. The error message included in the email error report will direct you to the point at which the program failed. Appropriate testing and debugging is required. If any data was uploaded/updated within the SQL database without the program succesfully completing entirely, this data must be removed from the SQL database before the newly corrected program is executed again. 

# Imports
|Module/Library|Alias|Use|
|:--|:--|:--|
|datetime||Record specific code execution times for `program_report`. Program terminated datetime recorded if any error incurred during program execution.| 
|pandas|pd|Handle structured data as DataFrames.|
|json||Encode/decode JSON data to ensure proper formatting for SQL database.|
|numpy|np|Filter various python null values and convert to SQL accepted Nulls.|
|pymssql||Interface with SQL database.|
|requests||HTTP interface with rater8 API.|
|smtplib||SMTP interface for sending email error reports upon program failure.|
|email||Create and format email messages for error reporting used in conjunction with smtplib.|
|sys||Terminate program upon error incursion.|
|linecache||Record error code in email report upon program termination.|

# Functions
|Function|Description|
|:--|:--|
|runtime|Records code execution time for a function. Records execution time for function in `program_report`. If program is completed, records 'Program' and 'Completed' to `program_report`.|
|connect_to_sql_db|Creates connection and cursor for SQL database interactions.|
|disconnect_from_sql_db|Closes cursor and connection. Incorporated in error handling function to ensure SQL database disconnection.|
|email_error_report|Sends email with error report. Terminates program. Disconnects from SQL database if still connected.|
|retrieve_db_data|Connect to SQL db with `connect_to_sql_db`, retrieve loc/prov data for current month, create Dataframe to return as program variable, `disconnect_from_sql_db`.|
|get_rater8_permissions|Get rater8 permissions from API for loc/prov's. Create pandas dataframes for each. Clean clientCodes for dataframes; providers have single, locations have multiple. Return DataFrames as program defined variables.|
|get_rater8_data|Iterates over rater8 loc/prov permissions to make GET requests to API and aggregate data to a DataFrame. Loc/prov permission data is added to corresponding data upon iteration as this data is not included in the rater8 API. i.e. rater8 data retrieved for provider "John Doe" includes response data, but does not include the field 'provider'="John Doe". This field is inherent with the API GET request, so it is concatenated with the retrieved data along with the 'clientCodes' and 'provId'. Loc/prov reponse aggregated data is returned as python variable for further use.|
|prepare_data|Formats response data in loc/prov DataFrames to adhere to SQL database table formats. Floats convert to integers. Various nulls are converted to python None to register as Null in SQL.|
|upload_data|Robust function to upload various DataFrames to SQL database (response data, reports). Response data requires deleting SQL data to replace with new data. Reports are additive and do not require deleting SQL table data.|
|update_table|Execute SQL stored procedures to update dbo tables from uploaded raw tables for current month.|
|get_prov_loc_new_company_count_report|Get counts of new responses for each loc/prov. Record data to loc/prov reports and return DataFrames to be uploaded to respective reports.|
|permissions_report|Determine loc/prov that are unsubscribed/newly-subscribed to rater8. Return respective dataframes to be uploaded as reports to SQL database.|
