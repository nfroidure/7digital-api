![7digital](http://i.imgur.com/StUnvCy.png?1)
# Node.js API Client

Current head build status:

[![Build Status](https://travis-ci.org/raoulmillais/node-7digital-api.png?branch=master)](http://travis-ci.org/raoulmillais/node-7digital-api)

## About 7digital


7digital.com is an online music store operating in over 16 countries and
offering more than 11 million high quality DRM free MP3s (320kbps) from all
major labels and wide range of idependent labels and distributors. 7digital
API will give you access to the full catalogue including high quality album
art, 30s preview clips for all tracks, commissions on sales, integrated
purchasing and full length streaming. More details at
[developer.7digital.net](http://developer.7digital.net/)

### What is this?


A serverside javascript client for the 7digital API .
Full code documentation for the most recent release can be found [here](http://raoulmillais.github.com/node-7digital-api/api.html)

### Installation


[![NPM](https://nodei.co/npm/7digital-api.png?downloads=true)](https://nodei.co/npm/7digital-api/)

Install it via [npm](http://npmjs.org/)

```bash
npm install --save 7digital-api
```

### Usage


See the examples folder for examples of how to use this.  If you have included
7digital-api in your dependencies in the package.json file, you can use the
like so:

```javascript
var api = require('7digital-api'),
	artists = new api.Artists();

artists.getReleases({ artistid: 1 }, function(err, data) {
	console.dir(data);
});
```

To supply your OAuth credentials or if you want XML responses, you can use the
configure function.  Here is how you can do so:


```javascript
var api, artists;

api = require('7digital-api').configure({
	format: 'XML',
	consumerkey: 'MY_KEY_HERE',
	consumersecret: 'MY_SECRET_HERE',
	defaultParams: { country: 'fr' }
});

artists = new api.Artists();

artists.getReleases({ artistid: 1 }, function(err, data) {
	console.dir(data);
});
```

You can specify default parameters on a per resource basis also:


```javascript
var api, artists;

api = require('7digital-api').configure({
	defaultParams: {
		country: 'fr'
	}
});

artists = new api.Artists({ defaultParams: { pageSize: 15 } });

artists.getReleases({ artistid: 1 }, function(err, data) {
	// 15 releases in france
	console.dir(data);
});
```

See [developer.7digital.net](http://developer.7digital.net/) for full details
of the API endpoints and the parameters they accept.

### Making requests on behalf of a user to OAuth protected endpoints

**NOTE: The oauth access method changed considerably in 0.19.0**

This example assumes you have access to the oauth/requestToken/authorise
endpoint to authenticate users.  If you do not have this access you will
need to send the user to the authoriseUrl provided by `getRequestToken`
and complete the auth flow when your callbackUrl is hit.

```javascript
var api = require('7digital-api').configure({
	consumerkey: 'YOUR_KEY_HERE',
	consumersecret: 'YOUR_SECRET_HERE',
	defaultParams: {
		country: 'fr'
	}
});

var oauth = new api.OAuth();
oauth.getRequestToken('http://callbackurl.com/', authoriseToken);
function authoriseToken(err, requesttoken, requestsecret) {
	oauth.authoriseRequestToken({
		username: 'joe@bloggs.com',
		password: 'top-secret',
		token: requesttoken
	}, function (err) {
		oauth.getAccessToken({
			requesttoken: requesttoken,
			requestsecret: requestsecret
		}, function (err, accesstoken, accesssecret) {
			// use the token and secret to call secure endpoints.
			var apiForJoeBloggs = api.reconfigure({ 
				defaultParams: {
					accesstoken: accesstoken,
					accesssecret: accesssecret
				}
			});
			var user = new apiForJoeBloggs.User();
			user.getLocker({
				pageSize: 1
			}, function (err, response) {
				// Do something with the locker
			});
		});
	});
```

See oauth.js and create-user.js in the examples folder for examples of the
OAuth flow for acquiring an authorised access token and secret that you will
need to access any of the protected endpoints on behalf of a user.

```bash
node examples\oauth.js
```

### Running the tests

To run the unit tests:

    npm test

There are also integration tests. Tests for various error handling scenarios
are run against a stub 7d api. It can be installed
with:

    npm install git://github.com/7digital/api-stub.git

Some of the integration tests (around the client's handling of OAuth) run
against the real 7d api. In order for these tests to work, several environment
variables need to be set:

- `NODE_API_CLIENT_TESTS_CONSUMER_KEY`
- `NODE_API_CLIENT_TESTS_CONSUMER_SECRET`

Your 7d api key and secret, which can be obtained from
https://api-signup.7digital.com

- `NODE_API_CLIENT_TESTS_VOUCHER_CODE`

The code for a voucher which can be applied to a basket containing an item of
1p, used for a two-legged OAuth test.

- `NODE_API_CLIENT_TESTS_USER_TOKEN`
- `NODE_API_CLIENT_TESTS_USER_SECRET`

A token and secret for access to any user's resources for a given consumer key
and secret. These are used for 3-legged OAuth tests, and can be obtained by
running `node ./examples/oauth.js` and following the prompts.

If these vars are set, the tests can then be run with:

    mocha spec-integration/
