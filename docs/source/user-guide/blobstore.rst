BlobStore Manual
================

Build
------

.. code-block:: bash

   $ git clone https://github.com/chubaofs/blobstore.git
   $ cd blobstore
   $ source env.sh
   $ ./build.sh

If  ``build`` successful, the following executable files will be generated in the ``bin`` directory

    1. clustermgr
    2. blobnode
    3. allocator
    4. mqproxy
    5. access
    6. scheduler
    7. tinker
    8. worker
    9. cli

Cluster Deployment
------------------

Since modules are related to a certain extent, they need to be deployed in the following order to avoid deployment failure due to service dependencies.

Basic Environment
::::::::::::::::::

1. Platform Support

    `Linux`

2. Component

    `MongoDB <https://docs.mongodb.com/manual/tutorial/>`_

    `Kafka <https://kafka.apache.org/documentation/#basic_ops>`_

    `Consul <https://learn.hashicorp.com/tutorials/consul/get-started-install?in=consul/getting-started>`_ （each node）

Clustermgr
::::::::::::::::

At least three nodes are required to deploy clustermgr to ensure service availability.  When starting a node, you need to change the corresponding configuration file and ensure that the associated configuration between the cluster nodes is consistent.

1. Start Service（three-node cluster）

.. code-block:: bash

   nohup ./clustermgr -f clustermgr.conf
   nohup ./clustermgr -f clustermgr1.conf
   nohup ./clustermgr -f clustermgr2.conf

2. Example： ``clustermgr.conf``

.. code-block:: json

   {
        "bind_addr":":9998",
        "cluster_id":1,
        "idc":["z0"],
        "log": {
            "level": 0,
            "filename": "./run/cluster0.log"
        },
        "auditlog":{ #
            "logdir": "/tmp/clustermgr/"
        },
        "region": "test-region",
        "normal_db_path":"/tmp/normaldb0",
        "normal_db_option": { # auto create directory
            "create_if_missing": true
        },
        "code_mode_policies": [
            {"mode_name":"EC3P3","min_size":0,"max_size":1024,"size_ratio":0.2,"enable":true}
        ],
        "volume_mgr_config":{
            "volume_db_path":"/tmp/volumedb0",
            "volume_db_option": {
                "create_if_missing": true
            }
        },
        "cluster_config":{
            "init_volume_num":100
        },
        "raft_config": {
            "raft_db_path": "/tmp/raftdb0",
            "raft_db_option": {
                "create_if_missing": true
            },
            "server_config": {
                "nodeId": 1,
                "listen_port": 10110,
                "raft_wal_dir": "/tmp/raftwal0",
                "peers": {"1":"127.0.0.1:10110","2":"127.0.0.1:10111","3":"127.0.0.1:10112"}
            },
            "raft_node_config":{
                "node_protocol": "http://",
                "nodes": {"1":"127.0.0.1:9998", "2":"127.0.0.1:9999", "3":"127.0.0.1:10000"}
            }
        },
        "disk_mgr_config":{
            "rack_aware":false,
            "host_aware":false
        }
   }

Blobnode
::::::::::::::::

1. Create related directories under the compiled blobnode binary directory

.. code-block:: bash

   # This directory corresponds to the path of the configuration file
   mkdir -p ./run/disks/disk{1..6} # Each directory needs to be mounted on a disk to ensure the accuracy of data collection
   mkdir -p ./run/auditlog

2. Start Service

.. code-block:: bash

   nohup ./blobnode -f blobnode.conf

3. Example of  ``blobnode.conf``:

.. code-block:: json

   {
        "bind_addr": ":8899",
        "cluster_id": 1,
        "idc": "z0",
        "rack": "testrack",
        "host": "http://127.0.0.1:8899",
        "disks": [
            {"path": "./run/disks/disk1", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk2", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk3", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk4", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk5", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk6", "auto_format": true,"max_chunks": 1024}
        ],
        "clustermgr": {
            "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
        },
        "disk_config":{
            "disk_reserved_space_B": 1,   # for debug
            "must_mount_point": true      # for debug
        },
        "log":{ # running log
            "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
            "filename": "./run/blobnode.log"
        },
        "auditlog": {
            "logdir": "./run/auditlog"
        }
   }

Allocator
::::::::::::::::

1. It is recommended to deploy at least two nodes to ensure high availability for allocator.

2. Create an audit log directory and start the service

.. code-block:: bash

   mkdir /tmp/allocator
   nohup ./allocator -f allocator.conf

3. Example of ``allocator.conf``:

.. code-block:: json

   {
        "bind_addr": ":9100",
        "host": "http://127.0.0.1:9100", # replace with host ip
        "cluster_id": 1,
        "idc": "z0",
        "clustermgr": {
            "hosts": [
                "http://127.0.0.1:9998",
                "http://127.0.0.1:9999",
                "http://127.0.0.1:10000"
            ]
        },
        "log":{ # running log
            "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
            "filename": "/tmp/allocator.log" # running log file
        },
        "auditlog": {
            "logdir": "/tmp/allocator"
        }
   }

MQproxy
::::::::::::::::

1. Based on kafka，Need to create blob_delete_topic, shard_repair_topic, shard_repair_priority_topic corresponding topics in advance

.. code-block:: bash
   # example
   bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic blob_delete

2. Start Service

.. code-block:: bash

   # To ensure availability, each computer room `idc` needs to deploy at least one mqproxy node
   nohup ./mqproxy -f mqproxy.conf 10.84.28.170:9095

3. Example of ``mqproxy.conf``:

.. code-block:: json

   {
        "bind_addr": ":9600", # service port
        "cluster_id":1, # cluster id
        "clustermgr":{ # hosts of clustermgr
            "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
        },
        "mq":{
            "blob_delete_topic":"blob_delete",
            "shard_repair_topic":"shard_repair",
            "shard_repair_priority_topic":"shard_repair_prior",
            "msg_sender_cfg":{ # kafka ip
                "broker_list":["127.0.0.1:9092"]
            }
        },
        "service_register":{ # service info
            "host":"http://127.0.0.1:9600",
            "idc":"z0"
        },
        "log":{ # running log
          "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
          "filename": "/tmp/mqproxy.log" # running log file
        },
        "auditlog": {
            "logdir": "./auditlog/mqproxy"
        }
   }

Access
::::::::::::::::

1. Start Service

.. code-block:: bash

   # The access module is a stateless single node deployment
   nohup ./access -f access.conf

2. Example of ``access.conf``:

.. code-block:: json

   {
        "bind_addr": ":9500", # prot
        "log": { # running log
            "filename": "/tmp/access.log" # log file
        },
        "auditlog": {
            "logdir": "./auditlog/access"
        },
        "consul_agent_addr": "127.0.0.1:8500", # IP of consul service
        "service_register": {
            "consul_addr": "127.0.0.1:8500",
            "service_ip": "x.x.x.x" # access service IP
        },
        "stream": { # access server configuration
            "idc": "z0",
            "cluster_config": { # clustermgr config
                "region": "test-region", # region info
            }
        }
   }

Scheduler
::::::::::

1. Based on mongodb，need to create database.db_name, task_archive_store_db_name database

2. Start Service

.. code-block:: bash

   nohup ./scheduler -f scheduler.conf

3. Example of ``scheduler.conf``:

.. code-block:: json

   {
      "bind_addr": ":9800", # port
      "cluster_id": 1, # cluster id
      "clustermgr": { # hosts of clustermgr
        "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
      },
      "database": {
        "mongo": {
          "uri": "mongodb://127.0.0.1:27017"
        },
        "db_name": "scheduler", # database name
      },
      "task_archive_store_db": {#
        "mongo": {
          "uri": "mongodb://127.0.0.1:27017"
        },
        "db_name": "task_archive_store",
      },
      "log":{# running log
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/scheduler.log"
      },
      "auditlog": {
        "logdir": "./auditlog/scheduler"
      }
   }

Worker
:::::::

1. Start Service

.. code-block:: bash

   # At least one worker node is deployed in each computer room `idc`
   nohup ./worker -f worker.conf

3. Example of  ``worker.conf``:

.. code-block:: json

   {
      "bind_addr": ":9910", # port
      "cluster_id": 1,
      "service_register": { # service info
        "host": "http://127.0.0.1:9910",
        "idc": "z0"
      },
      "scheduler": {# scheduler config
        "host": "http://127.0.0.1:9800"
      },
      "dropped_bid_record": { # the reason of dropped blob id
        "dir": "./dropped"
      },
      "log":{
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/worker.log"
      },
      "auditlog": {
        "logdir": "./auditlog/worker"
      }
   }

Tinker
:::::::

1. Based on kafka，create shard_repair_conf.fail_topic_cfg.topic and viblob_delete_conf.fail_topic_cfg.topic in advance.

2. Based on mongodb，need to create database_conf.db_name.

3. Start service

.. code-block:: bash

   # Deploy at least one node to configure all partitions in the topic of consumption kafka
   nohup ./tinker -f tinker.conf

4. Example of  ``tinker.conf``:

.. code-block:: json

   {
      "bind_addr": ":9700", # port
      "cluster_id":1,
      "database_conf": {# mongodb
          "mongo": {
            "uri": "mongodb://127.0.0.1:27017"
          },
          "db_name": "tinker",
      },
      "shard_repair":{
           "broker_list":["127.0.0.1:9092"], # kafka host
           "priority_topics":[
               {
                    "priority":1, # Repair priority, the larger the value, the higher the priority
                    "topic":"shard_repair",
                    "partitions":[0]
               },
               {
                   "priority":2,
                   "topic":"shard_repair_prior",
                   "partitions":[0]
                }
           ],
           "fail_topic":{# Repair failed topic consumption configuration
                "topic":"shard_repair_failed",
                "partitions":[0]
           }
      },
      "blob_delete":{
            "broker_list":["127.0.0.1:9092"],
            "normal_topic":{
                "topic":"blob_delete",
                "partitions":[0]
            },
            "fail_topic":{# Deletefailed topic consumption configuration
                "topic":"fail_blob_delete",
                "partitions":[0]
            },
            "safe_delay_time_h":72, # expire
            "dellog":{
                "dir": "./delete_log"
            }
      },
      "clustermgr": { # hosts of clustermgr
          "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
       },
      "scheduler": {# host of scheduler
          "host": "http://127.0.0.1:9800"
      },
      "service_register":{ # service info
          "host":"http://127.0.0.1:9700",
          "idc":"z0"
      },
      "log":{
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/tinker.log"
      },
      "auditlog": {
        "logdir": "./auditlog/tinker"
      }
   }

Configuration Instructions
:::::::::::::::::::::::::::

1. clustermgr
    1) code_mode_policies
    Example:

    .. code-block:: json

        {
           "code_mode" : "EC3P3" # The specific strategy scheme, see the appendix
           "min_size" : 0 # Minimum upload blob size is 0
           "max_size" : 1024 # Maximum upload blob size is 01024
           "size_ratio" : 1 # Storage space ratio of different policies
           "enable" : true # Whether to enable this policy, true represents enable, false represents disable
        }



Test
------

Start Cli
:::::::::::

1. After starting ``cli`` on any machine in the cluster, set the access address by issuing the following command:

.. code-block:: bash

   ./cli

   # Set access address
   $> config set Key-Access-PriorityAddrs http://127.0.0.1:9500


Verification
::::::::::::::

.. code-block:: bash

   # Upload file， response the location of the file，（-d,  the actual content of the file）
   $> access put -v -d "test -data-"
   # Response
   {"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}

   # Download file，need the location of the file
   $> access get -v -l '{"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}'

   # Delete file，-l represent location；Confirm manually
   $> access del -v -l '{"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}'

Tips
-----

1.  For clustermgr and blobnode deployment failures, redeployment needs to clean up residual data to avoid registration disk failure or data display errors by issuing the following command:

.. code-block:: bash

   # blobnode example
   rm -f -r ./run/disks/disk*/.*
   rm -f -r ./run/disks/disk*/*

   # clustermgr example
   rm -f -r /tmp/raft*
   rm -f -r /tmp/volume*
   rm -f -r /tmp/clustermgr*
   rm -f -r /tmp/normal*

2. After all modules are successfully deployed, upload verification needs to be delayed for a period of time, waiting for the successful volume creation.

Appendix
---------

1. Code Mode Policies

.. csv-table::
   :header: "Type", "Descriptions"

   "EC15P12", "{N: 15, M: 12, L: 0, AZCount: 3, PutQuorum: 24, GetQuorum: 0, MinShardSize: 2048}"
   "EC6P6", "{N: 06, M: 06, L: 0, AZCount: 3, PutQuorum: 11, GetQuorum: 0, MinShardSize: 2048}"
   "EC16P20L2", "{N: 16, M: 20, L: 2, AZCount: 2, PutQuorum: 34, GetQuorum: 0, MinShardSize: 2048}"
   "EC6P10L2", "{N: 06, M: 10, L: 2, AZCount: 2, PutQuorum: 14, GetQuorum: 0, MinShardSize: 2048}"
   "EC12P4", "{N: 12, M: 04, L: 0, AZCount: 1, PutQuorum: 15, GetQuorum: 0, MinShardSize: 2048}"
   "EC3P3", "{N: 6, M: 3, L: 3, AZCount: 3, PutQuorum: 9, GetQuorum: 0, MinShardSize: 2048}"

*Where N: the number of data blocks, M: number of check blocks,, L: Number of local check blocks, AZCount: the count of AZ,  PutQuorum: (N + M) / AZCount + N <= PutQuorum <= M + N， MinShardSize: Minimum shard size, fill data into 0-N shards continuously, if the data size is less than MinShardSize*N, it will be aligned with zero bytes*, see `details <https://github.com/chubaofs/chubaofs/blobstore/common/codemode/codemode.go>`_ .