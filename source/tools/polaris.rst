Polaris
=======

Polaris is a server that provides node discovery for Aergo server.

For bootstrapping new nodes, the npaddpeers configuration option can be used to connect to designated peers.

However, since Aergo version 0.11, the new Polarais server has added the ability to automatically connect to nodes belonging to the specified block chain without manually creating the node list information.

Features
--------

* Aergo nodes can query addresses of other Aergo nodes. In this case, the chain of the Aergo node and the chain of Polaris must be the same.
* Aergo nodes can register itself with Polaris. Polaris checks to see if it can connect to the Aergo node and adds it to the node list.
* One Polaris server per designated block chain

Building Polaris
----------------

This section describes how to build Polaris from source without using the Docker.
Polaris is available as a sub-module in the aergo project currently in version 0.11.

1. Get the source from github.com/aergoio/aergo.
2. Build the polaris executable with :code:`make polaris`.

Configuration
-------------

Four files are used to set Polaris behavior.

1. Polaris configuration file: Determines the overall operation of Polaris. It also specifies the path to other configuration related files.
2. Private key file: PK file to use for Polaris communication. It uses the same format as aergosvr's private key file.
3. Genesis file: Contains the chain information of nodes to be provided by Polaris. Use the same format as the genesis file used to initialize aergosvr.
4. Log configuration file: The file name is arglog.toml, and it uses the same format as the file used by aergosvr.

Configuration file creation and example
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create private key file
"""""""""""""""""""""""

It can be generated by aergocli using the keygen command.

::

	aergocli keygen mychain-polaris
	Wrote files mychain-polaris.{key,pub,id}.

Genesis file creation
"""""""""""""""""""""

.. code-block:: json

	{
	    "chain_id":{
	        "magic": "[insert an identifier string for your network]",
	        "public": false,
	        "mainnet": false,
	        "coinbasefee": "1000000000",
	        "consensus": "dpos"
	    },
	    "timestamp": 1548918000000000000,
	    "balance": {
	    },
	    "bps": [
	    ]
	}

Polaris configuration file
""""""""""""""""""""""""""

.. code-block:: toml

	authdir = "/blockchain/polaris/auth"       # base directory for files about authentication and authorization

	[rpc]
	netserviceaddr = "127.0.0.1"               # RPC access address. The default setting is 127.0.0.1, which allows RPC access only on the local machine and blocks RPC connections remotely.
	netserviceport = 8915

	[p2p]
	netprotocoladdr = "[real IP address]"      # An externally accessible IP address or domain name
	netprotocolport = 8916                     
	npbindaddr = ""                  
	npkey = "mychain-polaris.key"              # Location of private key file

	[polaris]
	allowprivate = true                        # Whether to allow the private address of the node's access address. Used when building Polaris for private chains operated within a test or private network.
	genesisfile = "[location of genesis file]" # Genesis file location
	enableblacklist = false                    # Whether to turn on blacklist or not. blacklist entries will be saved in <authdir>


Log configuration file
""""""""""""""""""""""

Refer to the `arglog documentation <../running-node/configuration.html#logging-options>`__.


Running Polaris
---------------

Using Docker
^^^^^^^^^^^^
::

	docker run -d -w /tools -v /blockchain/polaris:/tools -p 8916:8916 -p 8915:8915 --restart="always" --name polaris-node aergo/polaris polaris --home /tools --config /tools/polaris-conf.toml

Manually
^^^^^^^^

Manually build and run the live polaris executable in the following format:

::

	./polaris --config polaris-conf.toml

Connecting to Polaris
---------------------

To connect to a Polaris server, supply its address in the aergosvr configuration file.

See `Node Configuration <../running-node/configuration.html>`__ for details.

.. code-block:: toml

	[p2p]
	...
	npusepolaris= true
	npaddpolarises = [
	    "/ip4/192.168.0.2/tcp/8915/p2p/16Uiu2HAmJCmxe7CrgTbJBgzyG8rx5Z5vybXPWQHHGQ7aRJfBsoFs"
	]
    ...

Colaris
-------

Colaris is a client for Polaris RPC connection.

Building colaris
^^^^^^^^^^^^^^^^

Like Polaris, build as a sub-module of aergo.

1. Get the source from github.com/aergoio/aergo.
2. Build the executable with :code:`make colaris`.

Usage
^^^^^

It is the same interface as aergocli.

::

	./colaris [flags] <command> [[arg1]...]


Flags
"""""

1. :code:`-H <hostname>` Address to remote server when requesting. The default value is localhost (127.0.0.1)
2. :code:`-p <portnumber>` RPC port number, default is 8915

Commands
""""""""

:code:`node`: returns the actor state of Polaris.

:code:`current`: returns list of nodes registered in Polaris.

Example:

:: 

	ubuntu@mypolaris:/blockchain/polaris$ ./colaris current
	{
	 "total": 1,
	 "peers": [
	  {
	   "address": {
	    "address": "52.231.31.38",
	    "port": 7846,
	    "peerID": "16Uiu2HAmBfFABqQ2eWwNMv1A2WJCqVykgPS2sz72jrYTHeZgyors"
	   },
	   "connected": 1549526282,
	   "lastCheck": 1549526463
	  }
	 ]
	}

:code:`config`: set or get dynamic configurations of polaris. There is just a single config 'blacklist', in version 1.2.3. 

:code:`blacklist`: set or get blacklist configuration. blacklist entry can be set by peerID, ip address, cidr or combined. NOTE: it is NOT for security rather avoiding unintentional fork, since changing ip address or peer id is very easy. 

Example:

::

	ubuntu@mypolaris:/blockchain/polaris$ ./colaris config blacklist show
	ubuntu@mypolaris:/blockchain/polaris$ ./colaris config blacklist add --peerid 16Uiu2HAmDFV41vku39rsMtXBaFT1MFUDyHxXiDJrUDt7gJycSKnX --address "192.168.1.11"
	ubuntu@mypolaris:/blockchain/polaris$ ./colaris config blacklist add --address "192.168.1.11"
	ubuntu@mypolaris:/blockchain/polaris$ ./colaris config blacklist add --cidr "2001:0db8:0123:4567:89ab:cdef:1234:5678/96"
	ubuntu@mypolaris:/blockchain/polaris$ ./colaris config blacklist rm 2
