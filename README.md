## General Description

Track the horses and horseriders in the equidia's 30s rush for coursedereference:
- analyse horse race from multiple motion cameras;
- detect horseriders position and track the position;
- generate "Bounding box" for each rider

### horse and horseriders tracking state machine steps description

<img src = 'https://g.gravizo.com/svg? @startuml; (*) --> if "Some AC" then; -->[true] "activity 1"; if "" then; -> "activity 3" as a3; else; if "Other test" then; -left-> "activity 5"; else; --> "activity 6"; endif; endif; else; ->[false] "activity 2"; endif; a3 --> if "last test" then; --> "activity 7"; --> (*); else; -> "activity 8"; endif; @enduml' />

* [set-fps](./rush/SVC-164-set-fps.md) - Extend input with information on frames configuration.

* [set-segments](./rush/SVC-164-set-segments.md) - Extend input with information on segment configuration.

* [set-meta-regexps](./rush/SVC-164-set-meta-regexps.md) - Extend input with information on meta data extraction.

* [meta](./rush/SVC-144-meta.md) - Extract meta information from the video and save into `<*.meta>` file
basic
* [segments](./rush/SVC-152-segments.md) - Tranform the 30seconds videos into 15 seconds segments

* [frames](./rush/SVC-153-orchestrate-frame-extraction.md) - Tranform the 15 seconds segments into 5fps frames (so it is 5*15=75 frames per segment)

* extract-passfirst : pass input as output to set-cam-motion-conf step (with extract-passthrough)

* [set-cam-motion-conf](./rush/SVC-167-set-cam-conf.md) : Configure the cam-motion camera
Exporting the tracks to client-friendly format of bounding boxes called 'bboxTracks'
* set-prediction : Set basic configuration for prediction process

* [set-prediction-lambda](./rush/SVC-167-set-cam-conf.md) : Configure wich lambda to use for prediction

* [prediction](./rush/SVC-162-orchestrate-prediction.md) - Actial pridiction made here

* [set-multiview-tracking](./rush/SVC-167-set-cam-conf.md) : Configure the multiview-tracking

* [set-pitch-conf](./rush/SVC-158-orchestrate-tracking.md) - configure the pitch-conf

* [extract-cam-motion](./rush/SVC-158-orchestrate-tracking.md) - Determine how the camera is moving, and save it in the cam-motion folder 

* [detections-to-multiview-tracks](./rush/SVC-164-list-cameras.md) - Determine the tracks of the horses

* [bbox-tracks-from-multiview-tracks](./rush/SVC-166-tracks-to-trajectories) - Exporting the tracks to client-friendly format of bounding boxes called 'bboxTracks'

### Detailed description

To understand the step function execution of a rush, an example is taken below:

### Url of the step execution

https://eu-west-1.console.aws.amazon.com/states/home?region=eu-west-1#/executions/details/arn:aws:states:eu-west-1:170670752151:execution:dev-bri-states-coursederef:bfefae32-b39d-3cd5-3143-e4fa351d888b

### Input

To view the *Input* of your execution, expand the **Input** section under **Execution details**.

#### Input Example

```
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "eu-west-1",
      "eventTime": "2021-02-05T13:03:07.712Z",
      "eventName": "ObjectCreated:CompleteMultipartUpload",
      "userIdentity": {
        "principalId": "AWS:AIDAJYYQ2YQQXUC4BRT2C"
      },
      "requestParameters": {
        "sourceIPAddress": "122.177.12.38"
      },
      "responseElements": {
        "x-amz-request-id": "4K7N7J5PEW9KBTDY",
        "x-amz-id-2": "r0qAh9bpEoASL67wyl8K/+KqqSz4l0FxscZHlk8UxqgiZiXSKvYiJEOIZFddTf/OU0uK04j33B8t9eniTavUpcLIaYuPXDsX"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "exS3-v2--a30179dd272ad07d0aecd80d20377c7b",
        "bucket": {
          "name": "dev-bri-bucket-rush-eu-west-1",
          "ownerIdentity": {
            "principalId": "A3FY53OCZ6UPPQ"
          },
          "arn": "arn:aws:s3:::dev-bri-bucket-rush-eu-west-1"
        },
        "object": {
          "key": "equidia/20200421_coursesderef/2019-12-04_R1C5_A_testsmita2/footage/2019-12-04_R1C5_A_testsmita2.footage",
          "size": 166111316,
          "eTag": "12ba36e4ac787b9194ba5782400b8f1b-10",
          "versionId": "ETVZ7DOa6Y9RDCBVlhoWuBMK8xUQdC3B",
          "sequencer": "00601D4199F056F6B3"
        }
      }
    }
  ],
  "frames": {
    "fps": 5,
    "resolution": "1920x1080",
    "deinterlace": 1
  },
  "classification": {
    "labels": [
      "horsedetect"
    ],
    "classNames": [
      "horserider",
      "horse"
    ],
    "name": "equidia-horsedetect-fcos-20210120",
    "model": {
      "fcos": {
        "name": "equidia/horse-reftracking-detection/2021-01-21_12_28_08_6k-finetune-corrected-train-eval2races-aug-nms5_resnet50_coco_07.h5",
        "filenames": [
          "equidia/horse-reftracking-detection/2021-01-21_12_28_08_6k-finetune-corrected-train-eval2races-aug-nms5_resnet50_coco_07.h5"
        ]
      },
      "reid": {
        "name": "equidia/horse-reftracking-detection/reid-20201026",
        "filenames": [
          "equidia/horse-reftracking-detection/reid-20201026/saved_model.pb",
          "equidia/horse-reftracking-detection/reid-20201026/variables/variables.data-00000-of-00001",
          "equidia/horse-reftracking-detection/reid-20201026/variables/variables.index"
        ]
      },
      "bucket": "trn-bri-bucket-models-eu-west-1"
    },
    "type": "fcos",
    "fcos_resolution": "1333x750",
    "resolution": "1920x1080"
  },
  "predictionLambdaFunctionName": "dev-stj-predict"
}
```

#### Description of Input

* "name"

Bucket on [AWS S3](https://s3.console.aws.amazon.com/s3/home?region=us-east-1) where rush is stored.

In this example,

````
"name": "dev-bri-bucket-rush-eu-west-1",
````

* "key" : Path to rush file in declared s3 bucket.

````
"key": "equidia/20200421_coursesderef/2019-12-04_R1C5_A_testsmita2/footage/2019-12-04_R1C5_A_testsmita2.footage",
````

### Output
To view the *Output* of your execution, expand the **Output** section under [Execution details](#input).

#### Output Example

### Step's details
To view custom step details like Input, Output or Exeption, select Step in the **Visual workflow** and expand the Input, Output or Exeption Section.

#### Github repository

https://github.com/teamklap/sls-node-rush-salad-analysis
