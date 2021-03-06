EF URL Shortener
================

[![Build Status](https://travis-ci.org/ellisonleao/ef-url-shortener.svg?branch=master)](https://travis-ci.org/ellisonleao/ef-url-shortener)
[![Coverage Status](https://coveralls.io/repos/github/ellisonleao/ef-url-shortener/badge.svg)](https://coveralls.io/github/ellisonleao/ef-url-shortener)

A URL Shortening microservice API

## Prerequisites:

- MongoDB
- Python 3

Installing:


- With pip

```
pip install git+git://github.com/ellisonleao/ef-url-shortener.git
```

- Cloning

A [virtualenv](https://pypi.python.org/pypi/virtualenv) is recommended

```bash
git clone github.com/ellisonleao/ef-url-shortener
cd ef-url-shortener
pip install -r requirements.txt
```

## Running locally:

Some env vars are required for the service:

- **`HOST`** - Shortening URL Host
- **`MONGO_URL`** - MongoDB url

There is also a optional:

- **`PORT`** - Run the service on http port


To run the project, execute the following:

```bash
export HOST="http://somehost"
export MONGO_URL="mongodb://mongourl/dbname"

make run
```

you can also use just

```
make run
```

which will default the `HOST` to `http://ef.me` and `MONGO_URL` to `mongodb://localhost:27017/ef_shortener` and

## Testing

In order to test the project, create a `MONGODB_URI_TEST` env variable pointing to a test mongo db, then type:

```
make test
```

## Deploying


You can deploy directly on Heroku by clicking below

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)


## Endpoints:

- All API requests must pass a `X-Api-Key` header with the generated api key for the user.
- All API requests must use `application/json` as payload content-type on post requests

## `POST /api/user`

**Add user**

Parameters:

- `email` : A valid email address

Example request:

```bash
curl -XPOST http://host/api/user -d '{"email": "a valid email"} -H 'Content-Type: application/json'
```

Example response:

```
HTTP/1.0 200 OK
Date: GMT Date
Server: Some web server
content-length: 143
content-type: application/json

{
    "api_key": "4cc5963bf4682d1cd6d855f3cd45cb765d2f743f883f459cc6c7ee421d9ee927e89a5357bd04586483ea00094c8b6656bbbfaca3c8b7513d6507a85f16ebcdcd"
}
```

That api key should be used for every user request.


## `GET /api/short`

**Short url**

This endpoint creates a short url, given a long one

Parameters:

- **`long_url`** - Valid URL
- **`code`** - (Optional) custom code for short url. Max length: 9 chars

Example request:

```bash
curl http://host/api/short?long_url=http://google.com -H 'X-Api-Key: userapikey'
```

Example response

```
HTTP/1.0 201 Created
Date: GMT Date
Server: Some web server
content-length: 39
content-type: application/json

{
    "short_url": "http://host/XYZCxoXeS"
}
```

Other responses:

- `409` - Already added long_url
- `400` - Bad request
- `401` - Unauthorized request


## `GET /api/expand`

**Expand url**

This endpoint returns information about the original url and the short url.

Parameters:

- `short_url` - Valid URL

Example request:

```bash
curl http://host/api/expand?short_url=http://host/code -H 'X-Api-Key: userapikey'
```

Example response

```
HTTP/1.0 200 OK
Date: GMT Date
Server: Some web server
content-length: 75
content-type: application/json

{
    "long_url": "http://somelongurl.com",
    "short_url": "http://host/code"
}
```

Other responses:

- `404` - Short url not found
- `400` - Bad request
- `401` - Unauthorized request


## `GET /api/urls`

**User urls**

This endpoint returns a list of urls for a determinated user.

Parameters:

- `page` - Get specific page

Example request:

```bash
curl http://host/api/user?page=2 -H 'X-Api-Key: userapikey'
```

Example response

```
HTTP/1.0 200 OK
Date: GMT Datetime
Server: Some web server
content-length: 591
content-type: application/json

[
    {
        "long_url": "http://www.google.com/123",
        "short_url": "http://host/somecode",
        "total_accesses": 6,
        "url_access": [
            {
                "date": "2017-03-20T17:06:41.876000"
            },
            {
                "date": "2017-03-20T17:06:43.402000"
            },
            {
                "date": "2017-03-20T17:06:44.232000"
            },
            {
                "date": "2017-03-20T17:15:29.990000"
            },
            {
                "date": "2017-03-20T17:15:30.911000"
            },
            {
                "date": "2017-03-20T17:15:31.747000"
            }
        ]
    },
    {
        "long_url": "http://www.g1.com.br",
        "short_url": "http://host/code",
        "total_accesses": 0,
        "url_access": []
    }
]
```

Other responses:

- `401` - Unauthorized request


## `GET /api/urls/:code`

**Return a user url by code**

This endpoint returns a single url information.

Example request:

```bash
curl http://host/api/urls/code -H 'X-Api-Key: userapitoken'
```

Example response:

```
HTTP/1.0 200 OK
Date: GMT Datetime
Server: Some web server
content-length: 179
content-type: application/json

{
    "code": "sDzlSqcTh",
    "created_at": "2017-03-21T20:37:57.190000",
    "long_url": "http://www.g1.com.br",
    "short_url": "http://ef.me/sDzlSqcTh",
    "total_accesses": 0,
    "url_access": []
}
```

Other responses:

- `404` - url not found
- `400` - Bad request
- `401` - Unauthorized request


## `GET /s/:code`

**Short url redirect**

Endpoint for the shorl url redirect. Ideally a new location would be added on a web server to this endpoint to handle the short url hostname.

Example request:

```
curl http://host/code
```

Example response:

```
HTTP/1.0 301 Moved Permanently
Date: GMT Datatime
Server: Some web server
content-length: 0
content-type: application/json
location: http://some.longurl
```


## Improvements Roadmap

Several things could be added as improvements for scalability and security:

- **OAuth2 for authentication** - using a client secret with a client id the user could generate a token for requests. That token should be invalidated at some point. Some social auth could be added too
- **Caching** - Adding a caching layer on top of the api will improve the response time. Varnish cache could be an option.
- **Nginx** - Nginx could be used for web server, load balancer and reverse proxy to configure the short url host. Currently the service is running with gunicorn+meinheld , but we could configure to use with uwsgi or other wsgi handler.
- **Logging** - Add some log handler. Could be [Sentry](https://sentry.io/) for error logs and some other handler for access logs.
- **Mocks on tests** - Right now the tests are using a real mongodb connection. Optimal way is to mock the db connection and other db operations.

