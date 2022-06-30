# CRM Notes to Analytics
## Purpose
This repository addresses the need to display Notes created in Zoho CRM within Zoho Analytics. The Zoho CRM / Zoho Analytics default integration does not pull Notes on Zoho CRM records into Zoho Analytics. In order to do this, new Notes need to be queried from Zoho CRM and sent to Zoho Analytics via API.

## Implementation Overview
The major components of this implementation include:
1. Scheduled Zoho CRM function which performs the following:
  - Retrieve all newly created Notes on Account records
  - Send Notes to Zoho Analytics via API
2. Table in Zoho Analytics to store Notes

## Connections
Two connections within Zoho CRM are required:
- Zoho CRM: `scope=ZohoCRM.modules.ALL`
- Zoho Analytics: `scope=ZohoAnalytics.data.all`

Use the default connections in Zoho CRM for both of these.

## Retrieve New Notes
Since this will run on a scheduled function which is invoked every 2 hours, we will request all the Notes *updated* within the last two hours. 
The endpoint to use is the general [Notes endpoint](zoho.com/crm/developer/docs/api/v2/get-notes.html) in the Zoho CRM API. To retrieve the most recently updated Notes, include the ```{"If-Modified-Since":modifiedSinceStr}``` snippet as your ```headers``` parameter, after defining your ```modifiedSinceStr``` as 2 hours ago.

Here is a sample request:
```
notesResponse = invokeurl
  [
   url :"https://www.zohoapis.com/crm/v3/Notes?fields=Created_By,Created_Time,Note_Title,Modified_By,Modified_Time,Note_Content,$se_module,Parent_Id,id&page=" + page + "&per_page=" + perPage
   type :GET
   headers:{"If-Modified-Since":modifiedSinceStr}
   detailed:true
   connection:"crm"
  ];
```

In the case where there are more than 200 updated Notes in the previous two hours, the request will need to be wrapped in an [API Pagination Block](https://github.com/TheWorkflowAcademy/api-pagination-zohocrm) to perform multiple requests of Notes. This essentially simulates a `while` loop in Deluge. The condtion will be whether the `Modified_Time` is within the last two hours.

## Create Notes in Analytics
Once all the Notes are queried you can now send them over to Zoho Analytics via API. The Zoho Analytics endpoints for manipulating data in a table do not include an "Add or Update" endpoint, so we need to determine beforehand if you will "Add" or "Update" the row in Analytics.

Within a ```for each``` loop for all of our notes, we run a search, using the export data API endopoint, to see if the specific Note already exists in our Analytics table. That search looks something like this:
```
config = Map();
 config.put("responseFormat","json");
 config.put("keyValueFormat",true);
 criteria = "\"Note ID\"=" + id.toString();
 config.put("criteria",criteria);
 paramsMap = Map();
 paramsMap.put("CONFIG",config.toString());
 // ------
 exportDataResponse = invokeurl
 [
  url :exportUrl
  type :GET
  parameters:paramsMap
  headers:headersMap
  connection:"analytics"
 ];
 info "Export Data Response: " + exportDataResponse;
 ```
 
 Then, based on the result, we can determine if we need to update a row in Analytics or add a row
 
 ```
 if(!exportDataResponse.get("data").isEmpty())
{
// Update the row with the rowMap that we created with our Note fields
}
else
{
// Create a new row in our Analytics table using the same rowMap
}
```

Here are the endpoints for the Zoho Analytics API:
- Add Row: https://www.zoho.com/analytics/api/v2/#add-row
- Update Data: https://www.zoho.com/analytics/api/v2/#update-row (However, we have had problems with this API call and suggest using the integration task found here
- Export data API: https://www.zoho.com/analytics/api/v2/#export-data


An alternative to calling the Export Data API to check and see if a record exists is to simply let the CRM data determine if a row needs to be updated or added. Some simple logic for "Is the created time different than the modified time?" should do the trick because we are calling all *modified records* in the last 2 hours. 

In this repo though, we demonstrate how to use the Export Data API

## Table in Zoho Analytics
A table will need to be created in Zoho Analytics to store the Account Notes. Here is some documentation to do so: https://www.zoho.com/analytics/help/table/creating-table.html

The following columns will need to be created:
- Note ID
- Note Title
- Content
- Module
- Parent ID
- Modified By
- Created By
- Modified Time
- Created Time
