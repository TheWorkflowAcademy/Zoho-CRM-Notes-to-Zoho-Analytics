# Zoho-CRM-Notes-to-Zoho-Analytics
## Purpose
This repository addresses the need to display Notes from Zoho CRM within Zoho Analytics. The Zoho CRM / Zoho Analytics default integration does not pull Notes on Zoho CRM records into Zoho Analytics. In order to do this, new Notes need to be queryied from Zoho CRM and sent to Zoho Analytics via API.

## Implementation Overview
The major components of this implementation include:
1. Scheduled Zoho CRM function which performs the following:
  - Retrieve all newly created Notes
  - Send Notes to Zoho Analytics via API
2. Table in Zoho Analytics to store Notes

## Connections
Two connections within Zoho CRM are required:
- Zoho CRM: `scope=ZohoCRM.modules.ALL`
- Zoho Analytics: `scope=ZohoAnalytics.data.all`

Use the default connections in Zoho CRM for both of these.

## Retreive New Notes
Since this will run on a scheduled function which is invoked every 2 hours, we will request all the Notes *updated* within the last two hours. 

The endpoint to use is the general [Notes endpoint](zoho.com/crm/developer/docs/api/v2/get-notes.html) in the Zoho CRM API.
