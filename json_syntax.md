# JSON Syntax

In order to understand AS3 declarations, you have to understand the json data structure first.

When you use automation tools, like Ansible, you have to understand the input you provide and the output data structure your tool you use gives you. There can be a difference if you use a bash script and "jq" or ansible with "jsonquery".

Before we go into the details of tools, we have to understand the different JSON datastructures and how to differentiate between them.

## JSON Data structures

JSON has two data structure: Objects and arrays

### JSON Objects

An Object is a set of name-value pairs.

An Object is enclosed in braces " { } ". Their name-value pairs are separated by coma and the name and value pair are separated by a colon ":" Names in an object are strings, whereas values may be any of the seven value types, including other object or an array.

For example, here an Object with a set of name-value pairs

``` json
{   
	"class": "AS3",
	"action": "deploy",
	"persist": true
}
```

### JSON Array

An Array is a list of values.

An Array is enclosed in brackets: " [ ] " Their values are seperated by a coma "," Each value in an array may be of a different type, including another array or an object.

For example, here a name-value pair of name: "members" and the value is an array that includes two elements. the first element is a name-value pair, where the name is "servicePort" and the value is a number, in this case "80". the second element is a name-value pair where name is "serverAddresses" and the value is another array with two elements.

``` json
{
	"members": [{
		"servicePort": 80,
		"serverAddresses": [
			"192.168.6.10",
			"192.168.6.11"
		]
	}]
}
```

### JSON value types

JSON has seven value types:
* string - string is enclosed in quotes
    ``` json
    {
        "name": "thisisastring"
    }
    ```
* number
    ``` json
    {
        "servicePort": 25
    }
    ```
* object - the value can be another object e.g "service": {}
    ``` json
    {
    	"Application1": {
            "class": "Application",
    		"service": {
                "class": "Service_HTTP",
                "pool": "web_pool"
            } 
    	}
    }
    ``` 
* array - the value can be another array
    ``` json
    {
    	"Application2": {
            "class": "Application",
    		"service": {
                "class": "Service_HTTP",
                "virtualAddresses": [
                    "10.0.6.10"
                ]
            }
    }
    ```
* true
    ``` json
    {
    	"persist": true
    }
    ```
* false
    ``` json
    {
        "persist": false
    }
    ```
* null - returns the value "null" if the requested object does not exist
    ``` json
    {

    }
    ```