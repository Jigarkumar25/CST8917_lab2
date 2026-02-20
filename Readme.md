# Smart Image Analyzer with Azure Durable Functions

CST8917 -- Serverless Applications\
Lab 2

------------------------------------------------------------------------

## 1. Overview

This lab implements a Smart Image Analyzer using Azure Durable
Functions.

The system processes images uploaded to a Blob Storage container,
analyzes them in parallel using multiple activity functions, generates a
combined report, and stores the results in Azure Table Storage.

This implementation demonstrates:

-   Blob Trigger
-   Durable Orchestrator
-   Fan-Out / Fan-In pattern
-   Activity Functions
-   Azure Table Storage integration
-   HTTP Trigger for retrieving results

------------------------------------------------------------------------

## 2. Architecture and Workflow

### Processing Flow

1.  User uploads an image to the `images` Blob container.
2.  Blob Trigger detects the new file.
3.  Durable Orchestrator is started.
4.  Orchestrator executes four activities in parallel:
    -   analyze_colors
    -   analyze_objects
    -   analyze_text
    -   analyze_metadata
5.  Results are aggregated.
6.  A combined report is generated.
7.  Final results are stored in Azure Table Storage.
8.  Results are retrieved using an HTTP trigger.

------------------------------------------------------------------------

## 3. Durable Functions Pattern

### Fan-Out / Fan-In Pattern

The orchestrator runs multiple independent tasks in parallel:

``` python
tasks = [
    context.call_activity("analyze_colors", input),
    context.call_activity("analyze_objects", input),
    context.call_activity("analyze_text", input),
    context.call_activity("analyze_metadata", input)
]

results = yield context.task_all(tasks)
```

This demonstrates:

-   Parallel execution
-   Aggregation of activity results
-   Scalable serverless orchestration

------------------------------------------------------------------------

## 4. Project Structure

    lab2/
    │
    ├── function_app.py
    ├── requirements.txt
    ├── host.json
    ├── local.settings.example.json
    ├── test-function.http
    └── README.md

------------------------------------------------------------------------

## 5. Local Development Setup

### Requirements

-   Python 3.11
-   Azure Functions Core Tools v4
-   Node.js
-   Azurite (Azure Storage Emulator)

### Install Dependencies

``` bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Start Azurite

``` bash
npx azurite --silent --location .azurite --debug .azurite/debug.log --skipApiVersionCheck
```

### Start Function App

``` bash
func start
```

------------------------------------------------------------------------

## 6. Local Testing

### Upload Image

``` python
from azure.storage.blob import BlobServiceClient

conn = "UseDevelopmentStorage=true"
svc = BlobServiceClient.from_connection_string(conn)
container = svc.get_container_client("images")

with open("2.jpg", "rb") as f:
    container.upload_blob("2.jpg", f, overwrite=True)
```

This triggers the Durable workflow.

------------------------------------------------------------------------

## 7. Example Execution Output

After uploading an image, the logs show:

-   New image detected
-   Orchestrator started
-   Analyzing colors
-   Analyzing objects
-   Analyzing text
-   Analyzing metadata
-   Generating combined report
-   Results stored with ID

This confirms:

-   Blob Trigger execution
-   Parallel activity execution (Fan-Out)
-   Result aggregation (Fan-In)
-   Table Storage persistence

------------------------------------------------------------------------

## 8. HTTP Endpoint Testing

### Get latest results

GET http://localhost:7071/api/results

### Get limited results

GET http://localhost:7071/api/results?limit=3

### Get specific result

GET http://localhost:7071/api/results/{id}

Testing was performed using:

-   Browser
-   VS Code REST Client (test-function.http)

------------------------------------------------------------------------

## 9. Cloud Deployment Attempt

Deployment to Azure (West US 2) was attempted.

The deployment failed due to subscription quota restrictions:

SubscriptionIsOverQuotaForSku\
Current Limit (PremiumMV4 VMs): 0

Because compute quota is set to zero for the required SKU, Azure was
unable to provision the Function App in this subscription.

All required functionality was successfully validated locally using:

-   Azure Functions Core Tools
-   Azurite
-   Azure Table Storage Emulator

------------------------------------------------------------------------

## 10. Conclusion

This lab demonstrates:

-   Event-driven serverless architecture
-   Durable Functions orchestration
-   Fan-Out / Fan-In pattern
-   Azure Blob Storage integration
-   Azure Table Storage persistence
-   HTTP-based result retrieval

The complete workflow was successfully tested and verified locally.
