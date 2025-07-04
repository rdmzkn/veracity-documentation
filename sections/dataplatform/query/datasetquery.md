---
author: Veracity
description: This is a tutorial how to call API endpoints of Data Workbench with sample Python code.
---

# Query datasets using API

## API endpoints
To browse the api, go [here](https://developer.veracity.com/docs/section/api-explorer/76904bcb-1aaf-4a2f-8512-3af36fdadb2f/developerportal/dataworkbenchv2-swagger.json).
**See section Datasets**

### Baseurl
See [overview of base urls](https://developer.veracity.com/docs/section/dataplatform/apiendpoints)

### Authentication and authorization
To authenticate and authorize your calls, get your API key and a bearer token [here](../auth.md).

## Python code example

### Authenticate with service account

### Step 1: Authentication: Get Veracity token for service principle/service account
Service Account Id (Client Id), secret and api key (subscription key) for your workspace are defined under tab API Integration in data Workbench Portal.

```python
import requests
import json

# Token URL for authentication 
token_url = "https://login.microsoftonline.com/dnvglb2cprod.onmicrosoft.com/oauth2/token"
clientId =  <myServiceAccountId>
secret =   <myServiceAccountSecret>

# define the request payload    
payload = {"resource": "https://dnvglb2cprod.onmicrosoft.com/83054ebf-1d7b-43f5-82ad-b2bde84d7b75",
          "grant_type": "client_credentials",
          "client_id": clientId,
          "client_secret" :secret
          }
response = requests.post(token_url, data=payload)   
if response.status_code == 200:
        veracityToken = response.json().get("access_token")
else:
        print(f"Error: {response.status_code}")

```

### Query for datasets

```python
import requests
import json
 
mySubcriptionKey = <api key for service account>
workspaceId = < workspace id>

base_url = "https://api.veracity.com/veracity/dw/gateway/api/v2"
endpoint = f"/workspaces/{workspaceId}/datasets/query"
url = base_url + endpoint
 
headers = {
    "Content-Type": "application/json",
    "Ocp-Apim-Subscription-Key": mySubcriptionKey,
    "Authorization": f"Bearer {veracityToken}",
    "User-Agent": "python-requests/2.31.0"
}
payload = {
    "pageIndex": 1,
    "pageSize": 10,         
    "sortColumn": "CreatedOn",
    "sortDirection" : "Descending"     
}

try:
    response = requests.post(url, json=payload, headers=headers)
    response.raise_for_status()   
except requests.exceptions.RequestException as e:
    print(f"Error AS token: {e}")
    
result = response.json()
print(result)

```

### Query for data within a dataset
Schemas must have query filters defined to be able to query using these data query filters such as column Greater or Less than a value.

```python
import requests
import json
from datetime import datetime, timedelta
 
mySubcriptionKey = <api key for service account>
workspaceId = < workspace id>
datasetId = <dataset id>

base_url = "https://api.veracity.com/veracity/dw/gateway/api/v2"
endpoint = f"/workspaces/{workspaceId}/datasets/{datasetId}/query"
url = base_url + endpoint
 
headers = {
    "Content-Type": "application/json",
    "Ocp-Apim-Subscription-Key": mySubcriptionKey,
    "Authorization": f"Bearer {veracityToken}",
    "User-Agent": "python-requests/2.31.0"
}
# column filter is used to readuse number of columns in response, and query filter is used to filter on content
payload = {
    "pageIndex": 1,
    "pageSize": 100,         
    "sortColumn": "CreatedOn",
    "sortDirection" : "Descending",
    "columnFilter": ["Winddir", "Wind", "Irr", "Temp"]
    # to use query filter, the schema used for the dataset must support it    
}

try:
    response = requests.post(url, json=payload, headers=headers)
    response.raise_for_status()   
except requests.exceptions.RequestException as e:
    print(f"Error AS token: {e}")
    
result = response.json()
print(result)
```

### Query for dataset using SAS URI
**Get URI**
```python
import requests
import json
 
mySubcriptionKey = dbutils.secrets.get(scope="secrets", key="bkal_subKey")
workspaceId = "473ef07d-e914-4db4-959f-6eb737e391a6"
datasetId = "7f0e5cc3-f5c6-43cb-9414-634cb1ae7f4f"

base_url = "https://api.veracity.com/veracity/dw/gateway/api/v2"
endpoint = f"/workspaces/{workspaceId}/datasets/{datasetId}/sas?durationInMinutes=60&type=dfs"
url = base_url + endpoint
 
headers = {
    "Content-Type": "application/json",
    "Ocp-Apim-Subscription-Key": mySubcriptionKey,
    "Authorization": f"Bearer {veracityToken}",
    "User-Agent": "python-requests/2.31.0"
}
try:
    response = requests.get(url, headers=headers)
    response.raise_for_status()   
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
    
read_sas_uri = response.json()
```

**Use sas uri**
Each dataset is stored as a deltalake folder with transaction logs in folder _delta_logs and parquest file(s). You can read this using libraries.

```python
from azure.storage.filedatalake import DataLakeServiceClient
from urllib.parse import urlparse

# Your SAS URI from step 2
sas_url = read_sas_uri

# Parse the URL - it points to a flder
parsed_url = urlparse(sas_url)
account_url = f"{parsed_url.scheme}://{parsed_url.netloc}"
file_system_name = parsed_url.path.strip("/").split("/")[0]
folder_path = "/".join(parsed_url.path.strip("/").split("/")[1:]) if len(parsed_url.path.strip("/").split("/")) > 1 else ""
sas_token = parsed_url.query if parsed_url.query.startswith('sv=') else ''

# Create the service client
service_client = DataLakeServiceClient(account_url=account_url, credential=sas_token)
directoryClient = service_client.get_directory_client(file_system_name, folder_path) if folder_path else None

filesystem_client = service_client.get_file_system_client(file_system=file_system_name)

# List files in the folder
paths = filesystem_client.get_paths(path=folder_path, recursive=False)
pathsLst = list(paths)

# each dataset is stored as a deltalake folder with transaction logs in in folder _delta_logs and parquest file(s). You can read this using libraries.
print(f"\nContents of folder '{folder_path}':")
for path in pathsLst:
    print(f"- {path.name} (Directory: {path.is_directory})")   
```


## C# code example

`POST: {baseurl}/workspaces/{workspaceId}/datasets/query`

Body in request:
```csharp
{
  "isBaseDataset": true,
  "pageIndex": 0,
  "pageSize": 0,
  "sortColumn": "string",
  "sortDirection": "Ascending",
  "datasetName": "string",
  "tags": {},
  "createdAfter": "string",
  "createdBefore": "string",
  "schemaVersionIds": [
    "string"
  ]
}
```

Example
```
{ 
    "PageIndex": 1,
    "PageSize": 100,         
    "SortColumn": "CreatedOn",
    "SortDirection" : "Descending"     
}
```


### Query data within a dataset

```csharp
 public async Task QueryDataSet(string veracityToken, string workspaceId, string subscriptionKey)
 {
     string url = $"https://api.veracity.com/veracity/dw/gateway/api/v2/workspaces/{workspaceId}/datasets/query";

     var token = veracityToken;
               
     var postData = new Dictionary<string, string>
     {
         {"pageIndex", "1"},
         {"pageSize", "10"},
         {"sortColumn", "CreatedOn"},
         {"sortDirection",  "Descending"}
     };

     string jsonString = JsonConvert.SerializeObject(postData);
     HttpContent content = new StringContent(jsonString, Encoding.UTF8, "application/json");

     if (_httpClient == null)
         _httpClient = new HttpClient();

     _httpClient.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
     _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
     try
     {
         var result = await _httpClient.PostAsync(url,content);
         if (result.IsSuccessStatusCode)
         {
             var listOfDataset = result.Content.ReadAsStringAsync().Result;
         }
     }
     catch (Exception ex)
     {
     }     
 }
```

The request body contains the filters. Schemas must have query filters defined to be able to query using these data query filters such as column Greater or Less than a value.
```json
{
  "pageIndex": 0,
  "pageSize": 0,
  "queryFilters": [
    {
      "column": "string",
      "filterType": "List",
      "filterValues": [
        "string"
      ]
    }
  ],
  "columnFilter": [
    "string"
  ],
  "sorting": {
    "column": "string",
    "order": "Ascending"
  }
}
```

** Example **
The following filter will filter on timestamp and data channel ids. In addition return datapoints in descending order.
```csharp
{
"pageIndex": 1,
"pageSize": 500,
"columnFilter": [
"Timestamp", "DataChannelId", "Value", "_ValueNumeric"
],
"queryFilters": [
    {
      "column": "Timestamp",
      "filterType": "Greater",
      "filterValues": [
        "2024-05-01"
      ]
    },
    {
      "column": "Timestamp",
      "filterType": "Less",
      "filterValues": [
        "2024-05-03"
      ]
    },
    {
      "column": "DataChannelId",
      "filterType": "Equals",
      "filterValues": [
        "HAYS_FC-RUN1-TT01",
        "Emerson_700XA-MOL_PCT_0"
      ]
    }
  ],
"sorting": {
       "column": "Timestamp",
       "order": "Descending"
   }
}
```

