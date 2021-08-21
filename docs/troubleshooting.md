# Troubleshooting

This document contains common problems and their solutions.
## `Refused to Connect` in Modal or Workspace View

This is not an issue with Dashy, but instead caused by the target app preventing direct access through embedded elements. It can be fixed by setting the [`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) HTTP header set to `ALLOW [path to Dashy]` or `SAMEORIGIN`, as defined in [RFC-7034](https://datatracker.ietf.org/doc/html/rfc7034). These settings are usually set in the config file for the web server that's hosting the target application, here are some examples of how to enable cross-origin access with common web servers:

### NGINX
In NGINX, you can use the [`add_header`](https://nginx.org/en/docs/http/ngx_http_headers_module.html) module within the app block.
```
server {
  ...
  add_header X-Frame-Options SAMEORIGIN always;
}
```
Then reload with `service nginx reload`

### Caddy

In Caddy, you can use the [`header`](https://caddyserver.com/docs/caddyfile/directives/header) directive.

```yaml
header {
  X-Frame-Options SAMEORIGIN
}
```

### Apache

In Apache, you can use the [`mod_headers`](https://httpd.apache.org/docs/current/mod/mod_headers.html) module to set the `X-Frame-Options` in your config file. This file is usually located somewhere like `/etc/apache2/httpd.conf

```
Header set X-Frame-Options: "ALLOW-FROM http://[dashy-location]/" 
```

---

## Yarn Error

For more info, see [Issue #1](https://github.com/Lissy93/dashy/issues/1)

First of all, check that you've got yarn installed correctly - see the [yarn installation docs](https://classic.yarnpkg.com/en/docs/install) for more info.

If you're getting an error about scenarios, then you've likely installed the wrong yarn... (you're [not](https://github.com/yarnpkg/yarn/issues/2821) the only one!). You can fix it by uninstalling, adding the correct repo, and reinstalling, for example, in Debian:
- `sudo apt remove yarn`
- `curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
- `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
- `sudo apt update && sudo apt install yarn`

Alternatively, as a workaround, you have several options:
- Try using [NPM](https://www.npmjs.com/get-npm) instead: So clone, cd, then run `npm install`, `npm run build` and `npm start`
- Try using [Docker](https://www.docker.com/get-started) instead, and all of the system setup and dependencies will already be taken care of. So from within the directory, just run `docker build -t lissy93/dashy .` to build, and then use docker start to run the project, e.g: `docker run -it -p 8080:80 lissy93/dashy` (see the [deploying docs](https://github.com/Lissy93/dashy/blob/master/docs/deployment.md#deploy-with-docker) for more info)

---

## DockerHub `toomanyrequests`

This situation relates to error messages similar to one of the following, returned when pulling, updating or running the Docker container from Docker Hub.

```
Continuing execution. Pulling image lissy93/dashy:release-1.6.0 
error pulling image configuration: toomanyrequests
```
or
```
You have reached your pull rate limit. You may increase the limit by authenticating and upgrading: https://www.docker.com/increase-rate-limit
```

When DockerHub returns one of these errors, or a `429` status, that means you've hit your rate limit. This was [introduced](https://www.docker.com/blog/scaling-docker-to-serve-millions-more-developers-network-egress/) last year, and prevents unauthenticated or free users from running docker pull more than 100 times per 6 hours.
You can [check your rate limit status](https://www.docker.com/blog/checking-your-current-docker-pull-rate-limits-and-status/) by looking for the `ratelimit-remaining` header in any DockerHub responses. 

#### Solution 1 - Use an alternate container registry
- Dashy is also availible through GHCR, which at present does not have any hard limits. Just use `docker pull ghcr.io/lissy93/dashy:latest` to fetch the image
- You can also build the image from source, by cloning the repo, and running `docker build -t dashy .` or use the pre-made docker compose

#### Solution 2 - Increase your rate limits
- Logging in to DockerHub will increase your rate limit from 100 requests to 200 requests per 6 hour period
- Upgrading to a Pro for $5/month will increase your image requests to 5,000 per day, and any plans above have no rate limits
- Since rate limits are counted based on your IP address, proxying your requests, or using a VPN may work

---

## Config Validation Errors
The configuration file is validated against [Dashy's Schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.json) using AJV.

First, check that your syntax is valid, using [YAML Validator](https://codebeautify.org/yaml-validator/) or [JSON Validator](https://codebeautify.org/jsonvalidator). If the issue persists, then take a look at the [schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.json), and verify that the field you are trying to add/ modify matches the required format. You can also use [this tool](https://www.jsonschemavalidator.net/s/JFUj7X9J) to validate your JSON config against the schema, or run `yarn validate-config`.

If you're trying to use a recently released feature, and are getting a warning, this is likely because you've not yet updated the the current latest version of Dashy.

If the issue still persists, you should raise an issue.

---

## Warnings in the Console during deploy
Please acknowledge the difference between errors and warnings before raising an issue about messages in the console. It's not unusual to see warnings about a new version of a certain package being available, an asset bundle bing oversized or a service worker not yet having a cache. These shouldn't have any impact on the running application, so please don't raise issues about these unless it directly relates to a bug or issue you're experiencing. Errors on the other hand should not appear in the console, and they are worth looking into further.