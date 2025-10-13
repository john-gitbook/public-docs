---
icon: file-binary
---

# API

The ICA APIs are hosted via a swagger-based UI in accordance to the OpenAPI 3.0 specification.

[API Reference](https://ica.illumina.com/ica/api/swagger/index.html)

[API Beginner Guide](https://help.ica.illumina.com/tutorials/api-introduction)

## CORS Support

The APIs support Cross-origin resources sharing (CORS).

***

## Cursor- versus Offset-based Pagination

Cursor-based pagination is fast, but does not support sorting, while offset-based pagination is slower, but supports sorting. **If no pagination method is selected** in the API call, the sort parameter is used to choose the default pagination.

* When no sorting is set, cursor based pagination will be used by default.
* When sorting is requested, offset-based pagination is used.

### Cursor-based pagination

Uses a pointer to a specific record and then returns the records following this record. It is able to handle data updates without skipping records or displaying them twice and at a faster speed, but can only handle small data jumps. There is also no easy option to sort or no way to skip pages, so if you want page X, all pages leading up to X must also be requested because each page contains the pointer to the next page.

Cursor-based pagination uses the parameter `pageSize` (amount of rows to return) in combination with `pageToken` (the cursor to get subsequent results). The parameter `remainingRecords` shows the number of records after the current cursor which are not shown. `remainingRecords` is an approximation only, as this number can increase or decrease while you are paging because records might be added or removed.

**Example**

Suppose a list of projects, with 2 projects returned per page (enter your API key in the YOUR\_API\_KEY part of the expression)

{% code fullWidth="false" %}
`````
'https://ica.illumina.com/ica/rest/api/projects?includeHiddenProjects=false&pageSize=2' \
-H 'accept: application/vnd.illumina.v3+json' \
-H 'X-API-Key: YOUR_API_KEY'````
`````
{% endcode %}

The response will contain a pointer to the next page `"nextPageToken": "A_STRING_OF_LETTERS_AND_NUMBERS"` You then use that pointer (the string of numbers and letters) to get the next 2 entries by replacing A\_STRING\_OF\_LETTERS\_AND\_NUMBERS by the returned next page token

{% code fullWidth="false" %}
```
curl -X 'GET' \
  'https://ica.illumina.com/ica/rest/api/projects?includeHiddenProjects=false&pageToken=A_STRING_OF_LETTERS_AND_NUMBERS_HERE&pageSize=2' \
  -H 'accept: application/vnd.illumina.v3+json' \
  -H 'X-API-Key: YOUR_API_KEY'
```
{% endcode %}

### Offset-based pagination

Returns records based on their position in a table. The data set is divided into pages and the offset parameter is used to determine which page to display. Offset pagination can quickly perform large jumps in data because you do not need to query previous pages and it allows custom sorting. The downside is that offset-based pagination is susceptible to missing records or displaying records twice when there have been updates while it is paginating.

In ICA, offset-based pagination is limited to 200.000 rows, and does not guarantee unique results across all pages as data can be updated during pagination.

Offset pagination uses the parameter `pageSize` (amount of rows to return) in combination with `pageOffset` (amount of rows to skip) and `totalItemCount` (The total number of records matching the search criteria). The larger your page size is, the slower the response time.

**Example**

`GET /api/projects/{projectId}/analyses` with `pageOffset` to 0. This will return the `totalItemCount`. If you want to sort the results, you can use `reference desc` for descending)

`curl -X 'GET' \'<SERVER>/ica/rest/api/projects/<PROJECT_ID>/analyses?pageOffset=2&pageSize=2&sort=reference%20desc' \ -H 'accept: application/vnd.illumina.v3+json' \ -H 'X-API-Key: <AUTH_KEY>'`

***

## Starting a Pipeline

If you want to start a pipeline, you need an **activation code** and the **analysis storage**.

* **Activation codes** are tokens which allow you to run your analyses and are used for accounting. ICA will automatically determine the best matching activation code, but this can be overwritten if needed.
* **Analysis storage** is used to determine the required storage size for your data and results. If this size is too small, the workflow will fail because of insufficient storage space, if it is too large, you are charged for storage which is not needed. In general, it is best to use the settings advised by the workflow designer.

For more details, use the [API Reference](https://ica.illumina.com/ica/api/swagger/index.html).

1. To obtain the analysis storage, perform `GET /api/analysisStorages` to obtain the list of available storages and copy the id of the size which you want to use for your pipeline.
2. ICA determines the best matching activation code.
3. Do `POST /api/projects/{projectId}/analysis:cwl` or `POST /api/projects/{projectId}/analysis:nextflow` with the obtained value for analysis storage.

***

## File output behavior

For [advanced output mappings](../project/p-flow/f-analyses.md#analysis-output-mappings), the overwrite behavior for file output can be set with the API parameter actionOnExist. This is a string, which supports the following options if a file or folder by that name already exists at that location.

* **Overwrite** (default) : The existing file is overwritten.
* **Rename** : The file is renamed by appending an incremental counter, before the extension.
* **Skip** : The new file will not be uploaded.

***

## Using data from your own S3 Storage

Prerequisite : have your S3 bucket and credentials ready _(public S3 buckets without credentials are not supported)_

1. (option 1) If you do not have credentials, then from the API perform **POST /api/storageCredentials** to store the storage credentials of your S3 bucket. The response will indicate the **id** (uuid) of the credentials.
2. (option 2) If you already have credentials which you want to use, perform a **GET /api/storageCredentials** to retrieve the list of storage credentials and note the **id** (uuid) which has been assigned to the credentials.
3. From the API perform a **POST /api/projects/{projectId}/analysis:cwl** (or Nextflow) with the required parameters for your analysis and

* `CreateCwlAnalysis > CwlanalysisInput > CwlAnalysisStructuredInput > AnalysisDataInput > AnalysisInputExternalData > url : s3://your_S3_address`
* `CreateCwlAnalysis > CwlanalysisInput > CwlAnalysisStructuredInput > AnalysisDataInput > AnalysisInputExternalData > type : pattern: s3`
* `CreateCwlAnalysis > CwlanalysisInput > CwlAnalysisStructuredInput > AnalysisDataInput > AnalysisInputExternalData > AnalysisS3DataDetails > storageCredentialsId: uuid from step 2`
* `CreateCwlAnalysis > CwlanalysisInput > CwlAnalysisStructuredInput > AnalysisDataInput > AnalysisInputExternalData > mountPath : location where the input file will be located on the machine that is running the pipeline.`\
  Preferably use a relative path, though an absolute path is allowed. The path must end with the file name which may differ from the original input data.

***

## Rate Limiting

The ICA API contains a rate limiter to prevent excessive requests from interfering with normal operations. If you get `error 429 Too many requests`, then your requests have exceeded the rate limit. To prevent this, please apply the following design principles during pipeline creation.

* Implement exponential backoff instead of a fixed retry time. Exponential backoff increases the delay between each retry attempt to increase the chance of the retry attempt succeeding without flooding the system with requests.
* Determine for each API call in your design if it is really necessary.
* If all else fails, contact Illumina technical support to help optimize your pipeline design and API requests.

***

## Duplicate Files on SampleCreationBatch

When running POST /api/projects/{projectId}/sampleCreationBatch, the input is checked to see if there are any duplicate files. The file in the lowest-level folder will be considered as the original file and duplicates in higher-level folders or the root folder will be ignored as input. Suppose you have the files `main\file1`, `main\file2` and `main\folder1\file1`. Then, the input will be `main\file2` because it has no duplicate, and `main\folder1\file1` because, even though it is a duplicate file, this instance is in the lowest level folder. `main\file1` will not be used because it is a duplicate in a higher-level folder.

***

## Analyses Ouputs Endpoint

API request  `GET/api/projects/{projectId}/analyses/{analysisId}/outputs` will return `analysisData` as part of the response. This is done for backwards compatibility and should not actively be used. For this reason, analysisData is not documented in the [API Reference](https://ica.illumina.com/ica/api/swagger/index.html) page.

***

## Starting Nextflow Pipeline with Custom Input

The API call `POST /api/projects/{projectId}/analysis:nextflowWithCustomInput` triggers the execution of a Nextflow pipeline with a custom input in YAML or JSON format, bypassing all input forms typically required by the pipeline. The input will directly provide all necessary data for the pipeline, overriding the default form-based input collection.

### Details

#### URL Parameters

* **projectId** (required): The unique identifier of the project where the pipeline is located. This  parameter must be provided in the URL.

#### Request Body

* The request body must contain the input that will be passed to the pipeline, overriding any default inputs.

#### Fields in analysisInput:

* **customInput** (required): A JSON/YAML object containing all the input data which the pipeline requires. The structure and content of the input depend on the specific pipeline, but should follow the expected format defined by the pipeline's logic. The custom input consists of a simple Map structure. These are key and value pairs used in your main.nf.

#### Considerations

* **Input Validation**: Ensure that all required fields for the pipeline are included in the inputJson object. If any critical input is missing or incorrect, the pipeline will not run successfully.
* **Pipeline-Specific Fields**: The structure and names of the fields in inputJson will vary depending on the pipeline. Refer to your pipeline documentation for specific details about the expected inputs.

### Example

#### main.nf

```
params.in = "test.test"
inputFile = file(params.txts[0])
 
println "maxagesum= $params.maxagesum"
println "ttt= $params.ttt"
println "txts= $params.txts"
println "notallowedcondition= $params.notallowedcondition"
println "group= $params.group"
 
for (groupinstance in params.group) {
    println "groupinstance= $groupinstance"
    println "groupinstance.info= $groupinstance.info"
    for (infoinstance in groupinstance.info) {
      println "infoinstance= $infoinstance"
      infofile = file(infoinstance)
      println "info file ===> name= ${infofile.getName()} uri= ${infofile.toUriString()} size= ${infofile.size()}"
 
    }
}
 
process writetooutput {
        input:
        file x from inputFile
 
        publishDir 'out', pattern: 'out/*', mode: 'move'
        output:
        path 'out/*'
        script:
        """
        #!/bin/bash
        printf "\n maxagesum was "
        printf '${params.maxagesum}'
        printf "\n ttt was "
        printf '${params.ttt}'
        printf "\n txts was "
        printf '${params.txts}'
        printf "\n notallowedcondition was "
        printf '${params.notallowedcondition}'
        printf "\n group was "
        printf '${params.group}'
 
        printf "\n "
        for ((n=0;n<3;n++))
        do
            date +"%H:%M:%S"
            sleep 10
        done
        printf in
        printf "create dir"
        mkdir out
        printf "copy start"
        cp $x out
        printf "copy done"
        """
}
```

#### Example string

To be provided with the request. The unescaped JSON is provided for legibility.

{% tabs %}
{% tab title="JSON" %}
```json
"{\"maxagesum\":null,\"ttt\":[\"test\"],\"txts\":[\"dummy.txt\"],\"notallowedcondition\":null,\"notallowedrole\":null,\"group\":[{\"role\":\"c\",\"conditions\":[\"1cancer\",\"covid1\"],\"age\":111,\"info\":[\"test.csv\"]},{\"role\":\"f\",\"conditions\":[\"cancer22\",\"covewd2\"],\"age\":null,\"info\":[\"test.csv\"]}]}"
```
{% endtab %}
{% tab title="Unescaped JSON" %}
```json
{
  "maxagesum":
     null,
  "ttt": [
    "test"
  ],
  "txts": [
    "dummy.txt"
  ],
  "group": [
    {
      "role": "c",
      "conditions": [
        "1cancer",
        "covid1"
      ],
      "age": 111,
      "info": [
        "test.csv"
      ]
    },
    {
      "role": "f",
      "conditions": [
        "cancer22",
        "covewd2"
      ],
      "info": [
        "test.csv"
      ]
    }
  ]
}
```
{% endtab %}
{% tab title="YAML" %}
```yaml
maxagesum: null
ttt:
  - test
txts:
  - dummy.txt
notallowedcondition: null
notallowedrole: null
group:
  - role: c
    conditions:
      - 1cancer
      - covid1
    age: 111
    info:
      - test.csv
  - role: f
    conditions:
      - cancer22
      - covewd2
    age: null
    info:
      - test.csv
```
{% endtab %}
{% endtabs %}

#### Request Body

```
{
  "userReference": "string",
  "pipelineId": "string",
  "tags": {
    "technicalTags": [
      "string"
    ],
    "userTags": [
      "string"
    ],
    "referenceTags": [
      "string"
    ]
  },
  "analysisStorageId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "outputParentFolderId": "string",
  "analysisOutput": [
    {
      "sourcePath": "string",
      "type": "FILE",
      "targetProjectId": "string",
      "targetPath": "string",
      "actionOnExist": "string"
    }
  ],
  "analysisInput": {
    "customInput": "string",
    "dataIds": [
      "string"
    ],
    "mounts": [
      {
        "dataId": "string",
        "mountPath": "string"
      }
    ],
    "externalData": [
      {
        "url": "string",
        "type": "string",
        "mountPath": "string",
        "s3Details": {
          "storageCredentialsId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
        },
        "basespaceDetails": {
          "workgroupId": "string",
          "extensions": "string",
          "pathPrefix": "string"
        }
      }
    ]
  }
}
```
