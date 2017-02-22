# Request Types

## Upgrade

The "upgrade" request type allows a client to add a role to a session, so that authenticated requests can be performed from that point forward.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'upgrade',
  'user' : '5551234567',
  'password' : 'a5a2df...'
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "upgrade"
user | string | true | The phone number of the user. Or the email address of the crm user.
password | string | true | The SHA256 hash of the users password. Or the google access token of the crm user.
 
### Response Format 

```javascript

// Truncated Response Object
{
	'session' : {
		...
		'roles' : ['newRole', 'other role'] 
	},
	'response' : {
		...
		'code' : 'ok', 
		...
	}
	...
}
```

Key | Details
--- | ------- 
session.roles | This array now reflects any new role granted to the session.
response.code | If set to "ok", the session upgrade was successful.

<aside class="warning">
All requests (and subscriptions) made previous to the upgrade will continue to process with the roles that were active in the session at the time of the original request. 
</aside> 


## Downgrade

The "downgrade" request type allows a client to remove roles from a session, effectively downgrading the session to an anonymous/public one with no privileges.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'downgrade'
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "downgrade"
 
### Response Format 

```javascript

// Truncated Response Object
{
	'session' : {
		...
		'roles' : [] 
	},
	'response' : {
		...
		'code' : 'ok', 
		...
	}
	...
}
```

Key | Details
--- | ------- 
session.roles | This array is now empty, reflecting the removal of all roles.
response.code | If set to "ok", the session downgrade was successful.

<aside class="warning">
All requests (and subscriptions) made previous to the downgrade will continue to process with the roles that were active in the session at the time of the original request. 
</aside> 

## Create

The "create" request type allows a client to create a resource with specific characteristics, provided that the client's session has the role privilege necessary to do so. This request-type is analagous to the POST method in HTTP.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'create',
  'topic' : 'users' // An example of a topic
  'params' : {...}
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "create"
topic | string | true | The name of the topic that the request concerns. 
params | object | true | An object containing zero or more key-value pairs of parameters related to the resource being created. 
  
### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok', 
		'payload' : {...} 
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", the resource was created successfully.
response.payload | An object containing zero or more key-value pairs representing the resource created.

## Read

The "read" request type allows a client to retrieve a resource with specific characteristics, provided that the client's session has the role privilege necessary to do so. This request-type is analagous to the GET method in HTTP.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'read',
  'topic' : 'users' // An example of a topic
  'criteria' : {...}
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "read"
topic | string | true | The name of the topic that the request concerns. 
criteria | object | true | An object containing zero or more key-value pairs that represent the criteria to filter resources on. The service will return all resources that match all criteria.


### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok', 
		'payload' : {...} 
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", one or more resources were retrieved successfully.
response.payload | An object containing zero or more key-value pairs representing one of the resources retrieved.

<aside class="warning">
A single 'read' request may yield many response messages.
</aside> 


## Update

The "update" request type allows a client to modify a resource (or resources) matching with specific characteristics, with new characteristics, provided that the client's session has the role privilege necessary to do so. This request-type is analagous to the PUT method in HTTP.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'read',
  'topic' : 'users' // An example of a topic
  'criteria' : {...},
  'params' : {...}
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "read"
topic | string | true | The name of the topic that the request concerns. 
criteria | object | true | An object containing zero or more key-value pairs that represent the criteria to filter resources on. The service will perform modifications on all resource that match all criteria.
params | object | true | An object containing zero or more key-value pairs of parameters that should be used to modify the objects that match the criteria.

### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok', 
		...
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", one or more resources were modified successfully.


## Delete

The "delete" request type allows a client to delete a resource with specific characteristics, provided that the client's session has the role privilege necessary to do so. This request-type is analagous to the DELETE method in HTTP.

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'read',
  'topic' : 'users' // An example of a topic
  'criteria' : {...}
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "read"
topic | string | true | The name of the topic that the request concerns. 
criteria | object | true | An object containing zero or more key-value pairs that represent the criteria to filter resources on. The service will delete all resources that match all criteria.


### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok',
		...
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", one or more resources were deleted successfully.


## Subscribe


The "subscribe" request type allows a client to subscribe to all resources with specific characteristics. The service will immediately return all matching resources that the session has the privileges to access, and will label them as an "original match". It will subequently return all resources (that the session has the privileges to access) that match any of the conditions in the table below (and when returned, they will be labeled with the string in the "Event" column).

Event | Details
--- | ------- 
original match | A resource matches the criteria the first time the query is ran.
match created | A resource is created that matches the criteria 
match added | A resource that previously did not match the criteria now matches it
match deleted | A resource is deleted that previously matched the criteria
match modified | A resource that previously matched the criteria is modified, but still matches the criteria
match dropped | A resource that previously matched the criteria is modified and no longer matches the criteria

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'subscribe',
  'subscription' : '10irf0ej0fq30', // An example subscription id
  'topic' : 'users' // An example of a topic
  'criteria' : {...}
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "subscribe"
subscription | string | true | A unique ID representing the id that should be assigned to this subscription. This ID will be namespaced to your session internally, so must only be unique to your session, not globally. If you provide a string that you've previously used in this session, an error response will be returned.
topic | string | true | The name of the topic that the request concerns. 
criteria | object | true | An object containing zero or more key-value pairs that represent the criteria to filter resources on. 


### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok', 
		'event' : 'original match',
		'payload' : {...} 
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", one or more resources were retrieved successfully.
response.event | A string representing one of the events in the events-table.
response.payload | An object containing zero or more key-value pairs representing one of the resources retrieved.


<aside class="warning">
A single 'subscribe' request may yield many response messages, and those responses may not be unique.
</aside> 




## Unsubscribe

The "unsubscribe" request type allows a client to cancel a subscription that has been previously created. 

### Request Format

```javascript

// Build the data for the request
var data = {
  'type' : 'unsubscribe',
  'subscription' : '10irf0ej0fq30', // An example subscription id
};

// Send the request
ws.send(data);

```

Key | Type | Required? | Details
--- | ------- | ------ | ------
type | string  | true | "subscribe"
subscription | string | true | The ID of the subscription that should be cancelled. This should be identical to the 'subscription' key you provided when creating the subscription.

### Response Format 

```javascript

// Truncated Response Object
{
	'response' : {
		...
		'code' : 'ok'
	}
	...
}
```

Key | Details
--- | ------- 
response.code | If set to "ok", the subscription was deleted. 

<aside class="warning">
It may take several minutes (or longer) for an 'unsubscribe' request to halt subscription-related responses from being returned. 
</aside> 

