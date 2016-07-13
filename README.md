[![Build Status](https://travis-ci.org/apiaryio/heroku-datadog-drain-golang.svg?branch=master)](https://travis-ci.org/apiaryio/heroku-datadog-drain-golang)

# Heroku Datadog Drain

[![Deploy to Docker Cloud](https://files.cloud.docker.com/images/deploy-to-dockercloud.svg)](https://cloud.docker.com/stack/deploy/?repo=https://github.com/elbaschid/heroku-datadog-drain-golang)

Golang version of [NodeJS](https://github.com/ozinc/heroku-datadog-drain)

Funnel metrics from multiple Heroku apps into DataDog using statsd.

**Note:** you have to run a `statsd` server in addition to the drain.

Supported Heroku metrics:
- Heroku Router response times, status codes, etc.
- Application errors
- Heroku Dyno [runtime metrics](https://devcenter.heroku.com/articles/log-runtime-metrics)

## Get Started

```bash
git clone git@github.com:apiaryio/heroku-datadog-drain-golang.git
cd heroku-datadog-drain-golang
heroku create
heroku config:set ALLOWED_APPS=<your-app-slug> <YOUR-APP-SLUG>_PASSWORD=<password>
git push heroku master
heroku ps:scale web=1
heroku drains:add https://<your-app-slug>:<password>@<this-log-drain-app-slug>.herokuapp.com/ --app <your-app-slug>
```

## Configuration

The drain server can configured through environment variables. Let's say you have a Heroku app called `my-app` that you'd like to enable.

* Add the app name to the `ALLOWED_APPS` environment variables: `ALLOWED_APPS=my-app,another-app`

* Individual settings for the app can then be configured using environment variables prefixed with the app name (uppercased and dashes replaced with underscores):

	```sh
	MY_APP_PASSWORD=<some secure password>
	MY_APP_PREFIX=myapp
	MY_APP_TAGS=myapp,production
	```
* You can now configure a new drain on the Heroku app:	
	```
	$ heroku drains:add http://my-app:<some secure password>@<drain server URL> --app my-app
	``` 
	
### Server Settings

Depending on how and where you are running the `statsd` server, you might need to configure the URL and port for it when starting the drain. You specify it as `server:port` an example is below:

```
STATSD_URL=statsd.myserver.com:8125
```

You can enable debug output on standard out by setting `DATADOG_DRAIN_DEBUG=true`.
	
### All Configuration Values

| Variable                   | Required | Description                                  |
|----------------------------|----------|----------------------------------------------|
| `ALLOWED_APPS=my-app,..`   | yes      | Comma seperated list of app names            |
| `<APP-NAME>_PASSWORD=..`   | yes      | One per allowed app where <APP-NAME> corresponds to an app name from ALLOWED_APPS |
| `<APP-NAME>_TAGS=mytag,..` | no       | Comma seperated list of default tags for each app |
| `<APP-NAME>_PREFIX=yee`    | no       | String to be prepended to all metrics from a given app |
| `STATSD_URL=..`            | no       | Default: localhost:8125 |
| `DATADOG_DRAIN_DEBUG=true` | no       | If DEBUG is set, a lot of stuff will be logged 😄 |


## Deploy on Docker Cloud

This repo defines a [Docker Cloud](https://cloud.docker.com) stack in [docker-cloud.yml](./docker-cloud.yml) including this server, a load balancer and Datadog's own `dogstatsd` server. You should be able to easily deploy this setup to your own Docker Cloud cluster. 