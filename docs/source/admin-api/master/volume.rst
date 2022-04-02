Volume
======

Create
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/createVol?name=test&capacity=100&owner=cfs&mpCount=3"


| Allocate a set of data partition and a meta partition to the user.
| Default create 10 data partition and 3 meta partition when create volume.
| CubeFS uses the **Owner** parameter as the user ID. When creating a volume, if there is no user named the owner of the volume, a user with the user ID same as **Owner** will be automatically created; if a user named Owner already exists in the cluster, the volume will be owned by the user. For details, please see:: doc: `/admin-api/master/user`

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory", "Default"
   
   "name", "string", "volume name", "Yes", "None"
   "capacity", "int", "the quota of vol, unit is GB", "Yes", "None"
   "volType", "int", "volType: 0: replica-volume, 1:ec-volume", "Yes", "0"
   "owner", "string", "the owner of vol, and user ID of a user", "Yes", "None"
   "mpCount", "int", "the amount of initial meta partitions", "No", "3"
   "size", "int", "the size of data partitions, unit is GB", "No", "120"
   "followerRead", "bool", "enable read from follower", "No", "false"
   "crossZone", "bool", "cross zone or not. If it is true, parameter *zoneName* must be empty", "No", "false"
   "zoneName", "string", "specified zone", "No", "default (if *crossZone* is false)"
   "cacheRuleKey", "string", "for ec volume", "No", "None"
   "ebsBlkSize", "int", "ec block size，unit is byte", "No", "8"
   "cacheCap", "int", "Cache capacity for ec volume,unit is GB", "No", "Yes for ec volume"
   "cacheAction", "int", "Cache scenario for ec volume，0-do not cache, 1-cache when reading, 2-cache when reading or writing", "No", "0"
   "cacheThreshold", "int", "When it is less than this value for ec volume, it is written to the cahce,unit is byte", "No", "10"
   "cacheTTL", "int", "Cache elimination time for ec volume，unit is day", "No", "30"
   "cacheHighWater", "int", "Threshold for cache elimination for ec volume，when this value is reached, trigger dp content elimination", "No", "80"
   "cacheLowWater", "int", "When this value is reached, dp content elimination will not be eliminated，", "No", "60"
   "cacheLRUInterval", "int", "Obsolescence detection cycle for ec volume，unit is minute", "No", "5"

Delete
-------------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/delete?name=test&authKey=md5(owner)"


Mark the vol status to MarkDelete first, then delete data partition and meta partition asynchronous, finally delete meta data from persist store, ec-volume can be deleted only if used size is zero.

While deleting the volume, the policy information related to the volume will be deleted from all user information.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "name", "string", "volume name"
   "authKey", "string", "calculates the 32-bit MD5 value of the owner field as authentication information"
   "forceDelVol", "bool", "Whether to force delete the volume，default false"

Get
---------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/getVol?name=test" | python -m json.tool


Show the base information of the vol, such as name, the detail of data partitions and meta partitions and so on.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "name", "string", "volume name"

response

.. code-block:: json

   {
       "Authenticate": false,
        "CacheAction": 0,
        "CacheCapacity": 0,
        "CacheHighWater": 80,
        "CacheLowWater": 60,
        "CacheLruInterval": 5,
        "CacheRule": "",
        "CacheThreshold": 10485760,
        "CacheTtl": 30,
        "Capacity": 10,
        "CreateTime": "2022-03-31 16:08:31",
        "CrossZone": false,
        "DefaultPriority": false,
        "DefaultZonePrior": false,
        "DentryCount": 0,
        "Description": "",
        "DomainOn": false,
        "DpCnt": 0,
        "DpReplicaNum": 16,
        "DpSelectorName": "",
        "DpSelectorParm": "",
        "FollowerRead": true,
        "ID": 706,
        "InodeCount": 1,
        "MaxMetaPartitionID": 2319,
        "MpCnt": 3,
        "MpReplicaNum": 3,
        "Name": "abc",
        "NeedToLowerReplica": false,
        "ObjBlockSize": 8388608,
        "Owner": "cfs",
        "PreloadCapacity": 0,
        "RwDpCnt": 0,
        "Status": 0,
        "VolType": 1,
        "ZoneName": "default"
   }



Stat
-------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/client/volStat?name=test"


Show the status information of volume.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "name", "string", "volume name"
   "version", "", "volume version, 0: replica-volume, 1: ec-volume, default 0"

response

.. code-block:: json

   {
       "CacheTotalSize": 0,
       "CacheUsedRatio": "",
       "CacheUsedSize": 0,
       "EnableToken": false,
       "InodeCount": 1,
       "Name": "abc-test",
       "TotalSize": 10737418240,
       "UsedRatio": "0.00",
       "UsedSize": 0
   }


Update
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/update?name=test&capacity=100&authKey=md5(owner)"

Increase the quota of volume, or adjust other parameters.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory"

   "name", "string", "volume name", "Yes"
   "authKey", "string", "calculates the 32-bit MD5 value of the owner field as authentication information", "Yes"
   "capacity", "int", "the quota of vol, has to be 20 percent larger than the used space, unit is GB", "Yes"
   "zoneName", "string", "update zone name", "Yes"
   "followerRead", "bool", "enable read from follower", "No"
   "cacheCap", "int", "Cache capacity for ec volume,unit is GB", "No", "Yes for ec volume"
   "cacheAction", "int", "Cache scenario for ec volume，0-do not cache, 1-cache when reading, 2-cache when reading or writing", "No"
   "cacheThreshold", "int", "When it is less than this value for ec volume, it is written to the cahce,unit is byte", "No"
   "cacheTTL", "int", "Cache elimination time for ec volume，unit is day", "No"
   "cacheHighWater", "int", "Threshold for cache elimination for ec volume，when this value is reached, trigger dp content elimination", "No"
   "cacheLowWater", "int", "When this value is reached, dp content elimination will not be eliminated，", "No"
   "cacheLRUInterval", "int", "Obsolescence detection cycle for ec volume，unit is minute", "No"
   "cacheRuleKey", "string", "modify cache rule", "No"
   "emptyCacheRule", "bool", "whether to empty cacheRule", "No"


List
--------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/list?keywords=test"

List all volumes information, and can be filtered by keywords.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory"

   "keywords", "string", "get volumes information which contains this keyword", "No"

response

.. code-block:: json

    [
       {
           "Name": "test1",
           "Owner": "cfs",
           "CreateTime": 0,
           "Status": 0,
           "TotalSize": 155515112832780000,
           "UsedSize": 155515112832780000
       },
       {
           "Name": "test2",
           "Owner": "cfs",
           "CreateTime": 0,
           "Status": 0,
           "TotalSize": 155515112832780000,
           "UsedSize": 155515112832780000
       }
    ]


Expand
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/expand?name=test&capacity=100&authKey=md5(owner) "

Expand the volume to the specified capacity

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory"

   "name", "string", "Volume name", "Yes"
   "authKey", "string", "Calculates the 32-bit MD5 value of the owner field as authentication information", "Yes"
   "capacity", "int", "Capacity after expaned,unit is GB", "Yes"


Shrink
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/shrink?name=test&capacity=100&authKey=md5(owner) "

Shrink the volume to the specified capacity

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory"

   "name", "string", "Volume name", "Yes"
   "authKey", "string", "Calculates the 32-bit MD5 value of the owner field as authentication information", "Yes"
   "capacity", "int", "Capacity after Shrinked,unit is GB", "Yes"


Create preload volume
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/dataPartition/createPreLoad?name=test&cacheTTL=60&capacity=100 "

create preload volume

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description", "Mandatory"

   "name", "string", "Volume name", "Yes"
   "cacheTTL", "int", "Elimination time for preload cache, unit is day, "Yes"
   "capacity", "int", "Capacity for preload cache,unit is GB", "Yes"
   "zoneName", "string", "Zone for preload data", "No"
