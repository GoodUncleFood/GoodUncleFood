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

## Downgrade

## Create

## Read

## Update

## Delete

## Subscribe

## Unsubscribe