# Keen.io Caching Proxy

Save time and bandwidth by caching your read requests.

# Quickstart with Docker

*Estimated Setup Time: 5 minutes*

* Run mongodb
  * [mongohq.com](mongohq.com), [mongolab.com](mongolab.com) if you're ultra lazy
* `docker run -i -t -e "MONGOHQ_URL=mongodb://<user>:<pass>@<host>:<port>/<db>" -p 45000:5000 cameron/keen-cache`
* Edit your dashboard's Keen configuration:

```JavaScript
Keen.configure(...);
Keen.client.keenUrl = 'http://<DOCKER_HOST>:4500';
```

# Note for Heroku Users

If you have the MonogHQ addon enabled, your `MONGOHQ_URL` environment variable will be populated automatically.

# Configuration

## Proxy-only API Keys

To restrict clients from hitting Keen directly, generate API keys that only work on the proxy:

* generate a new master key for the proxy
  * `require('crypto').randomBytes(16).toString('hex');`
* generate a new read-only, proxy-only API key
  * in the Keen JS SDK referenced in `package.json`, use `Keen.encryptScopedKey(proxyMasterKey, {'allowedOperations': ['read']})`
  * optionally, add filters or other query constraints as sibling keys of `allowedOperations` in the `Keen.encryptScopedKey` options object to further restrict the proxy-only key
* set the environment variables `KEEN_MASTER_KEY` and `KEEN_PROXY_MASTER_KEY` when running the node process
  * docker: `docker run -e "KEEN_MASTER_KEY=..." -e "KEEN_PROXY_MASTER_KEY=..." ...`
  * heroku: see `.env.sample`

## CORS

Seting the ALLOWED_DOMAINS environment variable to a JSON array of hosts (`['proto://hostname[:port]',...]`) will restrict access to those domains via the `Access-Control-Allow-Origin` header.

## Cache Expiry

Cache expiry (10 minutes) is handled by MongoDB's `expireAfterSeconds` feature on the `cachedAt` index. If you change the timeout, delete the index and restart the app (the index will be re-created automatically).
