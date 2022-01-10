Clustermgr
===============

Statistics
--------

.. code-block:: bash

	curl "http://127.0.0.1:9998/stat" | python3 -m json.tool 

Show cluster statistics, including raft status, space status, volume information statistics, etc.

Response

.. code-block:: json

   {
	    "leader_host": "127.0.0.1:9998",
	    "raft_status": {
		"nodeId": 1,
		"term": 1,
		"vote": 1,
		"commit": 5,
		"leader": 1,
		"raftState": "StateLeader",
		"applied": 5,
		"raftApplied": 5,
		"transferee": 0,
		"peers": [
		    {
			"id": 2,
			"host": "127.0.0.1:10111",
			"match": 5,
			"next": 6,
			"state": "ProgressStateReplicate",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    },
		    {
			"id": 3,
			"host": "127.0.0.1:10112",
			"match": 5,
			"next": 6,
			"state": "ProgressStateReplicate",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    },
		    {
			"id": 1,
			"host": "127.0.0.1:10110",
			"match": 5,
			"next": 6,
			"state": "ProgressStateProbe",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    }
		]
	    },
	    "space_stat": {
		"total_space": 0,
		"free_space": 0,
		"used_space": 0,
		"writable_space": 0,
		"total_blob_node": 0,
		"total_disk": 0,
		"disk_stat_infos": [
		    {
			"idc": "z0",
			"total": 0,
			"total_chunk": 0,
			"total_free_chunk": 0,
			"available": 0,
			"readonly": 0,
			"expired": 0,
			"broken": 0,
			"repairing": 0,
			"repaired": 0,
			"dropping": 0,
			"dropped": 0
		    }
		]
	    },
	    "volume_stat": {
		"total_volume": 0,
		"idle_volume": 0,
		"can_alloc_volume": 0,
		"active_volume": 0,
		"lock_volume": 0,
		"unlocking_volume": 0
	    }
   }
   

Member Add
---------

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1, "host": "127.0.0.1:9998", "member_type": 2}' "http://127.0.0.1:9998/member/add" 
   
Add a cluster node, specify the node type, address and id.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Descriptions"

   "peer_id", "uint64", "raft node id，unique"
   "host", "string", "host address"
   "member_type", "uint8", "node type，1(leaner) and 2(normal)"
   
Member Remove
--------

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1}' "http://127.0.0.1:9998/member/remove"

Remove node by id.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Descriptions"

   "peer_id", "uint64", "raft node id，unique"
   
Leadership Transfer
-------------------

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1}' "http://127.0.0.1:9998/leadership/transfer"
   
Transfer leadership by node id.

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Descriptions"

   "peer_id", "uint64", "raft node id，unique"
   
Task Management
-----------------

.. csv-table::
   :header: "type", "key", "value"

   "Disk Repair", "disk_repair", "Enable/Disable"
   "Balance", "balance", "Enable/Disable"
   "Disk Drop", "disk_drop", "Enable/Disable"
   "Delete", "blob_delete", "Enable/Disable"
   "Shard Repair", "shard_repair",	"Enable/Disable"
   "Inspection", "vol_inspect", "Enable/Disable"
   
Task State

.. code-block:: bash

   curl http://127.0.0.1:9998/config/get?key=balance

Task Enable

.. code-block:: bash

   curl -X POST http://127.0.0.1:9998/config/set -d '{"key":"balance","value":"Enable"}' --header 'Content-Type: application/json'

Task Disable

.. code-block:: bash

   curl -X POST http://127.0.0.1:9998/config/set -d '{"key":"balance","value":"Disable"}' --header 'Content-Type: application/json'


