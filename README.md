nsqjs
=====
The official NodeJS client for the [nsq](http://nsq.io/) client protocol. This implementation attempts to be
fully compliant and maintain feature parity with the official Go ([go-nsq](https://github.com/bitly/go-nsq)) and Python ([pynsq](https://github.com/bitly/pynsq)) clients.

[![Build Status](https://travis-ci.org/dudleycarr/nsqjs.png?branch=master)](https://travis-ci.org/dudleycarr/nsqjs)

[![NPM](https://nodei.co/npm/nsqjs.png?downloads=true)](https://nodei.co/npm/nsqjs/)

Usage
-----

### new Reader(topic, channel, options)
The topic and channel arguments are strings and must be specified. The options
argument is optional. Below are the parameters that can be specified in the
options object.

* ```maxInFlight: 1``` <br/>
  The maximum number of messages to process at once. This value is shared between nsqd connections. It's highly recommended that this value is greater than the number of nsqd connections.
* ```heartbeatInterval: 30``` <br/>
  The frequency in seconds at which the nsqd will send heartbeats to this Reader.
* ```maxBackoffDuration: 128``` <br/>
  The maximum amount of time (seconds) the Reader will backoff for any single backoff
  event.
* ```maxAttempts: 0``` <br/>
  The number of times to a message can be requeued before it will be handed to the DISCARD handler and then automatically finished. 0 means that there is **no limit.** If not DISCARD handler is specified and `maxAttempts > 0`, then the message will be finished automatically when the number attempts has been exhausted.
* ```requeueDelay: 90``` <br/>
  The default amount of time (seconds) a message requeued should be delayed by before being dispatched by nsqd.
* ```nsqdTCPAddresses``` <br/>
  A string or an array of string representing the host/port pair for nsqd instances.
  <br/> For example: `['localhost:4150']`
* ```lookupdHTTPAddresses``` <br/>
  A string or an array of strings representing the host/port pair of nsqlookupd instaces or the full HTTP/HTTPS URIs of the nsqlookupd instances.
  <br/> For example: `['localhost:4161']`, `['http://localhost/lookup']`, `['http://localhost/path/lookup?extra_param=true']`
* ```lookupdPollInterval: 60``` <br/>
  The frequency in seconds for querying lookupd instances.
* ```lookupdPollJitter: 0.3``` <br/>
  The jitter applied to the start of querying lookupd instances periodically.
* ```tls: false``` <br/>
  Use TLS if nsqd has TLS support enabled.
* ```tlsVerification: true``` <br/>
  Require verification of the TLS cert. This needs to be false if you're using
  a self signed cert.
* ```deflate: false``` <br/>
  Use zlib Deflate compression.
* ```deflateLevel: 6``` <br/>
  Use zlib Deflate compression level.
* ```snappy: false``` <br/>
  Use Snappy compression.
* ```authSecret: null```<br/>
  Authenticate using the provided auth secret.
* ```outputBufferSize: null```<br/>
  The size in bytes of the buffer nsqd will use when writing to this client. -1
  disables buffering. ```outputBufferSize >= 64```
* ```outputBufferTimeout: null```<br/>
  The timeout after which any data that nsqd has buffered will be flushed to this client. Value is in milliseconds. ```1 <= outputBufferTimeout```. A value of ```-1``` disables timeouts.
* ```messageTimeout: null```<br/>
  Sets the server-side message timeout in milliseconds for messages delivered to this client.
* ```sampleRate: null```<br/>
  Deliver a percentage of all messages received to this connection. ```1 <=
  sampleRate <= 99```
* ```clientId: null```<br/>
  An identifier used to disambiguate this client.
  

Reader events are:

* `Reader.MESSAGE` or `message`
* `Reader.DISCARD` or `discard`
* `Reader.ERROR` or `error`
* `Reader.NSQD_CONNECTED` or `nsqd_connected`
* `Reader.NSQD_CLOSED` or `nsqd_closed`

`Reader.MESSAGE` and `Reader.DISCARD` both produce `Message` objects.
`Reader.NSQD_CONNECTED` and `Reader.NSQD_CLOSED` events both provide the host
and port of the nsqd to which the event pertains.

These methods are available on a Reader object:
* `connect()` <br/>
  Connect to the nsqds specified or connect to nsqds discovered via
  lookupd.
* `close()` <br/>
  Disconnect from all nsqds. Does not wait for in-flight messages to complete.
* `pause()` <br/>
  Pause the Reader by stopping message flow. Does not affect in-flight
  messages.
* `unpause()` <br/>
  Unpauses the Reader by resuming normal message flow.
* `isPaused()` <br/>
  `true` if paused, `false` otherwise.


### Message
The following properties and methods are available on Message objects produced by a Reader
instance.

* `timestamp` <br/>
  Numeric timestamp for the Message provided by nsqd.
* `attempts` <br/>
  The number of attempts that have been made to process this message.
* `id` <br/>
  The opaque string id for the Message provided by nsqd.
* `hasResponded` <br/>
  Boolean for whether or not a response has been sent.
* `body` <br/>
  The message payload as a Buffer object.
* `json()` <br/>
  Parses message payload as JSON and caches the result.
* `timeUntilTimeout(hard=false)`: <br/>
  Returns the amount of time until the message times out. If the hard argument
  is provided, then it calculates the time until the hard timeout when nsqd
  will requeue inspite of touch events.
* `finish()` <br/>
  Finish the message as successful.
* `requeue(delay=null, backoff=true)`
  The delay is in seconds. This is how long nsqd will hold on the message
  before attempting it again. The backoff parameter indicates that we should
  treat this as an error within this process and we need to backoff to recover.
* `touch()` <br/>
  Tell nsqd that you want extra time to process the message. It extends the
  soft timeout by the normal timeout amount.

### new Writer(nsqdHost, nsqdPort, options)
Allows messages to be sent to an nsqd. 

Available Writer options:
* ```tls: false``` <br/>
  Use TLS if nsqd has TLS support enabled.
* ```tlsVerification: true``` <br/>
  Require verification of the TLS cert. This needs to be false if you're using
  a self signed cert.
* ```deflate: false``` <br/>
  Use zlib Deflate compression.
* ```deflateLevel: 6``` <br/>
  Use zlib Deflate compression level.
* ```snappy: false``` <br/>
  Use Snappy compression.
* ```clientId: null```<br/>
  An identifier used to disambiguate this client.

Writer events are:

* `Writer.READY` or `ready`
* `Writer.CLOSED` or `closed`
* `Writer.ERROR` or `error`

These methods are available on a Writer object:
* `connect()` <br/>
  Connect to the nsqd specified.
* `close()` <br/>
  Disconnect from the nsqd.
* `publish(topic, msgs, [callback])` <br/>
  `topic` is a string. `msgs` is either a string, a `Buffer`, JSON serializable
  object, a list of strings / `Buffers` / JSON serializable objects. `callback` takes a single `error` argument.

### Simple example

Start [nsqd](http://nsq.io/components/nsqd.html) and
[nsqdlookupd](http://nsq.io/components/nsqlookupd.html)
```bash
# nsqdLookupd Listens on 4161 for HTTP requests and 4160 for TCP requests
$ nsqlookupd &
$ nsqd --lookupd-tcp-address=127.0.0.1:4160 &
```

#### JavaScript
```js
var nsq = require('nsqjs');

var reader = new nsq.Reader('sample_topic', 'test_channel', {
  lookupdHTTPAddresses: '127.0.0.1:4161'
});

reader.connect();

reader.on('message', function (msg) {
  console.log('Received message [%s]: %s', msg.id, msg.body.toString());
  msg.finish();
});
```

#### CoffeeScript
```coffee-script
nsq = require 'nsqjs'

topic = 'sample_topic'
channel = 'test_channel'
options =
  lookupdHTTPAddresses: '127.0.0.1:4161'

reader = new nsq.Reader topic, channel, options
reader.connect()

reader.on nsq.Reader.MESSAGE, (msg) ->
  console.log "Received message [#{msg.id}]: #{msg.body.toString()}"
  msg.finish()
```

Publish a message to nsqd to be consumed by the sample client:
```bash
$ curl -d "it really tied the room together" http://localhost:4151/pub?topic=sample_topic
```

### Example with message timeouts

This script simulates a message that takes a long time to process or at least
longer than the default message timeout. To ensure that the message doesn't
timeout while being processed, touch events are sent to keep it alive.

#### JavaScript
```js
var nsq = require('nsqjs');

var reader = new nsq.Reader('sample_topic', 'test_channel', {
  lookupdHTTPAddresses: '127.0.0.1:4161'
});

reader.connect();

reader.on('message', function (msg) {
  console.log('Received message [%s]', msg.id);

  function touch() {
    if (!msg.hasResponded) {
      console.log('Touch [%s]', msg.id);
      msg.touch();
      // Touch the message again a second before the next timeout.
      setTimeout(touch, msg.timeUntilTimeout() - 1000);
    }
  }

  function finish() {
    console.log('Finished message [%s]: %s', msg.id, msg.body.toString());
    msg.finish();
  }

  console.log('Message timeout is %f secs.', msg.timeUntilTimeout() / 1000);
  setTimeout(touch, msg.timeUntilTimeout() - 1000);

  // Finish the message after 2 timeout periods and 1 second.
  setTimeout(finish, msg.timeUntilTimeout() * 2 + 1000);
});
```

#### CoffeeScript
```coffee-script
{Reader} = require 'nsqjs'

topic = 'sample_topic'
channel = 'test_channel'
options =
  lookupdHTTPAddresses: '127.0.0.1:4161'

reader = new Reader topic, channel, options
reader.connect()

reader.on Reader.MESSAGE, (msg) ->
  console.log "Received message [#{msg.id}]"

  touch = ->
    unless msg.hasResponded
      console.log "Touch [#{msg.id}]"
      msg.touch()
      # Touch the message again a second before the next timeout.
      setTimeout touch, msg.timeUntilTimeout() - 1000

  finish = ->
    console.log "Finished message [#{msg.id}]: #{msg.body.toString()}"
    msg.finish()

  console.log "Message timeout is #{msg.timeUntilTimeout() / 1000} secs."
  setTimeout touch, msg.timeUntilTimeout() - 1000

  # Finish the message after 2 timeout periods and 1 second.
  setTimeout finish, msg.timeUntilTimeout() * 2 + 1000
```

### Enable nsqjs debugging
nsqjs used [debug](https://github.com/visionmedia/debug) to log debug output.

To see all nsqjs events:
```
$ DEBUG=nsqjs:* node my_nsqjs_script.js
```

To see all reader events:
```
$ DEBUG=nsqjs:reader:* node my_nsqjs_script.js
```

To see a specific reader's events:
```
$ DEBUG=nsqjs:reader:<topic>/<channel>:* node my_nsqjs_script.js
```
> Replace `<topic>` and `<channel>`

To see all writer events:
```
$ DEBUG=nsqjs:writer:* node my_nsqjs_script.js
```


### A Writer Example

The writer sends a single message and then a list of messages.

#### JavaScript
```js
var nsq = require('nsqjs');

var w = new nsq.Writer('127.0.0.1', 4150);

w.connect();

w.on('ready', function () {
  w.publish('sample_topic', 'it really tied the room together');
  w.publish('sample_topic', [
    'Uh, excuse me. Mark it zero. Next frame.', 
    'Smokey, this is not \'Nam. This is bowling. There are rules.'
  ]);
  w.publish('sample_topic', 'Wu?', function (err) {
    if (err) { return console.error(err.message); }
    console.log('Message sent successfully');
    w.close();
  });
});

w.on('closed', function () {
  console.log('Writer closed');
});
```

#### CoffeeScript
```coffee-script
{Writer} = require 'nsqjs'

w = new Writer '127.0.0.1', 4150
w.connect()

w.on Writer.READY, ->
  w.publish 'sample_topic', 'it really tied the room together'
  w.publish 'sample_topic', ['Uh, excuse me. Mark it zero. Next frame.', 
    'Smokey, this is not \'Nam. This is bowling. There are rules.']
  w.publish 'sample_topic', 'Wu?', (err) ->
    console.log 'Message sent successfully' unless err?
    w.close()

w.on Writer.CLOSED, ->
  console.log 'Writer closed'
```

Changes
-------
* **0.7.7**
  * Build with Node v4
* **0.7.6**
  * Fix npm install by adding .npmignore.
* **0.7.3**
  * Slightly better invalid topic and channel error messages.
  * Handle more conditions for failing to publish a message.
* **0.7.2**
	* Fix build for iojs and node v0.12
	* Bumped snappystream version.
* **0.7.1**
  * Fix connection returning to max connection RDY after backoff
  * Fix backoff ignored when `message.finish` is called after backoff event.
* **0.7.0**
  * Fixes for configuration breakages
  * Fix for AUTH
  * Fix for pause / unpause
  * Automatically finish messages when maxAttempts have been exceeded.
  * `maxAttempts` is now by default 0. [ Breaking Change! ]
  * discarded messages will now be sent to the `MESSAGE` listener if there's no
    `DISCARD` listener.
  * Support for emphemeral topics.
  * Support for 64 char topic / channel names.
  * Support for Lookupd URLs
  * Deprecate StateChangeLogger infavor of `debug` [ Breaking Change! ]
* **0.6.0**
  * Added support for authentication
  * Added support for sample rate
  * Added support for specifying outputBufferSize and outputBufferTimeout
  * Added support for specifying msg_timeout
  * Refactored configuration checks
  * Breaking change for NSQDConnection constructor. NSQDConnection takes an
    options argument instead of each option as a parameter.
* **0.5.1**
  * Fix for not failing when the nsqd isn't available on start.
* **0.5.0**
  * Reworked FrameBuffer
  * Added TLS support for Reader and Writer
  * Added Deflate support
  * Added Snappy support
* **0.4.1**
	* Fixed a logging issue on NSQConnection disconnected
* **0.4.0**
	* Added `close`, `pause`, and `unpause` to Reader
	* Added callback for Writer publish
  * Expose error events on Reader/Writer
  * Expose nsqd connect / disconnect events
  * Fix crash when Message `finish`, `requeue`, etc after nsqd disconnect
  * Fix lookupd only queried on startup.
* **0.3.1**
  * Fixed sending an array of Buffers
  * Fixed sending a message with multi-byte characters
* **0.3.0**
  * Added Writer implementation
* **0.2.1**
  * Added prepublish compilation to JavaScript.
* **0.2.0**
  * `ReaderRdy`, `ConnectionRdy` implementation
  * `Reader` implementation
  * Initial documentation
  * `NSQDConnection`
    * Moved defaults to `Reader`
    * Support protocol / state logging
    * `connect()` now happens on next tick so that it can be called before event
      handlers are registered.
  * `Message`
    * Correctly support `TOUCH` events
    * Support soft and hard timeout timings
    * JSON parsing of message body
* **0.1.0**
  * `NSQDConnection` implementation
  * `wire` implementation
  * `Message` implementation
