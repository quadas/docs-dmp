# Quadas DSP API for DMP
Quadas DSP provides APIs for DMPs to import segments into it.

# Prequisition
## Communication
The communication between DMP and API service must be through https. The request must be a HTTP 1.1 Post request.
Request content type must be "application/json". The API service response is always a json object.

## Model
A segment in a DMP is identified by it's DMP platform ID and DMP segment ID. The combination of the two values must be unique in a DMP.

# API
The API service provides the following APIs.
1.  **syncSegment**
1.  **getSegmentStatus**

## syncSegment
---
Import a DMP segment into Quadas DSP.
A DMP segment is associated to exact one Quadas DSP. The DSP segment can be imported into the same Quadas DSP segment for several times.

### Request
---
The request is an JSON objects with following attributes. 

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|token        |String     |token identify the request comes from which DMP. |
|dmpPlatformId|Long       |The DMP platform id of the DMP segment.  |
|dmpSegmentId |Long       |The DMP segment id of the DMP segment.   |
|name         |String     |The name of the segment. It will be displayed in DSP.  |
|downloadUrl  |String     |The URL which DSP can download the latest version of the DMP segment|
|visibilities |Array[Visiblity] |**Visibiliy** objects specify which platforms and advertisers can use the segment in DSP. |

A **Visibility** object contains the following attributes.

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|platformId   |Long       |Which DSP platform can access this DMP segment.|
|advertisers  |Array[Long]|It's **optional**. Which advertisers under the DSP platform can access the segment. If adverstisers is empty, all advertisers in the platform can access the segment. |

This is an example of the request.
```
{
  "token": "aklsjdflkabweoifoei",
  "dmpPlatformId": 1234,
  "dmpSegmentId": 5678,
  "name": "A group of people which are good at video game.",
  "downloadUrl": "http://good.storage.somewhere/segment.csv",
  "visibilities": [
    {
      "platformId": 9012
    },
    {
      "platformId": 9013,
      "visibilities": [1, 3, 5]
    }
  ]
}
```

### Response
---
The response of syncSegment contains following attributes.

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|result       |Int        |0 means the request is accepted. The downloading will be running in the background.|
|reason       |String     |The attribute is **optional**. When result is not 0, **reason** shows some detail about the error.  |

When an error happens while processing the request, the returned HTTP status code will be 500.

## getSegmentStatus
---
Get the current status of Quadas DSP segment which is associated to the specific DMP segment.

### Request
---
The request is an JSON objects with following attributes. 

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|token        |String     |token identify the request comes from which DMP. |
|dmpPlatformId|Long       |The DMP platform id of the DMP segment.  |
|dmpSegmentId |Long       |The DMP segment id of the DMP segment.   |

### Response
---

The response contains the following attributes.

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|result       |Int        |0 means the request is processed successfully. |
|reason       |String     |It's optinal. When result is not 0, **reason** shows some detail about the error.  |
|segmentStatus|SegmentStatus  |It presents only when result is 0. It includes the status of the segemnt.  |

SegmentStatus contains following attributes.

|Name         |Type       |Description         |
|-------------|-----------|--------------------|
|dmpPlatformId|Long       |The DMP platform id of the DMP segment.  |
|dmpSegmentId |Long       |The DMP segment id of the DMP segment.   |
|status       |Long       |The status code of the segment.  |
|downloadFailReason|String  |It's optinoal and only exists when **status** is DOWNLOADING_FAIL. |

Status values.

|Value    |Name       |Description|
|:-------:|-----------|-----------|
|-1       |UNKNOWN    |It's the initial status when a segment is just created.  |
|0        |OUT_OF_DATE|The DMP segment is newer than the DSP segment.  |
|1        |DOWNLOADING|The DMP segment is being downloaed to Quadas DSP. |
|2        |DOWNLOADING_FAIL |The DMP segment is not downloaded successfully. |
|3        |DOWNLOADED |The DMP segmnet has been downloaded into Quadas DSP.  |
|4        |UPDATING   |Quadas DSP is importing the downloaded DMP segment. |
|5        |UPDATED    |The DMP segment is imported into Quadas DSP.  |
