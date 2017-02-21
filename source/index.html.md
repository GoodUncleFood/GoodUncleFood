---
title: API Reference

language_tabs:
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

```javascript

// Test your browser to see if it supports websockets

if (window.WebSocket){
     console.log("This browser supports websockets.");
} else {
     console.log("This browser does not support websockets.");
}
```

Welcome to the Good Uncle Websocket API. The production and staging versions of this API can be at the following URLs:

* ws://socket.gooduncle.com
* ws://staging.socket.gooduncle.com

Accessing this API requires an [RFC6455](https://tools.ietf.org/html/rfc6455)-compatible websocket client/library. The Javascript examples in these docs are presented using the [HTML5 Websockets](https://www.w3.org/TR/2011/WD-websockets-20110419/) standard (and so should work in [most modern browsers](http://caniuse.com/#feat=websockets)), though any library (in any language) will do. 

The code for these examples can be found [here](https://gist.github.com/malcolmdiggs/6faa0c7174382a95d2085e4873eebaaf). Just download the HTML file, and view it in a browser, then open your inspector to see the console logs. 

## Opening and closing a Connection

```javascript


// Define the host
var host = 'ws://staging.socket.gooduncle.com';

// Establish a connection
var ws = new WebSocket(host);

// Bind to the onopen event if needed
ws.onopen = function(e){
	console.log('The socket is open');
};

// Bind to any errors experienes
ws.onerror = function(e){
	console.log('The socket is experiencing an error');
};

// Bind to the onclose event if needed 
// Note: Dont depend on this method, its unreliable. 
ws.onclose = function(e){
	console.log('The socket is closed');
};
 

```

To open a connection, create a client using your websocket-library of choice. 

Websockets can be closed in one of three ways: 

1. The client can close the connection
2. The server can close the connection
3. The connection between the client and server can be lost

Detecting/binding to case 1 is straightforward, but cases 2 and 3 are not. The websocket onclose event only fires if an 'end' frame is received from the server. If a connection is dropped/faulty however, no such frame may be received, so this event will never fire.

To compensate for this error-prone mechanism, we have developed our own methods for detecting cases 2 and 3, and those are detailed in the "Detecting a Faulty Connection" section below. Clients can continue to use the onclose event, but should not depend on it.



## Keeping a Connection Open

```javascript

// Send a single unicode character every 300ms
var _keepAlive = function(){
	ws.send('`');
};
var foo = setInterval(_keepAlive, 300);

```

The websocket server will automatically close the connection if nothing is received from your client for 3000 milliseconds.

To avoid such a closure, send a single unicode character message to the server once every ~300 milliseconds (though intervals greater or less than that are acceptable as long as it does not exceed 3000.

<aside class="warning">
This API makes no attempt to conform to the ping-pong standards expressed in RFC6455. Instead, we make use of our own "keep-alive"-like protocol explained above. In the future we may establish a more formalized sub-protocol to codify this, and other departures from RFC6455.
</aside>



## Detecting a Faulty Connection


```javascript

// Sample logic for detecting faulty connections
var lastReceivedMessage = Date.now();

// Bind to the onmessage event
ws.onmessage = function(e) {
  console.log('Received message',e.data);
  lastReceivedMessage = Date.now();
 };
 
// Every second, check if the connection is faulty.
var _loop = function(){
	var elapsed = Date.now() - lastReceivedMessage;
	if(elapsed > 3000){
		console.log("The connection is probably dead");
	} else {
		if(elapsed > 1000){
			console.log("The connection is faulty");
		} else {
			console.log("Everythings fine");
		}
	}
};
var bar = setInterval(_loop, 1000);

```

All connected clients are sent a small message, containing nothing other than a single unicode character, every ~300 milliseconds. This is to let the client know that the server is still connected.

If no message is received from the server for > ~1000 ms, the client should interpret this as a faulty/intermittent connection.

If no message is received from the server for > ~3000 ms, the client should interpret this as a closed connection, and should attempt to re-establish the connection. Once a connection is lost and re-established, no responses to previous queries will be sent, and all data-subscriptions will need to be re-requested.


## Sending and Receiving Messages

```javascript

// Typical Request
{
	'type' : 'read', // Required key
	'foo' : 'bar' // Other required and optional keys, depending on request type
}

// Typical Response
{
	'session' : {
		'id' : 'mySessionid', // String
		'roles' : [] // Array of strings
	},
	'request' : {
		'time' : 123456, // Number
		'payload' : {} // original data sent with the request
	},
	'response' : {
		'time' : 123456, // Number
		'code' : 'ok', // String
		'errors' : [], // Array of objects
		'payload' : {} // Object containing zero or more keys
	},
	'passport' : [] // Array of objects	
}
```

The API communicates entirely in JSON format. Any messages sent from a client that aren't valid JSON are ignored (though they do count towards keeping the connection open). Any messages received from the API that aren't valid JSON should be assumed to be nothing more than unicode characters sent by the API to let the client know the connection is still open.

All requests, to be considered valid, should be JSON objects, containing at least a "type" key, and the value of that key should be a string one of the request-types listed in the "Request Types" section below. In many cases, (such as create, read, update, or delete types), other keys such as "topic" are required as well. More details about the keys required for each request-type can be found in following sections. 

All valid responses sent by the API will contain the following keys, at least:

* session // An object containing information about the session
* request // An object containing information about the original request that triggered the response
* response // An object containing information about the response
* passport // An array of objects. Each object within this array represents a service or server that was involved in processing the request and the subsequent response. This array is used for debugging and analytics, and can be ignored by the client.


## Requests vs Responses

For the purpose of this documentation, "Requests" are messages sent from the client to the server, and "Responses" are messages sent from the server to client.

The server only ever sends responses in response to a request. Therefore, every response object will always contain a 'request' key embedded within it, as a reference to the original request that the response is responding to.

However, responses and requests do not necessarily have a one-to-one relationship. A single request may yield no response, one response, or many responses. Responses are also not time-bound or sequential. So, for example: requests A, B, and C, sent in that order, might yield 5 immediate responses to B, one response to C, 7 more responses to B, and no response to A at all. Then 60 minutes later, another response to C.

Responses are also not necessarily unique. This is particularly the case for responses to 'subscribe' requests. Responses to this type of request might yield several responses all related to the same objects / items.


## Request Types


* upgrade:  Add a role to a session so that authenticated requests can be performed
* downgrade : Remove a role from a session
* create : Equivalent to POST in HTTP
* read : Equivalent to GET in HTTP
* update : Equivalent to PUT in HTTP
* delete : Equivalent to DELETE in HTTP
* subscribe : Get results for a query, and get all subsequent results for that query if anything changes about the result set.
* unsubscribe : No longer receive subscriptions for a query.

See the #request-types section of these docs for more details about each request type.

## Response Codes

Within the response body, a 'response.code' key is always sent. That key will contain a single string, and that string will be one of the following:

* ok : Your request was performed successfully.
* forbidden : Your requests attempted to perform an action (or retrieve data) it was not authorized to perform/view.
* not found : The item you requested could not be found.
* bad request : There was an error with the structure of the request. You are missing fields or they contain invalid values. See the the 'response.errors' array for more details.
* error : The system encountered an error while trying to process your request.


# Upgrade

The "upgrade" request type allows a client to add a role to a session, so that authenticated requests can be performed from that point forward.

## Request Format

## Response Format 

# Downgrade

# Create

# Read

# Update

# Delete

# Subscribe

# Unsubscribe

