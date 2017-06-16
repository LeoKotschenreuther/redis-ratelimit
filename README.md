
  Rate limiting / throttling using Redis

  [![NPM Version][npm-image]][npm-url]
  [![NPM Downloads][downloads-image]][downloads-url]
  [![Build Status][travis-image]][travis-url]

```js
var slidingWindow = require('redis-ratelimit').slidingWindow
var async = require('async')

var count = 0;
async.doWhilst(function(done) {
    slidingWindow.check('counter', 10, 2, function(err, limited) {
        if(limited)
        {
            // Don't do anything, wait some amount of time
            // and check rate limit again
            setTimeout(done, 1000*5);
        }
        else
        {
            // Do work
            count++;
            done();
        }
    })
}, function() {
    return count < 10;
}, function(err) {
    // Done
    process.exit(0);
});

```

## Installation

```bash
$ npm install redis-ratelimit
```

## Features

  * Provides a distributed rate limit per key
  * Sliding window uses redis sorted sets for high performance
  * Fixed window using simple counter with expiration

## Documentation
### configure(port, host, options)

Optionally configures the underlying redis instance used by the rate limiter.

__Arguments__

* `port` - Port redis is listening on
* `host` - Host redis is on
* `options` - Options hash passed through to underlying redis createClient() call

### slidingWindow

The sliding window implementation

#### check(key, windowInSeconds, limit, callback)

Checks if the specified key over or under the rate limit.  If over rate limit the call to check is not counted
against the rate limit

__Arguments__

* `key` - Unique key the rate limit is associated with
* `windowInSeconds` - The length of the window during which the calls are rate limited.  This is not a bucket
but a sliding window
* `limit` - Number of calls that are allowed before the rate limit kicks in
* `callback(err, limited)` - A callback which indicates if the rate limit has kicked in or not.  If an error
occurred limited will be set to true.  If limited returns true the call to check() does not count against
the rate limit.

#### count(key, callback)

Returns the number of calls that are currently counting against the rate limit.

__Arguments__

* `key` - Unique key the rate limit is associated with
* `callback(err, count)` - A callback which returns the count of calls counting against the rate limit.

### fixedWindow

The fixed window implementation

#### check(key, windowInSeconds, limit, callback)

Checks if the specified key over or under the rate limit.  If over rate limit the call to check is not counted
against the rate limit

__Arguments__

* `key` - Unique key the rate limit is associated with
* `windowInSeconds` - The length of the window during which the calls are rate limited.  This is a bucket
not a sliding window.  The underlying redis key expires after this duration.
* `limit` - Number of calls that are allowed before the rate limit kicks in
* `callback(err, limited)` - A callback which indicates if the rate limit has kicked in or not.  If an error
occurred limited will be set to true.  If limited returns true the call to check() does not count against
the rate limit.

#### count(key, callback)

Returns the number of calls that are currently counting against the rate limit.  Note this number can be greater than the limit as each call to check increments the counter.

__Arguments__

* `key` - Unique key the rate limit is associated with
* `callback(err, count)` - A callback which returns the count of calls counting against the rate limit.


## People

The author is [Chris Kinsman](https://github.com/chriskinsman)

## License

  [MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/redis-ratelimit.svg?style=flat
[npm-url]: https://npmjs.org/package/redis-ratelimit
[downloads-image]: https://img.shields.io/npm/dm/redis-ratelimit.svg?style=flat
[downloads-url]: https://npmjs.org/package/redis-ratelimit
[travis-image]: https://img.shields.io/travis/chriskinsman/redis-ratelimit.svg?style=flat
[travis-url]: https://travis-ci.org/chriskinsman/redis-ratelimit
