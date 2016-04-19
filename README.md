`medic-gateway`
===============

An SMS gateway for Android.  Send and receive SMS from your webapp via an Android phone.

	+--------+                 +-----------+
	|  web   |                 |  medic-   | <-------- SMS
	| server | <---- HTTP ---- |  gateway  |
	|        |                 | (android) | --------> SMS
	+--------+                 +-----------+

# API

## GET

Expected response:

	{
		"medic-gateway": true
	}

## POST

`medic-gateway` will accept and process any relevant data received in a response.  However, it may choose to only send certain types of information in a particular request (e.g. only provide a webapp-terminating SMS), and will also poll the web service periodically for webapp-originating messages, even if it has no new data to pass to the web service.

If there are no entries for a list field (e.g. `deliveries`, `messages`), `medic-gateway` and clients may:

* provide a `null` value; or
* provide an empty array (`[]`); or
* omit the field completely

## Request

### Headers

The following headers will be set by requests:

header           | value
-----------------+-------------------
`Accept`         | `application/json`
`Accept-Charset` | `utf-8`
`Accept-Encoding`| `gzip`
`Cache-Control`  | `no-cache`
`Content-Type`   | `application/json`

Requests and responses may be sent with `Content-Encoding` set to `gzip`.

### Content

	{
		messages: [
			{
				id: <String: uuid, generated by `medic-gateway`>,
				from: <String: international phone number>,
				content: <String: message content>
			},
			...
		],
		deliveries: [
			{
				id: <String: uuid, generated by webapp>,
				status: <String: rejected|failed|delivered>
			},
			...
		],
	}

## Response

### Success

#### Http Status: `2xx`

Clients may respond with any status code in the `200`-`299` range, as they feel is
appropriate.  `medic-gateway` will treat all of these statuses the same.

#### Content

	{
		messages: [
			{
				id: <String: uuid, generated by webapp>,
				to: <String: local or international phone number>,
				content: <String: message content>
			},
			...
		],
	}

### Error

Response codes of `400` and above will be treated as errors.

If the response's `Content-Type` header is set to `application/json`, `medic-gateway` will attempt to parse the body as JSON.  The following structure is expected:

{
	error: true,
	message: <String: error message>
}

The `message` property may be logged and/or displayed to users in the `medic-gateway` UI.

### Other response codes

Treatment of response codes below `200` and between `300` and `399` will _probably_ be handled sensibly by Android.


# `demo-server`

To start the demo server locally:

	make demo-server

To list the data stored on the server:

	curl http://localhost:8000

To make the next good request to `/app` return an error:

	curl -X POST http://localhost:8000/error --data '"Something failed!"'

To add a webapp-originating message (to be send by `medic-gateway`):

	curl -vvv -X POST -d '{ "to": "+447123555888", "message": "hi" }' http://localhost:8000

To simulate a request from `medic-gateway`:

	curl -X POST http://localhost:8000/app -H "Accept: application/json" -H "Accept-Charset: utf-8" -H "Accept-Encoding: gzip" -H "Cache-Control: no-cache" -H "Content-Type: application/json" -d'{}'

To clear the data stored on the server:

	curl -X DELETE http://localhost:8000
