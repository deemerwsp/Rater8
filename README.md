# Rater8

The Rater8 program is designed to ingest monthly data from the rater8 API, update the Azure SQL database, generate various reports, and handle errors for debugging purposes.

The program is designed to refresh the month's data. To ensure all data for a month is captured, if the program execution data is the first of the month, the program will refesh last month's data in the SQL database with the entire month's data from the rater8 API. Three variables are defined at the start of the program to ensure this functionality: 'month', 'first_of_month', 'last_of_month'. These variables are defined based on the condition if the datetime is the first day of the month. All subsequent code uses these three variables. 

## Program Sections
1. **Imports:** Import necessary Python modules and libraries.
2. **Get Dates:** Retrieve current datetime for recording in the `program_report` and specify month range for data retrieval.  
3. **Credentials:** Define variables for sensitive login information.
4. **Functions:** Define program functions.
5. **Data Retrieval:** Retrieve data from the Azure SQL database and the rater8 API.
6. **Data Manipulation:** Format retrieved data to comply with SQL database tables.
7. **Upload Review Data to Raw Tables:** Upload formatted review data to raw tables in the SQL database.
8. **Create Reports:** Generate various reports based on the uploaded data.
9. **Update dbo Tables:** Execute SQL stored procedures to update dbo tables from the uploaded raw tables.
10. **Upload Reports:** Upload generated reports to the SQL database.

## Program Steps
### Step 1: Retrieve data from SQL Database
Responses data for this month and all API Permissions data is retrieved from the SQL database and stored as Dataframes. <br>
&emsp; DataFrames <br>
&emsp; Responses: prov_reviews_in_database, loc_reviews_in_database <br>
&emsp; Permissions: prov_perms_in_db, loc_perms_in_dab <br>
### Step 2: Retrieve rater8 API Permissions
Make API GET request to fetch API Permissions for Locations and Providers and create corresponding DataFrames. 
### Step 3: Retrieve rater8 Responses
### Step 4: Manipulate Data 
### Step 5: Upload Responses Data to raw Tables
### Step 6: Upload API Permissions Data
### Step 7: Create Reports
### Step 8: Update dbo Tables 
### Step 9: Upload Reports 


## Program Interfaces
1. VistamarConsulting Azure SQL Database
2. rater8 API
3. Email

## Error Handling
The function `email_error_report` is incorporated into all program functions for proper error handling and reporting. In case of an error, the program terminates, and an email with the `program_report` is sent. The termination time is recorded in the `program_report`. The email report provides details on the terminated program, allowing the recipient to analyze and determine the appropriate course of action. It is crucial to remove any data uploaded/updated in the SQL database if the program did not complete successfully before re-executing the corrected program.

**Note:**<br>
The workflow is structured to update the dbo tables from the raw tables towards the end of the program. An error is most likely to manifest before reaching this phase of the program.

### Example Error Case
The program successfully updated the Locations dbo table from the raw table before encountering an error during the attempt to update the Providers dbo table. This resulted in the termination of the program and the automatic generation of an email error report.

The point at which the program terminated is clearly indicated in the error message included in the email report. Additionally, the program_report attached to the email provides a comprehensive breakdown of the program's execution, allowing easy identification of the termination point. In this instance, there will be an execution time recorded for the process of updating the Locations dbo table, while subsequent update execution times will be marked as Null. (Note: the columns in program_report are arranged chronologically based on the program's execution sequence).

Given this scenario, data was uploaded to the raw schema, and Location response data was successfully updated to the dbo table. To address this error and rerun the program after debugging, it is recommended to locate the reviews that were updated to the Locations dbo table. This can be achieved by filtering based on the 'ReviewMonth' or by matching the data to the corresponding entries in the raw table. Subsequently, this newly identified data should be removed from the dbo table. This step ensures the absence of accidental duplicates within the dbo table upon re-execution of the program.

## Deployment
The program is crafted to function seamlessly as a stored procedure within SQL Server Management Studio (SSMS). Successful execution relies on the SSMS package for Machine Learning, particularly the SSMS stored procedure sp_execute_external_script. It's imperative to have the requisite external Python libraries installed within the stored procedure to ensure smooth functionality.

Scheduled for daily execution at 11:00 AM, the program is optimized to align with the operational hours of the rater8 API. This timing mitigates any potential disruptions, accounting for the API's downtime from 9:00 PM to 10:00 AM. By scheduling the program to run at 11:00 AM, it guarantees a reliable HTTP interaction with the rater8 API for seamless data retrieval and processing.

# Program Report 
|Column|Description|
|:--|:--|
|Id|The 'Id' field serves as an auto-incremented primary key, overseen by SQL Server. Given the additive nature of reports, there is no necessity to reseed the primary key before uploading the python program's data to the SQL table.|
|InsertDate|Date of program execution.|
|Launched|Timestamp the program launched execution.|
|DBLocRespRetr|Execution time for fetching month's Location Responses from SQL Database and storing as DataFrame. |
|DBProvRespRetr|Execution time for fetching month's Provider Responses from SQL Database and storing as DataFrame.|
|DBProvPermsRetr|Execution time for fetching all Provider API permissions from SQL Database and storing as DataFrame.|
|DBLocPermsRetr|Execution time for fetching all Location API permissions from SQL Database and storing as DataFrame.|
|APIPermsRetr|Execution time for fetching all API permissions from rater8 API and storing as DataFrame.|
|APILocRetr|Execution time for fetching month's Location Responses from rater8 API, aggregating all together as DataFrame.|
|APIProvRetr|Execution time for fetching month's Responses Responses from rater8 API, aggregating all together as DataFrame.||
|LocRespUpload|Execution time for uploading month's Location respones to SQL raw table.|
|ProvRespUpload|Execution time for uploading month's Provider respones to SQL raw table.|
|LocPermsUpload|Execution time for uploading Location API permissions to SQL rater8 table.|
|ProvPermsUpload|Execution time for uploading Provider API permissions to SQL rater8 table.|
|LocResp|Execution time to calulate counts of new responses per Location and create report DataFrame.|
|ProvResp|Execution time to calulate counts of new responses per Location per source company and create report DataFrame.|
|LocPerms|Execution time to identify Locations that are newly subscribed or unsubscribed to rater8 and create report DataFrame.|
|ProvPerms|Execution time to identify Providers that are newly subscribed or unsubscribed to rater8 and create report DataFrame.|
|LocRespUpdate|Execution time to upload month's Location response data to SQL raw table. |
|ProvRespUpdate|Execution time to upload month's Provider response data to SQL raw table. |
|QuestionsUpdate|Execution time to update rater8.QuestionResponses SQL table. |
|PermsReportUpload|Execution time to upload report of Locations and Providers that are newly subscribed or unsubscribed to rater8.|
|LocReportUpload|Execution time to update rater8.LocResponses with data from raw.Rater8LocResponses$.|
|ProvLocReportUpload|Execution time to update rater8.ProvLocResponses with data from raw.Rater8ProvResponses$.|
|Terminated|Timestamp the program encountered and error and was terminated. |
|Program||
|Completed||

# Python 
### Imports
|Module/Library|Alias|Use|
|:--|:--|:--|
|datetime||Capture precise timestamps for code execution in the program_report. Record the program's termination datetime if any errors occur during execution.| 
|pandas|pd|Manage structured data efficiently through DataFrames.|
|json||Encode/decode JSON data to ensure proper formatting for SQL database.|
|numpy|np|Filter various python null values and convert to SQL accepted Nulls.|
|pymssql||Establish an interface to interact with the SQL database.|
|requests||Facilitate HTTP interactions with the rater8 API.|
|smtplib||Employ the SMTP interface for sending email error reports in the event of program failure.|
|email||Generate and format email messages for error reporting, used in conjunction with smtplib.|
|sys||Terminate program upon encountering any error.|
|linecache|| Capture and include error code details in the email report upon program termination.|

### Functions
|Function|Description|
|:--|:--|
|runtime()|Records the function's execution time, documenting it in the program_report. Upon successful completion of the program, it records the total 'Program' execution time and the 'Completed' timestamp in the program_report.|
|connect_to_sql_db()|Establishes a connection and cursor for interactions with the SQL database.|
|disconnect_from_sql_db()|Closes the cursor and connection, integrated into the error handling function to ensure proper disconnection from the SQL database.|
|email_error_report()|Dispatches an email with an error report, terminates the program, and disconnects from the SQL database if still connected.|
|retrieve_db_data()|Establishes a connection to the SQL database using connect_to_sql_db, retrieves location/provider data for the current month, creates a DataFrame to return as a program variable, and then disconnects from the SQL database using disconnect_from_sql_db.|
|get_rater8_permissions()| Retrieves rater8 API permissions for locations and providers, creating pandas DataFrames for each. Cleans clientCodes for the DataFrames; providers have a single clientCode, while locations have multiple. Returns the DataFrames as program-defined variables.|
|get_rater8_data()|Iterates over rater8 location/provider permissions to make GET requests to the API and aggregates the data into a DataFrame. Location/provider permission data is added to corresponding data upon iteration. The response data is retrieved for providers (e.g., "John Doe"), including response data but without the field 'provider'="John Doe." This field is inherent in the API GET request, so it is concatenated with the retrieved data along with 'clientCodes' and 'provId'. The aggregated DataFrame of location/provider responses is returned as a Python variable for further use.e|
|prepare_data()|A robust function to upload various DataFrames to the SQL database (response data, reports). For response data, it requires deleting SQL data to replace it with new data. Reports are additive and do not require deleting SQL table data.|
|upload_data()|Robust function to upload various DataFrames to SQL database (response data, reports). Response data requires deleting SQL data to replace with new data. Reports are additive and do not require deleting SQL table data.|
|update_table()|Executes SQL stored procedures to update dbo tables from uploaded raw tables for the current month.|
|get_prov_loc_new_company_count_report()|Calculates counts of new responses for each loc/prov per review source, records data to loc/prov reports and returns DataFrames to be uploaded to respective reports.|
|permissions_report()|Determine loc/prov that are unsubscribed/newly-subscribed to rater8. Returns a DataFrame to be uploaded as report to SQL database.|

# SQL
### Tables
|Schema|Table|Description|
|:--|:--|:--|
|raw|rater8LocResponses$|Serves as the repository for raw data retrieved from the rater8 API for an entire month. The `upload_data` function is employed in this process, systematically erasing all pre-existing entries from the SQL table. Subsequently, the freshly obtained data from the API is seamlessly integrated into the SQL table, effectively overwriting the existing dataset.|
|raw|rater8ProvLocResponses$|Fulfills the identical role as the aforementioned raw.rater8LocResponses$ table, albeit exclusively for Provider Responses.|
|rater8|LocResponses|Main repository for all rater8 Location Responses.|
|rater8|ProvLocResponses|Main repository for all rater8 Provider Responses.|
|rater8|LocationPermissions|Holds the API permissions for locations registered with rater8. Its purpose is to pinpoint locations that have either newly subscribed or unsubscribed by cross-referencing with the program's recently acquired permissions table. Subsequently, the SQL table is updated with the refreshed permissions table obtained from the API.|
|rater8|ProviderPermissions|Fulfills the identical role as the aforementioned rater8.LocationPermissions table, albeit exclusively for Provider Permissions.|
|rater8|PermsReport|Newly subscribed or unsubscribed Locations and Providers.|
|rater8|LocResponsesUpdateReport|Provides a summary of new location responses, presenting the count of new responses for each location. It's important to note that locations exclusively receive responses from Google. The last row for each date encompasses the overall count of new location responses, summing up the data for all locations.|
|rater8|ProvLocResponsesUpdateReport|Fulfills the identical role as the aforementioned rater8.LocResponsesUpdateReport table, albeit exclusively for Provider responses.|
|rater8|ProgramUpdateReport|This feature furnishes detailed execution times for individual program segments, facilitating debugging in case of errors leading to program termination. Furthermore, it incorporates counts of new responses for both locations and providers, along with the tally of newly subscribed or unsubscribed locations and providers. Integrated into the program's error-handling mechanism, this report is dispatched alongside an email error report when the program is prematurely terminated for any reason.|

### Stored Procedures
|Stored Procedure|Description|
|:--|:--|
|UploadRater8LocResponses|Upload full month's data for location responses to |
|UpdateRater8LocResponses|This stored procedure refreshes the rater8.LocResponses table with the complete monthly dataset from raw.rater8LocResponses$. In the initial step, it clears all records in rater8.LocResponses where the 'ReviewMonth' matches the month provided in the stored procedure's argument, @ReviewMonth. The subsequent step involves reseeds the primary key using the last (max) 'Id' present in rater8.LocResponses. Finally, the data from raw.rater8LocResponses$ is loaded into rater8.LocResponses.|
|UploadRater8ProvLocResponses|This table fulfills the identical role as the aforementioned UpdateRater8LocResponses, albeit exclusively for Provider Responses.|
|UpdateRater8ProvLocResponses|Updates rater8.ProvLocResponses with the full month's data from raw.rater8ProvLocResponses$|
|UpdateQuestionResponses|Deletes question response data in rater8.QuestionResponses. Parses questions responses from raw.rater8ProvLocResponses$ and updates the full month's question responses to Rater8.QuestionResponses using the table-valued function dbo.tvfnParseJsonQuestions|

### Functions
|Function|Descriptioin|
|:--|:--|
|tvfnParseJsonQuestions|Parses survey questions to fields ProvLocResponseId, QuestionId, QuestionPosition, Rating, Comment. Takes the argument @Id to match survey question responses to provider responses.|
