# BATCH JOBS

## Goal

The purpose of this plugin is to leverage and utilize azure batch account in our nielsen application with ease. This plugin contains all the classes necessary to execute any task in the azure batch account.

## Table Of Contents

* [Understanding azure batch](./understanding-azure-batch)
* [Overview](./workflow-overview)
* [Setting up batch account](./setting-batch-account)
* [Writing plugins](./writing-plugins)
* [Common questions](./common-questions)
* [GOTCHAS](#gotchas)
* [Diagnosing a task](#diagnosing-a-run-that-is-stuck)
* [Design Discussions](#design-discussions)

### GOTCHAS

**Getting "HibernateException: No Session found for current thread" error when my event hook gets invoked?**
    
When a event hook is invoked there is no hiberanate session attached to that event so you will have you wrap your code that interacts with a domain object using "withNewTransaction". Eg:
    
```java 
    /**
     * Hook that will be invoked when the task is completed
     *
     * @param event
     * @param batchJobRunDocument
     * */
    void onTaskCompleteEvent(Map event, Map batchJobRunDocument) {
        String skuListId = batchJobRunDocument.referenceKeys.skuListId
        //appends DOE as evolution parameter    
        EvolutionParameter.withNewTransaction {
            List<Evolution> evolutions = lpoEvolutionService.appendDoeToEvolution(skuListId)
            evolutions.each { Evolution evolution ->
                evolution.save(flush: true)
            }
        }
    }
```

**Client application is not receiving any events even thought the task has completed**
    
This usually happens when the event hub namespace is not configured correctly. Please make sure that the "Firewalls and virtual networks" setting are configured properly for the event hub namespace resource correctly. There are 2 options to allow access:
* Allow all networks
* Allow selected networks
If for security reason we have enabled "Allow selected networks", then make sure your client ip is listed in the allowed list of IPS.
If the configure checks out, make sure that your batch account diagnostic is linked to the right event hub.


### DIAGNOSING A RUN THAT IS STUCK

Here the assumption is that the consuming application of this plugin has implemented diagnostic apis by extending the default controller implementation("BatchJobsControllerImpl"). The core logic of diagnostic apis are already embedded in the plugin, but because we cannot determine what java technology the consuming application might be in, and also which users should be allowed to execute those api - we leave the implementation details to the consuming application itself.
But having said that, the "BatchJobsControllerImpl" has defined following methods that could be leveraged to expose the diagnostic apis. Below we will provide an example of how to diagnose issue in studio application where the diagnostic apis have been implemented.

If you suspect that a batch task run is stuck or has not completed in the expected time frame then you can do following:

* Check the latest status of the task diretly from azure batch using following API: 
[GET] /platformservices/api/batchJobs/[ACCOUNT IDENTIFIER]/[JOB ID]/[TASK ID]/showTask
[EXAMPLE] /platformservices/api/batchJobs/analytics/lpoDoeGenerator/5cd30a6227369c2c1ccadedf/showTask
    The response should look like:
    ```json
    {
        "task": {
            "affinityInfo": null,
            "applicationPackageReferences": null,
            "authenticationTokenSettings": null,
            "commandLine": "/bin/bash -c \"mkdir Analytics_R_Source_Scripts && cp -r $(printenv AZ_BATCH_APP_PACKAGE_analytics_r_source_scripts)/* ./Analytics_R_Source_Scripts && Rscript \\\"Analytics_R_Source_Scripts/Templates/Analytics Template, DOE/LSSDOEBuilderStudio.R\\\" 45 5cc9a2ea273627e1a1e9f307\"",
            "constraints": {
                "maxTaskRetryCount": 0,
                "maxWallClockTime": {
                    "days": 10675199,
                    "fieldTypes": [
                        {
                            "name": "years"
                        },
                        {
                            "name": "months"
                        },
                        {
                            "name": "weeks"
                        },
                        {
                            "name": "days"
                        },
                        {
                            "name": "hours"
                        },
                        {
                            "name": "minutes"
                        },
                        {
                            "name": "seconds"
                        },
                        {
                            "name": "millis"
                        }
                    ],
                    "hours": 2,
                    "millis": 477,
                    "minutes": 48,
                    "months": 0,
                    "periodType": {
                        "name": "Standard"
                    },
                    "seconds": 5,
                    "values": [
                        0,
                        0,
                        0,
                        10675199,
                        2,
                        48,
                        5,
                        477
                    ],
                    "weeks": 0,
                    "years": 0
                },
                "retentionTime": {
                    "days": 2,
                    "fieldTypes": [
                        {
                            "name": "years"
                        },
                        {
                            "name": "months"
                        },
                        {
                            "name": "weeks"
                        },
                        {
                            "name": "days"
                        },
                        {
                            "name": "hours"
                        },
                        {
                            "name": "minutes"
                        },
                        {
                            "name": "seconds"
                        },
                        {
                            "name": "millis"
                        }
                    ],
                    "hours": 0,
                    "millis": 0,
                    "minutes": 0,
                    "months": 0,
                    "periodType": {
                        "name": "Standard"
                    },
                    "seconds": 0,
                    "values": [
                        0,
                        0,
                        0,
                        2,
                        0,
                        0,
                        0,
                        0
                    ],
                    "weeks": 0,
                    "years": 0
                }
            },
            "containerSettings": null,
            "creationTime": {
                "afterNow": false,
                "beforeNow": true,
                "centuryOfEra": 20,
                "chronology": {
                    "zone": {
                        "fixed": true,
                        "id": "UTC"
                    }
                },
                "dayOfMonth": 8,
                "dayOfWeek": 3,
                "dayOfYear": 128,
                "equalNow": false,
                "era": 1,
                "hourOfDay": 16,
                "millis": 1557334627748,
                "millisOfDay": 61027748,
                "millisOfSecond": 748,
                "minuteOfDay": 1017,
                "minuteOfHour": 57,
                "monthOfYear": 5,
                "secondOfDay": 61027,
                "secondOfMinute": 7,
                "weekOfWeekyear": 19,
                "weekyear": 2019,
                "year": 2019,
                "yearOfCentury": 19,
                "yearOfEra": 2019,
                "zone": {
                    "fixed": true,
                    "id": "UTC"
                }
            },
            "dependsOn": null,
            "displayName": null,
            "eTag": "0x8D6D3D6348AE99A",
            "environmentSettings": null,
            "executionInfo": {
                "containerInfo": null,
                "endTime": null,
                "exitCode": null,
                "failureInfo": null,
                "lastRequeueTime": null,
                "lastRetryTime": null,
                "requeueCount": 0,
                "result": null,
                "retryCount": 0,
                "startTime": {
                    "afterNow": false,
                    "beforeNow": true,
                    "centuryOfEra": 20,
                    "chronology": {
                        "zone": {
                            "fixed": true,
                            "id": "UTC"
                        }
                    },
                    "dayOfMonth": 8,
                    "dayOfWeek": 3,
                    "dayOfYear": 128,
                    "equalNow": false,
                    "era": 1,
                    "hourOfDay": 17,
                    "millis": 1557335532192,
                    "millisOfDay": 61932192,
                    "millisOfSecond": 192,
                    "minuteOfDay": 1032,
                    "minuteOfHour": 12,
                    "monthOfYear": 5,
                    "secondOfDay": 61932,
                    "secondOfMinute": 12,
                    "weekOfWeekyear": 19,
                    "weekyear": 2019,
                    "year": 2019,
                    "yearOfCentury": 19,
                    "yearOfEra": 2019,
                    "zone": {
                        "fixed": true,
                        "id": "UTC"
                    }
                }
            },
            "exitConditions": null,
            "id": "5cd30a6227369c2c1ccadedf",
            "lastModified": {
                "afterNow": false,
                "beforeNow": true,
                "centuryOfEra": 20,
                "chronology": {
                    "zone": {
                        "fixed": true,
                        "id": "UTC"
                    }
                },
                "dayOfMonth": 8,
                "dayOfWeek": 3,
                "dayOfYear": 128,
                "equalNow": false,
                "era": 1,
                "hourOfDay": 16,
                "millis": 1557334627748,
                "millisOfDay": 61027748,
                "millisOfSecond": 748,
                "minuteOfDay": 1017,
                "minuteOfHour": 57,
                "monthOfYear": 5,
                "secondOfDay": 61027,
                "secondOfMinute": 7,
                "weekOfWeekyear": 19,
                "weekyear": 2019,
                "year": 2019,
                "yearOfCentury": 19,
                "yearOfEra": 2019,
                "zone": {
                    "fixed": true,
                    "id": "UTC"
                }
            },
            "multiInstanceSettings": null,
            "nodeInfo": {
                "affinityId": "TVM:tvm-587366007_80-20190508t171044z",
                "nodeId": "tvm-587366007_80-20190508t171044z",
                "nodeUrl": "https://duseanalytics01ba.eastus.batch.azure.com/pools/lpoDoeGeneratorDynamicPool/nodes/tvm-587366007_80-20190508t171044z",
                "poolId": "lpoDoeGeneratorDynamicPool",
                "taskRootDirectory": "workitems/lpoDoeGenerator/job-1/5cd30a6227369c2c1ccadedf",
                "taskRootDirectoryUrl": "https://duseanalytics01ba.eastus.batch.azure.com/pools/lpoDoeGeneratorDynamicPool/nodes/tvm-587366007_80-20190508t171044z/files/workitems/lpoDoeGenerator/job-1/5cd30a6227369c2c1ccadedf"
            },
            "outputFiles": [
                {
                    "destination": {
                        "container": {
                            "containerUrl": "https://duseanalytics01bastg.blob.core.windows.net/5cd30a6227369c2c1ccadedf?sp=w&sr=c&sv=2017-07-29&se=2019-05-09T16%3A57%3A07Z&sig=Kemu%2Ba3fXnsiPmnNJ4f0FC2RfatSjQ7DIZ7JdedEfZY%3D",
                            "path": null
                        }
                    },
                    "filePattern": "../**/*.*",
                    "uploadOptions": {
                        "uploadCondition": "taskcompletion"
                    }
                }
            ],
            "previousState": "active",
            "previousStateTransitionTime": {
                "afterNow": false,
                "beforeNow": true,
                "centuryOfEra": 20,
                "chronology": {
                    "zone": {
                        "fixed": true,
                        "id": "UTC"
                    }
                },
                "dayOfMonth": 8,
                "dayOfWeek": 3,
                "dayOfYear": 128,
                "equalNow": false,
                "era": 1,
                "hourOfDay": 17,
                "millis": 1557335531729,
                "millisOfDay": 61931729,
                "millisOfSecond": 729,
                "minuteOfDay": 1032,
                "minuteOfHour": 12,
                "monthOfYear": 5,
                "secondOfDay": 61931,
                "secondOfMinute": 11,
                "weekOfWeekyear": 19,
                "weekyear": 2019,
                "year": 2019,
                "yearOfCentury": 19,
                "yearOfEra": 2019,
                "zone": {
                    "fixed": true,
                    "id": "UTC"
                }
            },
            "resourceFiles": [
                {
                    "autoStorageContainerName": null,
                    "blobPrefix": null,
                    "fileMode": null,
                    "filePath": "inputs/skus.json",
                    "httpUrl": "https://duseanalytics01bastg.blob.core.windows.net/5cd30a6227369c2c1ccadedf-input-files/skus.json?sp=r&sr=b&sv=2017-07-29&se=2019-05-09T16%3A57%3A07Z&sig=sBC5d%2B9YCLLIeMLIHzxAegHcbkkZ%2B1RuF%2F7A5QhZ7o4%3D",
                    "storageContainerUrl": null
                },
                {
                    "autoStorageContainerName": null,
                    "blobPrefix": null,
                    "fileMode": null,
                    "filePath": "inputs/sku-list-summary.json",
                    "httpUrl": "https://duseanalytics01bastg.blob.core.windows.net/5cd30a6227369c2c1ccadedf-input-files/sku-list-summary.json?sp=r&sr=b&sv=2017-07-29&se=2019-05-09T16%3A57%3A07Z&sig=oTyakwCXOsVz4XnVM0jRfwEN0DFOqsArQDX%2F3DbrAv0%3D",
                    "storageContainerUrl": null
                },
                {
                    "autoStorageContainerName": null,
                    "blobPrefix": null,
                    "fileMode": null,
                    "filePath": "inputs/sku-list-rules.json",
                    "httpUrl": "https://duseanalytics01bastg.blob.core.windows.net/5cd30a6227369c2c1ccadedf-input-files/sku-list-rules.json?sp=r&sr=b&sv=2017-07-29&se=2019-05-09T16%3A57%3A07Z&sig=rfxwfcZL0ojNjndWlDpjVQWrWW7oBSVr9Bb7mOB0ezE%3D",
                    "storageContainerUrl": null
                }
            ],
            "state": "running",
            "stateTransitionTime": {
                "afterNow": false,
                "beforeNow": true,
                "centuryOfEra": 20,
                "chronology": {
                    "zone": {
                        "fixed": true,
                        "id": "UTC"
                    }
                },
                "dayOfMonth": 8,
                "dayOfWeek": 3,
                "dayOfYear": 128,
                "equalNow": false,
                "era": 1,
                "hourOfDay": 17,
                "millis": 1557335532192,
                "millisOfDay": 61932192,
                "millisOfSecond": 192,
                "minuteOfDay": 1032,
                "minuteOfHour": 12,
                "monthOfYear": 5,
                "secondOfDay": 61932,
                "secondOfMinute": 12,
                "weekOfWeekyear": 19,
                "weekyear": 2019,
                "year": 2019,
                "yearOfCentury": 19,
                "yearOfEra": 2019,
                "zone": {
                    "fixed": true,
                    "id": "UTC"
                }
            },
            "stats": null,
            "url": "https://duseanalytics01ba.eastus.batch.azure.com/jobs/lpoDoeGenerator/tasks/5cd30a6227369c2c1ccadedf",
            "userIdentity": {
                "autoUser": {
                    "elevationLevel": "admin",
                    "scope": "task"
                },
                "username": null
            }
        }
    }
    ```

    In the api response check for the value set to "state" property. In the given example you can see that the current state value is set to "running". If in the consuming app you donot see this state change then first check when the task transitioned to this state which is stored in "stateTransitionTime" property. Since the plugin uses event hub(or polling if explicitly set to it) it usually has some latency of about a min, so if the event changed just a min ago then give it few more minutes for the events/messages to propagate to the consuming application.
    
    If the status change happened way before, then next thing to do is to check if we ever received the event in the cliet application.
    
* Check the events received by the application using following API:
[GET] /platformservices/api/batchJobs/[ACCOUNT IDENTIFIER]/[JOB ID]/[TASK ID]/showLocks
[EXAMPLE] /platformservices/api/batchJobs/analytics/lpoDoeGenerator/5cd30a6227369c2c1ccadedf/showLocks
    The response should look like:    
    ```json
    {
        "locks": [
            {
                "partitionKey": "5cd30a6227369c2c1ccadedf",
                "rowKey": "LPO-DEEWEN_duseanalytics01ba_lpoDoeGenerator_5cd30a6227369c2c1ccadedf_TaskStartEvent",
                "timestamp": "2019-05-08T17:14:01Z",
                "accountId": "duseanalytics01ba",
                "clientGroupId": "LPO-DEEWEN",
                "jobId": "lpoDoeGenerator",
                "taskId": "5cd30a6227369c2c1ccadedf",
                "operationName": "TaskStartEvent"
            }
        ]
    }
    ```
    
    In the above example, we see that the consuming application did receive the "TaskStartEvent", if this record were missing then we know for fact that the application did not receive the event/message associated with the running task.

* If you find that the rights events have not been triggered for a task run then you can manually trigger state change by the application using following API:
    [GET] /platformservices/api/batchJobs/[ACCOUNT IDENTIFIER]/[JOB ID]/[TASK ID]/changeState
    [EXAMPLE] /platformservices/api/batchJobs/analytics/lpoDoeGenerator/5cd30a6227369c2c1ccadedf/changeState
    
    
### Design Discussions    

* Should batch job manager library should be made into its own web service?
    
    | Advantages | Disadvantages |
    | --- | --- |
    | we can leverage the latest sdks that azure provides to interact with batch account | Adds one more point of failure, and one extra request hop which can be costly based on task |
    |  |It becomes a very thin service that does nothing but forward the request up to the batch, more work for devOps in terms of maintaining sets of these web services |
   

* Is event hub the right approach to gather state change of a task? Should we be using Azure Service Bus instead?
   This [article](http://microsoftintegration.guru/2015/03/03/azure-event-hubs-vs-azure-messaging/) is a good read which describes the use case scenario for both. Extracted from the article below:
  
    >From a target use case perspective if we consider some of our typical enterprise integration patterns then if you are implementing a pattern which uses a Command Message, or a Request/Reply Message then you probably want to use Azure Service Bus Messaging.  RPC patterns can be implemented using Request/Reply messages on Azure Service Bus using a response queue.  These are really about ESB and EAI style messaging patterns where you want to send messages between applications and probably want to use other features such as property based routing.
    >Azure Event Hubs is more likely to be used if you’re implementing patterns with Event Messages and you want somewhere reliable to send them that is capable of dealing with a massive scale but will allow you to do stuff with the events out of process.
    >With these core target use cases in mind it is easy to see where the scale differences come into play.  For messaging it’s about one application telling one or more apps to DO SOMETHING or GIVE ME SOMETHING.  The alternative is that in eventing the applications are saying SOMETHING HAS HAPPENED.  When you consider this in typical application scenarios and you put events into the telemetry and logging space you can quickly see that the SOMETHING HAS HAPPENED scenario will produce a lot more traffic than the other.
    
    From the above extract, its fair to conclude the we can use Event Hub to capture events regarding "WHAT HAS HAPPENED" with a task which is always a one way traffic from batch to consuming application. However we have not yet solved for two way interaction mechanism between a wroker process and invoker/consumer application for which sercive bus would be the ideal solution(which we will likely use and define a mechanism to do so soon).

    Now to the question that sould we use just one solution? Probably not because the transition states of the worker process is best left to be determined/dictated by the OS/VM where the process is actually executing where Event Hub comes to play with diagnostic feature. Leaving this responsibily to the worker process can open window to more issues:
    * if they donot invoke the message correctly or do not implement them at all. If its an batch application that cannot be mofied for eg an executable binary file
    * if an exception in the process results in it being unable to send those messages out(we could use timeout mechanism but that not a good solution)

* How should we support custom events from tasks such as send progress info?
* How long should the worker process resources live in the blob storage/how should we handle cleanup?
* How can worker process communicate back to the invoker application?

