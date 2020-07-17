# lib-hapi-good-tracer

[![Coverage Status](https://coveralls.io/repos/github/GoodwayGroup/lib-hapi-good-tracer/badge.svg?branch=master)](https://coveralls.io/github/GoodwayGroup/lib-hapi-good-tracer?branch=master) [![CircleCI](https://circleci.com/gh/GoodwayGroup/lib-hapi-good-tracer.svg?style=svg)](https://circleci.com/gh/GoodwayGroup/lib-hapi-good-tracer)

## Usage

This plugin will read, generate and track a tracer header on `request`s that is then injected into the `Good` log stream via an injected `tracer` object.

```json
{
  "tracer": {
    "uuid": "STRING",
    "depth": "INTEGER"
  }
}
```

The `depth` value provides insight into the hierarchy chain of requests. Combining the `uuid` + `depth` + `timestamp` of a request will provide a mapping to the request chain.

> NOTE: This module uses the [`debug`](https://www.npmjs.com/package/debug) logging tool. Use `DEBUG=hapi:plugins:good-tracer` to view debug logging.

```
$ npm install -S @goodwaygroup/lib-hapi-good-tracer
```

In your `index.js` for the Hapi server, register the plugin:

```js
await server.register({
    plugin: require('@goodwaygroup/lib-hapi-good-tracer'),
    options: {
        traceUUIDHeader: 'x-custom-trace-uuid', // optional defaults to 'x-gg-trace-uuid'
        traceDepthHeader: 'x-custom-trace-depth', // optional defaults to 'x-gg-trace-depth'
        enableStatsRoute: true, // optional defaults to false
        baseRoute: '/debug', // optional defaults to ''
        cache: {
            ttl: 60 // optional defaults to 120 seconds
        }
    }
});

// add stream to Good log reporters
const logReporters = {
    console: [
        server.plugins.goodTracer.GoodSourceTracer, // Stream Transform that will inject the tracer object
        {
            module: 'good-squeeze',
            name: 'Squeeze',
            args: [{
                response: { exclude: 'healthcheck' },
                log: '*',
                request: '*',
                error: '*',
                ops: '*'
            }]
        }, {
            module: 'good-squeeze',
            name: 'SafeJson'
        }, 'stdout']
};

await server.register({
    plugin: Good,
    options: {
        reporters: logReporters
    }
});
```

## Request Chain Hierarchy

Consider The following request chain:

[![](https://mermaid.ink/img/eyJjb2RlIjoiZ3JhcGggTFJcblx0QVtTZXJ2aWNlIEFdIC0tPiB8UmVxdWVzdCAxIGRlcHRoIDB8IEJbU2VydmljZSBCXVxuXHRCIC0tPiB8UmVxdWVzdCAxIGRlcHRoIDF8IENbU2VydmljZSBDXVxuICBCIC0tPiB8UmVxdWVzdCAyIGRlcHRoIDF8IENcbiAgQSAtLT4gfFJlcXVlc3QgMiBkZXB0aCAwfCBDIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiZ3JhcGggTFJcblx0QVtTZXJ2aWNlIEFdIC0tPiB8UmVxdWVzdCAxIGRlcHRoIDB8IEJbU2VydmljZSBCXVxuXHRCIC0tPiB8UmVxdWVzdCAxIGRlcHRoIDF8IENbU2VydmljZSBDXVxuICBCIC0tPiB8UmVxdWVzdCAyIGRlcHRoIDF8IENcbiAgQSAtLT4gfFJlcXVlc3QgMiBkZXB0aCAwfCBDIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)

The `depth` value provides insight into the hierarchy chain of requests. Combining the `uuid` + `depth` + `timestamp` of a request will provide a mapping to the request chain.

## In-Memory Cache

This plugin uses an in-memory cache that is used to pass the `tracer` information between the server, request and logger.

There is a global TTL per object that is reset on each `get` of the object.

If an object is stale for the length of the TTL, it will be culled.

There is a max number of keys that can be active at any time to help with memory concerns.

See [node-cache](https://github.com/node-cache/node-cache) for available settings.

## Configuration Options

> When passing a configuration option, it will overwrite the defaults.

- `traceUUIDHeader`: defaults to `x-gg-trace-uuid`. The header that is used for the Trace ID
- `traceDepthHeader`: defaults to `x-gg-trace-depth`. The header that is used for the Depth ID
- `enableStatsRoute`: defaults to `false`. Publish a route to `/good-tracer/stats` that exposes the current metrics for [`node-cache` statistics](https://github.com/node-cache/node-cache#statistics-stats).
- `baseRoute`: defaults to `''`. Prepends to the `/good-tracer/stats` route.
    - Example: `baseRoute = /serivce-awesome` results in `/serivce-awesome/good-tracer/stats`
- `cache`: internal memory cache settings. See [node-cache](https://github.com/node-cache/node-cache)
    - `ttl`: default 120 seconds
    - `checkPeriod`: default 5 minutes
    - `maxKeys`: default `5000`
    - `useClones`: default `false`
    - `extendTTLOnGet`: This feature will reset the TTL to the global TTL when a successful `get` occurs. This will extend the life of an item in the cache as a result. default `true`

## Running Tests

To run tests, just run the following:

```
npm test
```

All commits are tested on [CircleCI](https://circleci.com/gh/GoodwayGroup/workflows/lib-hapi-good-tracer)

## Linting

To run `eslint`:

```
npm run lint
```

To auto-resolve:

```
npm run lint:fix
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use milestones and `npm` version to bump versions. We also employ [git-chglog](https://github.com/git-chglog/git-chglog) to manage the [CHANGELOG.md](CHANGELOG.md). For the versions available, see the [tags on this repository](https://github.com/GoodwayGroup/lib-hapi-good-tracer/tags).

To initiate a version change:

```
npm version minor
```

## Authors

* **Derek Smith** - *Initial work* - [@clok](https://github.com/clok)

See also the list of [contributors](https://github.com/GoodwayGroup/lib-hapi-good-tracer/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

## Sponsors

[![goodwaygroup][goodwaygroup]](https://goodwaygroup.com)

[goodwaygroup]: https://s3.amazonaws.com/gw-crs-assets/goodwaygroup/logos/ggLogo_sm.png "Goodway Group"
