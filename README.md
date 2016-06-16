**Status**: Microservices is under active development.

# [Microservices](https://pypi.python.org/pypi/microservices/)

Package for building are microservices and clients.

HTTP service bases on [Browsable Web APIs for Flask](http://www.flaskapi.org)

HTTP client bases on [requests](http://docs.python-requests.org/en/master/)

---

## Overview

[Microservices](https://pypi.python.org/pypi/microservices/) is a package with helpers for building are microservices.

It is currently a work in progress, but the fundamentals are in place and you can already start building your microservices with it. If you want to start using Microservices right now go ahead and do so, but be sure to follow the release notes of new versions carefully.

![Screenshot](docs/screenshot.png)

## Roadmap

Future work on getting Microservices to a 1.0 release will include:

* SQS, AMQP and other transport protocols for microservices API.
* Full documentation
* Classes for building services with microservices context.

It is also possible that the core of Flask API could be refactored into an
external dependency, in order to make browsable APIs easily available to
any Python web framework.

## Installation

Requirements:

* Python 2.7+ or 3.3+

Install using `pip`.

    pip install microservices

Import and initialize your application.

    from microservices.http.service import Microservice

    app = Microservice(__main__)

## Responses

Return any valid response object as normal, or return a `list` or `dict`.

    @app.route(
        '/example/',
        resource=Resource(),
    )
    def example():
        return {'hello': 'world'}

A renderer for the response data will be selected using content negotiation
based on the client 'Accept' header.
If you're making the API request from a regular client,
this will default to a JSON response.
If you're viewing the API in a browser it'll default to the browsable
API HTML.

## Requests

Access the parsed request data using `request.data`.
This will handle JSON or form data by default.

    @app.route(
        '/example/',
        resource=Resource(),
    )
    def example():
        return {'request data': request.data}

## Example

The following example demonstrates a simple API for creating,
listing, updating and deleting notes.

```
from microservices.http.service import Microservice
from microservices.http.runners import base_run as run
from microservices.http.resources import ResourceInfo, Resource
from microservices.utils import set_logging
from flask import request, url_for

microservice = Microservice(__name__)

@microservice.route(
    '/second/',
    resource=Resource(
        info=ResourceInfo(
            resource='Second resource',
            GET='Get second resource',
        ),
        url=True,
    )
)
def second():
    return u'SECOND'

@microservice.route(
    '/second/',
    resource=Resource(
        info=ResourceInfo(
            POST='POST INFO',
        ),
        url=True,
    ),
    methods=['POST'],
)
def second_post():
    return request.data

@microservice.route(
    '/second/<string:test>/',
    resource=Resource(
        info=ResourceInfo(
            POST='POST INFO',
        ),
        url=True,
        url_params=dict(test='something'),
    ),
    methods=['POST', 'GET'],
)
def second_params(test):
    if request.method == 'POST':
        return request.data
    return test

@microservice.route(
    '/second/<string:test>/<int:two>/',
    resource=Resource(
        info=ResourceInfo(
            POST='POST INFO',
        ),
        url=lambda resource: url_for('second', _external=True),
    ),
    methods=['POST', 'GET'],
)
def second_params_two(test, two):
    if request.method == 'POST':
        return [test, two, request.data]
    return [test, two]

@microservice.route(
    '/',
    endpoint='Hello',
    methods=['GET', 'POST'],
    resource=Resource(
        info=ResourceInfo(
            resource=u'Hello world!',
            GET=u'Ask service about hello',
            POST=u'Answer for hello'
        ),
        url=True,
    ),
)
def hello():
    if request.method == 'POST':
        return request.data
    return u"POST something for hello"

if __name__ == "__main__":
    set_logging()
    run(microservice, debug=True)
```

Now run the microservice:

    $ python ./example.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

And open <http://127.0.0.1:5000/>.
You can then navigate between notes, and make `GET`, `PUT`, `POST`
and `DELETE` API requests.

Client for app:

```
from microservices.http.client import Client
from microservices.utils import set_logging, get_logger

set_logging(level='INFO')
logger = get_logger('microservices client')

client = Client(
    'http://localhost:5000/',
)

logger.info(client.get(key='response'))
logger.info(client.post(data={'test': 'tested'}))

second_resource = client.resource('second')

logger.info(second_resource.get(key='response'))
logger.info(second_resource.post(data={'test': 'tested'}))

logger.info(second_resource.get('test', key='response'))
logger.info(second_resource.post('test'))

one_two_resource = second_resource.resource('one', '2')
logger.info(one_two_resource.get(key='response'))
logger.info(one_two_resource.post(data={'test': 'tested'}))
```

After run you will see:

```
2016-06-16 14:11:10,997 - microservices.http.client - INFO - get: http://localhost:5000/
2016-06-16 14:11:11,000 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,002 - microservices client - INFO - POST something for hello
2016-06-16 14:11:11,002 - microservices.http.client - INFO - post: http://localhost:5000/
2016-06-16 14:11:11,003 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,004 - microservices client - INFO - {u'test': u'tested'}
2016-06-16 14:11:11,004 - microservices.http.client - INFO - get: http://localhost:5000/second/
2016-06-16 14:11:11,005 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,006 - microservices client - INFO - SECOND
2016-06-16 14:11:11,006 - microservices.http.client - INFO - post: http://localhost:5000/second/
2016-06-16 14:11:11,007 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,008 - microservices client - INFO - {u'test': u'tested'}
2016-06-16 14:11:11,008 - microservices.http.client - INFO - get: http://localhost:5000/second/test/
2016-06-16 14:11:11,009 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,010 - microservices client - INFO - test
2016-06-16 14:11:11,010 - microservices.http.client - INFO - post: http://localhost:5000/second/test/
2016-06-16 14:11:11,011 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,012 - microservices client - INFO - {}
2016-06-16 14:11:11,012 - microservices.http.client - INFO - get: http://localhost:5000/second/one/2/
2016-06-16 14:11:11,012 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,014 - microservices client - INFO - [u'one', 2]
2016-06-16 14:11:11,014 - microservices.http.client - INFO - post: http://localhost:5000/second/one/2/
2016-06-16 14:11:11,015 - requests.packages.urllib3.connectionpool - INFO - Starting new HTTP connection (1): localhost
2016-06-16 14:11:11,016 - microservices client - INFO - {u'response': [u'one', 2, {u'test': u'tested'}]}
```

## Credits

Many thanks to [Tom Christie](https://github.com/tomchristie/) for making the `flask_api`.

[pypi-link]: https://pypi.python.org/pypi/microservices/
[flask-api-link]: https://github.com/tomchristie/flask-api
