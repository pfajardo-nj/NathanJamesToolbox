# Table of contents
- [NathanJames Toolbox](#heading)
	* [Getting Started](#sub-heading)
		+ [Installing](#sub-sub-heading)
	* [Airtable Class](#sub-heading)
		+ [Overview](#sub-sub-heading)
		+ [Importing the module](#sub-sub-heading)
		+ [Create a URL using the table name](#sub-sub-heading)
		+ [Create a dictionary of columns](#sub-sub-heading)
		+ [Create a list of JSON containing all data](#sub-sub-heading)
		+ [Create a list of Airtable record IDs](#sub-sub-heading)
		+ [Delete a list of Airtable record IDs](#sub-sub-heading)
		+ [Push data to Airtable](#sub-sub-heading)
		+ [Create a list of data in an Airtable column](#sub-sub-heading)
		+ [Check Airtable table for duplicates based on column data](#sub-sub-heading)
		+ [Cleans up a string that is formatted as a list](#sub-sub-heading)
	* [Author](#sub-heading)
	* [License](#sub-heading)

<!-- toc -->

# NathanJames Toolbox

Collection of tools used by NathanJames

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Installing

You can use pip to install the package.

```
pip install NathanJamesToolbox
```

### AirtableToolbox Class
#### Overview
The class contains functions that help integrating with Airtable easier.

---

**Importing the module**

	>>> from NathanJamesToolbox import NathanJamesToolbox as nj

**AirtableToolbox Class Instance**

	_airtable = nj.airtableToolbox(<airtable base>, <airtable API Key>)
---
**Create a URL using the table name**
 - Create a formatted URL by supplying the table name

Example:

	>>> # _airtable.create_url(<Table Name>)
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master')
	>>> url
	'https://api.airtable.com/v0/Master'

**Create a dictionary of columns**
- Loop through all the pages and create a dictionary based on the table columns.

Example:

	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master?filterbyformula={SKU}=12345')
	
	>>> _dict_master = _airtable.create_dictionary(url, 'SKU', reverse=False)
	>>> _dict_master
	{'12345': ['recxyzId']}
	
	>>> _dict_master = _airtable.create_dictionary(url, 'SKU', reverse=False, 'Product Class', 'Item Status')
	>>> _dict_master
	{'12345': ['recxyzId', 'Regular', 'Live']}
	
**Create a list of JSON containing all data**
- Loop through all the pages and create a list containing all data on all columns.

Example:

	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master?filterbyformula={SKU}=12345')

	>>> _json_master = _airtable.get_json(url)
	>>> _json_master
	[{"id": "recxyzId","fields": {"SKU": "12345","Product Title": "Sample Title","PDS": ["recapABC"],"Product Class": "New","UPC": ["recABC"]}]
	
**Create a list of Airtable record IDs**
- Loop through all the pages and create a dictionary containing the {record ID: column data}.
- This is similar to create_dictionary but this limits the output to just 1 data point with the key being the record ID.
- The function also does not allow you to pass in query parameters.

Example:

	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> _json_master = _airtable.get_ids('Master', 'SKU')
	{'recxyzId': '12345', 'recabcId': '54321', 'recasdId': '74125'}

**Delete a list of Airtable record IDs**
- Delete records in Airtable based on record IDs.
- Please note that the function can only accept a maximum of 10 record IDs per request. This is an inherit limitation from Airtable.

Example:

	>>> # delete_ids(<table name>, <list_id>)
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> _airtable.delete_ids('Master', ['recxyzId', 'recabcId', 'recasdId'])

**Push data to Airtable (Patch / Post)**
- Sends either a patch or post request to the Airtable API.

|Status|Code|Reason|Description
|--|--|--|--|
|Success|200|OK|Request completed successfully.
|User Error|400|Bad Request|The request encoding is invalid; the request can't be parsed as a valid JSON.
|User Error|401|Unauthorized|Accessing a protected resource without authorization or with invalid credentials.
|User Error|402|Payment Required|The account associated with the API key making requests hits a quota that can be increased by upgrading the Airtable account plan.
|User Error|403|Forbidden|Accessing a protected resource with API credentials that don't have access to that resource.
|User Error|404|Not Found|Route or resource is not found. This error is returned when the request hits an undefined route, or if the resource doesn't exist (e.g. has been deleted).
|User Error|413|Request Entity Too Large|The request exceeded the maximum allowed payload size. You shouldn't encounter this under normal use.
|User Error|422|Invalid Request|The request data is invalid. This includes most of the base-specific validations. You will receive a detailed error message and code pointing to the exact issue.
|Server error|500|Internal Server Error|The server encountered an unexpected condition.
|Server error|502|Bad Gateway|Airtable's servers are restarting or an unexpected outage is in progress. You should generally not receive this error, and requests are safe to retry.
|Server error|503|Service Unavailable|The server could not process your request in time. The server could be temporarily unavailable, or it could have timed out processing your request. You should retry the request with backoffs.
	
- The function returns the status code

Example:
	
	>>> # push_data(url, payload, patch=True) *** if patch=True, send a patch request else if patch=False then send a post request
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master')
	>>> payload = {"records": [{"id": "recxyzId", "fields": {"Product Title": "New Sample Title"}}]}
	>>> req = _airtable.push_data(url, payload, patch=True)
	>>> req
	200

**Create a list of data in an Airtable column**
- Returns a list of row data based on the Airtable column.

Example:

	>>># create_list(url, column)
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master')
	>>> _list = _airtable.create_list(url, 'SKU')
	>>> _list
	['12345', '54321', '74125']

**Check Airtable table for duplicates based on column data**
- Returns a list of duplicate row data based on the Airtable column.

Example:

	>>># table_duplicate_check(url, baseName)
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> url = _airtable.create_url('Master')
	>>> _list_duplicates = _airtable.table_duplicate_check(url, 'SKU')
	>>> _list
	The following duplicates found on
	----table: Master
	----Data: ['74125']
	>>> # This means that the Master table contains 2 records with SKU 74125

**Cleans up a string that is formatted as a list**
- Returns a string that contains the following characters: ["'", "[", "]"]

Example:

	>>># clean_list_string(str)
	>>> _airtable = nj.airtableToolbox('abcdefg', 'xyzApiKey')
	>>> my_string = "['rec123456']"
	>>> my_string = _airtable.clean_list_string(my_string )
	>>> print(type(my_sting), my_sting)
	string	rec123456
	
---
### SlackToolbox Class
#### Overview
The class contains functions that help integrating with Slack easier.

---

**Importing the module**

	>>> from NathanJamesToolbox import NathanJamesToolbox as nj

**SlackToolboxClass Instance**

	_slack = nj.SlackToolbox(<api_key>, <channel_name>)
---
**Send a warning message in a slack channel**
- The function sends a message to a specific channel

Example:

	>>> # send_warning(pyfile=__file__, funcName=None, description=None)

	>>> import sys
	>>> _slack = nj.SlackToolbox('xyzAPIkey', 'General')
	>>> _slack.send_warning(__file__, 'check_stock_availability', 'Stock Availability for 74403 returned -33.0')

![Sample message](https://github.com/pfajardo-nj/NathanJamesToolbox/blob/master/assets/slack_error_sample.JPG)
	


## Authors

* **Paulo Fajardo** - *Initial work* - [github](https://github.com/pfajardo-nj/NathanJames-Automation-Script)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details