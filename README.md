# Batchelor
A lovely little Node.js module to perform batch requests with the Google REST API. Simply does it.

## Google API Batch Requests
This is a project to solve a need for a missing feature in the wonderfully epic and useful [google/google-api-nodejs-client](https://github.com/google/google-api-nodejs-client). Currently, this cannot be used to post media and has not been tested with posting anything but JSON.

In theory this library could be used with other APIs, but has only been tested with Google's APIs as that's what we need it for.

Feel free to get involved in development.

## Upgrading to 2.0

`batch.run(...)` now takes a node style callback, with an error as the first parameter and the result as the second.

**INCORRECT** (old, v1.0-style):

``` node
batch.run(function(result){});
```

**CORRECT** (new, v2.0-style):

``` node
batch.run(function(err, result){});
```

## Upgrading to 1.0

The API was changed in 1.0 to move from a singleton instance to a constructor. So before where you used `Batchelor` directly:

``` node
var Batchelor = require('batchelor')
Batchelor.init(...)
Batchelor.add(...)
Batchelor.run(...)
```

You now need to create an instance of Batchelor:

``` node
var Batchelor = require('batchelor')
var batch = new Batchelor(...)
batch.add(...)
batch.run(...)
```

See <https://github.com/wapisasa/batchelor/issues/4> for why this change was made.

## Installation

This library has also been distributed on `npm`. Install it with the following command:

``` sh
$ npm install batchelor --save
```

## How to Use
#### GET Requests
``` node
var Batchelor = require('batchelor');
```
Once the module has been included, we initialise it with all our default options:
``` node
var batch = new Batchelor({
	// Any batch uri endpoint in the form: https://www.googleapis.com/batch/<api>/<version>
	'uri':'https://www.googleapis.com/batch/gmail/v1/',
	'method':'POST',
	'auth': {
		'bearer': [... Google API Token ...]
	},
	'headers': {
		'Content-Type': 'multipart/mixed'
	}
});
```
We can then start adding requests to our batch. This can be done 2 ways:

As a one-off object:
``` node
batch.add({
	'method':'GET',
	'path':'/gmail/v1/users/me/messages/123'
})
```
Or an Array of objects:
``` node
batch.add([
	{
		'method':'GET',
		'path':'/gmail/v1/users/me/messages/123'
	},
	{
		'method':'GET',
		'path':'/gmail/v1/users/me/messages/456'
	},
	{
		'method':'GET',
		'path':/gmail/v1/users/me/messages/789'
	}
]);
```
Once you have added all of the requests you need, call `.run()`:
``` node
batch.run(function(err, response){
	if (err){
		console.log("Error: " + err.toString());
	} else {
		res.json(response);
	}
});
```
#### POST Requests
The above examples show `GET` requests. To perform a `POST` requires a few more settings:
``` node
batch.add({
	'method':'POST',
	'path':'/gmail/v1/users/me/messages/',
	'parameters':{
		'Content-Type':'application/json;',
		'body':{'object':{'originalContent': 'A wonderful batch post!'},'access': {'items': [{'type': 'domain'}],'domainRestricted': true}}
	}
});
```
#### Callbacks
By default, all responses are returned through the callback function in the `batch.run()` call. Alternatively, a callback can be supplied for each individual calls:
``` node
batch.add({
	'method':'POST',
	'path':'/gmail/v1/users/me/messages/',
	'parameters':{
		'Content-Type':'application/json;',
		'body':{'object':{'originalContent': 'Another wonderful batch post with callback!'},'access': {'items': [{'type': 'domain'}],'domainRestricted': true}}
	},
	'callback':function(response){
		console.log(response);
	}
});
```
#### Request and Response IDs
The module will assign a request a randomly generated unique `Content-ID` by default, but this can be supplied as part of the options to supply `batch.add()`:
``` node
batch.add({
	'method':'GET',
	'path':'/gmail/v1/users/me/messages/123',
	'requestId':'Batch_UniqueID_1'
})
```
#### A Couple of Little Gifts
###### Method Chaining
All methods return the `Batchelor` instance. So you can chain calls together.

``` node
batch.add([
	...
]).run(function(err, data){
	...
});
```
###### Data Pass-through
When passing options to the `.add()` you can include an Object called `extend`. In the case of providing a callback, this will be passed back as a second parameter. When using the default callback on the `.run()` call, an array of all data passed through will be added as a second parameter with the requestId as the key:
``` node
batch.add({
	...
	'extend':{
		...
	},
	'callback':function(response, extendObj){
		console.log(response, extendObj);
	}
});
```
This could be required, for example, when making multiple requests with different Auth data and then needing to make further requests with the same Auth data.

###### Resetting and Re-using
Once Batchelor has been run, there are certain use-cases for running futher batch requests on response. This requires the variables in the instance to be reset. This can be done using the `.reset()` call:
``` node
batch.run(function(err, response){

	//	Reset Batchelor for further use
	batch.reset();
	...

});
```

## To Do List-ish
These might get done if we end up needing them/have time:
* Limit requests per batch request
* Handle Media in API calls (no need for it here, feel free to write it)

## Building

`npm run build`

Outputs a commonjs-compatible bundle to `dist/index.js`.

## Running tests

`npm test`

## Publishing

Merging to master will automatically publish the package if commits with non-trivial changes have
been introduced (per [commit conventions](https://www.conventionalcommits.org)).

## Acknowledgement
Built on the clock by [@jamesmhaley](https://github.com/jamesmhaley) as an internal project of [wapisasa C.I.C.](http://www.wapisasa.com)
