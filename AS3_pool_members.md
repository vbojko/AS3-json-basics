# AS3 use cases for LTM pool declarations

To better understand pool member declarations, we will use following declaration as base. This declaration creates a tenant "poolapp" with an HTTP virtual server that uses the pool **"web_pool"**. The declaration for this app looks like this:

``` json
{
	"class": "ADC",
	"schemaVersion": "3.20.0",
	"example_pool_declarations": {
		"class": "Tenant",
		"poolapp": {
			"class": "Application",
			"service": {
				"class": "Service_HTTP",
				"virtualAddresses": [
					"10.10.10.10"
				],
				"pool": "web_pool"
			},
			"web_pool": {
				"class": "Pool",
				"monitors": [
					"http"
				],
				"members": [{
					"servicePort": 80,
					"serverAddresses": [
						"192.0.1.10",
						"192.0.1.11"
					]
				}]
			}
		}
	}
}

```

Lets have a look at the members section of the declaration:

``` json
"members": [{
	"servicePort": 80,
	"serverAddresses": [
		"192.0.1.10",
		"192.0.1.11"
	]
}]
```
the value of **"members"** is an array with a single element, that is an object

``` json
"members": [ { <object values> } ]
```

within the object you have two configuration parameters: "servicePort" and "ServerAddresses" with it's individual values.

``` json
{
	"servicePort": 80,
	"serverAddresses": [
		"192.0.1.10",
		"192.0.1.11"
	]
}
```
This definition tells AS3 to create a group of servers in the pool "web_pool" that have the same config parameters.

## Adding an additional server that listens on port 80 to the pool

What needs to be done to add an additional server to the Web_pool? If the server shoudl have the same configuration parameters like all other pool members, then you need to add the IP address of the server to the "serverAddresses" array.

Let's add 192.0.1.12 that listens on port 80 to the web_pool

``` json
{
	"servicePort": 80,
	"serverAddresses": [
		"192.0.1.10",
        "192.0.1.11",
        "192.0.1.12"
	]
}
```

Why did I not specify the server port 80? Because the configuration parameter "servicePort":80 defines that all IP Addresses in "serverAddresses" will use the same service port, which is 80.

The complete pool declaration for web_pool looks like this:

``` json
"web_pool": {
	"class": "Pool",
    "monitors": [
		"http"
	],
	"members": [{
		"servicePort": 80,
		"serverAddresses": [
			"192.0.1.10",
            "192.0.1.11",
            "192.0.1.12"
		]
	}]
}
```

## adding an additional server that listens on port 8080 to the pool

Now I need to add an additional server to the pool (192.0.1.13), but the application listens on port 8080 instead of 80. How to do this?

Well in this case I cannot just add it to the "serverAddresses" array, because all IP addresses in this array share the same server port, which is 80.

Remeber, what is the data structure of "members"? The data structure of "members" is and array with a single element, that is an object:

``` json
	"members": [
        {
		"servicePort": 80,
		"serverAddresses": [ "192.0.1.10","192.0.1.11","192.0.1.12"]
        }
    ]
```

And an array can have multiple elements. So, in order to add another server to the pool, that listens on a diffeent prot than 80, you have to add another object to the "members" array. The object you have to add looks like this:

``` json
    {
	"servicePort": 8080,
	"serverAddresses": [ "192.0.1.13"]
    }
```

so, you new members array definition is:

``` json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.10",
				"192.0.1.11",
				"192.0.1.12"
			]
		},
		{
			"servicePort": 8080,
			"serverAddresses": ["192.0.1.13"]
		}
	]
```

Great, lets add two more server to the pool that listen on port 8080. use IP Addresses 192.0.1.14 and 192.0.1.15.

The declaration for "members" looks like this:

```json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.10",
				"192.0.1.11",
				"192.0.1.12"
			]
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
				"192.0.1.15"
			]
		}
	]
```

Basically what happens here is that we have two groups of servers, where each group share the same propertys.

## adding a pool member that listens on multiple ports

Let's assume you have one server with the IP 192.0.1.16 that has the application running on two ports 80 and 8080. How to declare this?
Well, you have to add the IP address 192.0.1.16 to both objects, like this:

``` json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.10",
				"192.0.1.11",
                "192.0.1.12",
                "192.0.1.16"
			]
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
                "192.0.1.15",
                "192.0.1.16"
			]
		}
	]
```

## Adding Priority groups

Another common scenario is Priority grouping. Priority grouping tells BIG-IP that send traffic to a group of pool members first and if this group is not available, then send it to the pool members with a lower priority group.

The name-value pair for priority groups is: "priorityGroup": (integer)
let's assume we want all servers with port 80 to have a priority group value of 20 and all pool members with port 8080 to have a priority group value of 10. 

the "members" declaration looks like this:

``` json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.10",
				"192.0.1.11",
                "192.0.1.12",
                "192.0.1.16"
			],
			"priorityGroup": 20
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
                "192.0.1.15",
                "192.0.1.16"
			],
			"priorityGroup": 10
		}
	]
```

## Changing admin status for a group of pool members

Per default the admin status of a pool member is "enabled". Let us change the admin status of all servers with port 8080 to "disabled".
The parameter to change the admin status of a pool member is: "adminState" and the possible values are "enable", "disable" and "offline"

To extend the existing configuration and  declare all pool members with port 8080 diabled use followong declaration:

``` json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.10",
				"192.0.1.11",
                "192.0.1.12",
                "192.0.1.16"
			],
			"priorityGroup": 20
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
                "192.0.1.15",
                "192.0.1.16"
			],
			"priorityGroup": 10,
			"adminState": "disable"
		}
	]
```


## Changing the admin status for a single pool member

Ok, let's assume you want to disable pool member **"192.0.1.10"** that listens on port 80. In this case the configuration of that pool member is different from all other pool members that listen on port 80, because it is disabled. That means you have to declare a separate object in the array **"members"** that has the pool member specific settings.

The declaration with a single disabled pool member looks like this:

``` json
	"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.11",
                "192.0.1.12",
                "192.0.1.16"
			],
			"priorityGroup": 20
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
                "192.0.1.15",
                "192.0.1.16"
			],
			"priorityGroup": 10
		},
		{
			"servicePort": 80,
			"serverAddresses": [ "192.0.1.10" ],
			"priorityGroup": 20,
			"adminState": "disable"
		}	
	]
```
If you compare the declarations, there are two steps:
* remove the IP address **"192.0.1.10"** from the object with service port 80
* create a new object with service port 80 that includes the server IP and has the **"adminState": "disable"** setting

so, the current service configuration looks like this:

``` json
{
	"class": "ADC",
	"schemaVersion": "3.20.0",
	"example_pool_declarations": {
		"class": "Tenant",
		"poolapp": {
			"class": "Application",
			"service": {
				"class": "Service_HTTP",
				"virtualAddresses": [
					"10.10.10.10"
				],
				"pool": "web_pool"
			},
			"web_pool": {
				"class": "Pool",
				"monitors": [
					"http"
				],
				"members": [{
			"servicePort": 80,
			"serverAddresses": [
				"192.0.1.11",
                "192.0.1.12",
                "192.0.1.16"
			],
			"priorityGroup": 20
		},
		{
			"servicePort": 8080,
			"serverAddresses": [
				"192.0.1.13",
				"192.0.1.14",
                "192.0.1.15",
                "192.0.1.16"
			],
			"priorityGroup": 10
		},
		{
			"servicePort": 80,
			"serverAddresses": [ "192.0.1.10" ],
			"priorityGroup": 20,
			"adminState": "disable"
		}	
	]
			}
		}
	}
}

```


