# CRM Notes to Analytics
## Purpose
This repository addresses the need to display Notes on Accounts from Zoho CRM within Zoho Analytics. The Zoho CRM / Zoho Analytics default integration does not pull Notes on Zoho CRM records into Zoho Analytics. In order to do this, new Notes need to be queried from Zoho CRM and sent to Zoho Analytics via API.

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

## Retreive New Notes
Since this will run on a scheduled function which is invoked every 2 hours, we will request all the Notes *updated* within the last two hours. 
The endpoint to use is the general [Notes endpoint](zoho.com/crm/developer/docs/api/v2/get-notes.html) in the Zoho CRM API. To retrieve the most recently updated Notes, use the following query parameters`sort_by=Modified_Time` and `sort_order=desc`.

Here is a sample request:
```
notes = invokeurl
[
	url :"https://www.zohoapis.com/crm/v2/Notes?sort_by=Modified_Time&sort_order=desc&page=" + page + "&per_page=" + per_page
	type :GET
	connection:"zohocrm"
];
```

In the case where there are more than 200 updated Notes in the previous two hours, the request will need to be wrapped in an [API Pagination Block](https://github.com/TheWorkflowAcademy/api-pagination-zohocrm) to perform multiple requests of Notes. This essentially simulates a `while` loop in Deluge. The condtion will be whether the `Modified_Time` is within the last two hours.

**Only Notes from Account records should go into Analytics**. You can filter these with the `$se_module` field on the record.

## Create Notes in Analytics
Once all the Notes are queried you can now send them over to Zoho Analytics via API. The Zoho Analytics endpoints for manipulating data in a table do not include an "Add or Update" endpoint, so we need to determine beforehand if you will "Add" or "Update" the row in Analytics.

The condition for Adding a new Row is whether the `Created_Time` occurs in the previous two hours. When a new Note is created, the `Created_Time` and the `Modified_Time` are the same. Since we are querying the Notes from CRM based on `Modfieid_Time`, you can assume any Note not created in the last two hours needs to be updated in Analytics, since it was updated previously.

```
start_time = zoho.currenttime;
last_run_time = start_time.subHour(2);

for each  note in all_notes
{
  // Convert Modfied Time and Created Time to DateTime objects
	modified_time = note.get("Modified_Time").replaceAll("T"," ").toDateTime();
	created_time = note.get("Created_Time").replaceAll("T"," ").toDateTime();
  
  // If Created Time is within last two hours, create new Row in Analytics
	if(last_run_time > created_time)
	{
    // Create Note in Analytics
	}
  // Note should already exist in Analytics, but Note has been changed in Zoho CRM
	else
	{
		// Update Note in Analytics
	}
}
```

Here are the endpoints for the Zoho Analytics API:
- Add Row: https://www.zoho.com/analytics/api/#add-row
- Update Date: https://www.zoho.com/analytics/api/#update-data


In the `NotesToAnalytics.txt` file, there is Deluge code that was previously written to accomplish this. There are few key differences:
- Rather than checking whether the Created_Time occured in the last two hours (or whatever timeframe is chosen), an API Request was sent to Analytics to check if the record already exists. This might still be the right approach, but it is probably easier to determine whether to Create or Update a row based on CRM Data than making an HTTP Request. 


## Table in Zoho Analytics
A table will need to be created in Zoho Analytics to store the Account Notes. Here is some documentation to do so: https://www.zoho.com/analytics/help/table/creating-table.html

The following columns will need to be created:
- Title
- Content
- Account Name
- Account Owner
