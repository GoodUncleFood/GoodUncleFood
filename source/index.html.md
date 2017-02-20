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

Welcome to the Good Uncle Websocket API. The production and staging versions of this API can be at the following URLs:

* ws://socket.gooduncle.com
* ws://staging.socket.gooduncle.com

Accessing this API requires an [RFC6455](https://tools.ietf.org/html/rfc6455)-compatible websocket client/library. The Javascript examples in these docs are presented using the [sockjs-client](https://github.com/sockjs/sockjs-client) library, though any library (in any language) will do. 

## Opening and closing a Connection

```javascript

// Open the connection
var sock = new SockJS('ws://staging.socket.gooduncle.com');

// Bind to the onopen event if needed
sock.onopen = function() {
  console.log('open');
};

// Close the connection
sock.close();

// Bind to the onclose event if needed
sock.onclose = function() {
  console.log('close');
};
 

```

To open a connection, create a client using your websocket-library of choice. 

Websockets can be closed in one of three ways: 

1. The client can close the connection
2. The server can close the connection
3. The connection between the client and server can be lost

Detecting/binding to case 1 is straightforward, but cases 2 and 3 are not. To compensate for the error-prone mechanisms provided by the Websocket specification, we have developed our own methods for detecting cases 2 and 3, and those are detailed in the "Detecting a Faulty Connection" section below.



## Keeping a Connection Open

```javascript

// Send a single unicode character every 300ms
var _keepAlive = function(){
	sock.send('`');
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

sock.onmessage = function(e) {
  lastReceivedMessage = Date.now();
 };
 
 var _loop = function(){
 	var elapsed = Date.now() - lastReceivedMessage;
 	if(elapsed > 3000){
 	  console.log("The connection is probably dead");
 	} else {
 	  if(elapsed > 1000){
 	  	console.log("The connection is faulty");
 	  }
 	}
 };

var foo = setInterval(_loop, 1000);

```

All connected clients are sent a small message, containing nothing other than a single unicode character, every ~300 milliseconds. This is to let the client know that the server is still connected.

If no message is received from the server for > ~1000 ms, the client should interpret this as a faulty/intermittent connection.

If no message is received from the server for > ~3000 ms, the client should interpret this as a closed connection, and should attempt to re-establish the connection.


## Sending and Receiving Messages

```javascript

// Typical Request
{
	'type' : 'create'
}

// Typical Response
{
	'session_id' : 'mySessionid', 
	'session_roles' : [], // Array of strings
	'request_time' : 1234567, // Unix timestamp in milliseconds
	'response_time' : 1234567, 
	'passport_stamps' : [], // Array of objects
	'request_payload' : data, // The JSON object received in the request that triggered this response
	'response_code' : 'ok', 
	'response_errors' : [], // Array of objects
	'response_payload' : {} // Object containing zero or more keys
}
```

The API communicates entirely in JSON format. Any messages sent that aren't valid JSON are ignored (though they do count towards keeping the connection open). Any messages received from the API that aren't valid JSON should be assumed to be nothing more than unicode characters sent by the API to let the client know the connection is still open.

All requests, to be considered valid, should be JSON object structures, containing a "type" key, and that key should be a string one of the request-types listed in the "Request Types" section below. In many cases, (such as create, read, update, or delete types), other keys such as "topic" are required. More details about the key required for each request-type can be found in following sections. 

All valid responses sent by the API should contain the following keys, at least:


## Requests vs Responses



## Request Types

* ping * upgrade
* downgrade * create
* read
* update * delete * subscribe
* unsubscribe


...See the #request-types section of these docs for details.

## Valid Formats

## Invalid Formats

## Response Types

# Request Types

# Upgrade

> To authorize, use this code:

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="warning">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

