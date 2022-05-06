---
title: "API details"
date: 2022-05-09
description: "KernelCI API building blocks"
weight: 3
---

This guide describes KernelCI API building blocks such as Node model and PubSub interface. It also includes details about environment variables.


## Environment Variables

The instructions about environment file is described [here].(https://kernelci.org/docs/api/getting-started/#create-the-environment-file)

### Set ALGORITHM and ACCESS_TOKEN_EXPIRE_MINUTES in environment file

We need to specify an algorithm for JWT token encoding and decoding. ALGORITHM variable needs to be passed in the parameter for that.
ALGORITHM is set default to HS256.
We have used ACCESS_TOKEN_EXPIRE_MINUTES variable to set expiry time on generated jwt access token.
ACCESS_TOKEN_EXPIRE_MINUTES is set default to None.
If a user wants to change any of the above variables, they should be added to the .env file.


## Nodes

As a proof-of-concept, an object model called `Node` is defined in this API.
It's possible to create new objects and retrieve them via the API.

### Create a Node

To create an object of `Node` model, a `POST` request should be made along with the Node attributes. This requires an authentication token:

```
$ curl -X 'POST' \
  'http://localhost:8001/node' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJib2IifQ.ci1smeJeuX779PptTkuaG1SEdkp5M1S1AgYvX8VdB20' \
  -H 'Content-Type: application/json' \
  -d '{
  "name":"checkout",
  "revision":{"tree":"mainline",
  "url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git",
  "branch":"master",
  "commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26",
  "describe":"v5.16-rc4-31-g2a987e65025e"}
}'
{"_id":"61bda8f2eb1a63d2b7152418","kind":"node","name":"checkout","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":null,"status":"pending", "created":"2022-02-02T11:23:03.157648", "updated":"2022-02-02T11:23:03.157648"}
```

### Getting Nodes back

Reading Node doesn't require authentication, so plain URLs can be used.

To get node by ID, use `/node` endpoint with node ID as a path parameter:

```
$ curl http://localhost:8001/node/61bda8f2eb1a63d2b7152418
{"_id":"61bda8f2eb1a63d2b7152418","kind":"node","name":"checkout","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":null,"status":"pending", "created":"2022-02-02T11:23:03.157648", "updated":"2022-02-02T11:23:03.157648"}
```

To get all the nodes as a list, use the `/nodes` API endpoint:

```
$ curl http://localhost:8001/nodes
[{"_id":"61b052199bca2a448fe49673","kind":"node","name":"checkout","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":null,"status":"pass", "created":"2022-02-01T11:23:03.157648", "updated":"2022-02-02T11:23:03.157648"},{"_id":"61b052199bca2a448fe49674","kind":"node","name":"check-describe","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":"61b052199bca2a448fe49673","status":"pending", "created":"2022-01-02T10:23:03.157648", "updated":"2022-01-02T11:23:03.157648"}]
```

To get nodes by providing attributes, use `/nodes` endpoint with query parameters. All the attributes except node ID can be passed to this endpoint.
In case of ID, please use `/node` endpoint with node ID as described above.

```
$ curl 'http://localhost:8001/nodes?name=checkout&revision.tree=mainline'
[{"_id":"61b052199bca2a448fe49673","kind":"node","name":"checkout","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":null,"status":"pass", "created":"2022-02-01T11:23:03.157648", "updated":"2022-02-02T11:23:03.157648"}]
```

### Update a Node

To update an existing node, use PUT request to `node/{node_id}` endpoint.

```
$ curl -X 'PUT' \
  'http://localhost:8001/node/61bda8f2eb1a63d2b7152418' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJib2IifQ.ci1smeJeuX779PptTkuaG1SEdkp5M1S1AgYvX8VdB20' \
  -H 'Content-Type: application/json' \
  -d '{
  "name":"checkout-test",
  "revision":{"tree":"mainline",
  "url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git",
  "branch":"master",
  "commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26",
  "describe":"v5.16-rc4-31-g2a987e65025e"},
  "created":"2022-02-02T11:23:03.157648"
}'
{"_id":"61bda8f2eb1a63d2b7152418","kind":"node","name":"checkout-test","revision":{"tree":"mainline","url":"https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git","branch":"master","commit":"2a987e65025e2b79c6d453b78cb5985ac6e5eb26","describe":"v5.16-rc4-31-g2a987e65025e"},"parent":null,"status":"pending", "created":"2022-02-02T11:23:03.157648", "updated":"2022-02-02T12:23:03.157648"}
```


## Pub/Sub and CloudEvent

The API provides a publisher / subscriber interface so clients can listen to
events and publish them too.  All the events are formatted using
[CloudEvents](https://cloudevents.io).

### Listen & Publish CloudEvent

The [`client.py`](https://github.com/kernelci/kernelci-api/blob/main/api/client.py) script provides a reference implementation for publishing and listening to events.

For example, in a first terminal:

```
$ docker-compose exec api /bin/sh -c '\
TOKEN=<insert token here> \
/usr/bin/env python3 /home/kernelci/api/client.py \
listen abc'
Listening for events on channel abc.
Press Ctrl-C to stop.
```

Then in a second terminal:

```
$ docker-compose exec api /bin/sh -c '\
TOKEN=<insert token here> \
/usr/bin/env python3 /home/kernelci/api/client.py \
publish abc "Hello KernelCI"'
```

You should see the message appear in the first terminal (and stopping after
pressing Ctrl-C):

```
Message: Hello KernelCI
^CStopping.
```

Meanwhile, something like this should be seen in the API logs:

```
$ docker-compose logs api | tail -4
kernelci-api | INFO:     127.0.0.1:35752 - "POST /subscribe/abc HTTP/1.1" 200 OK
kernelci-api | INFO:     127.0.0.1:35810 - "POST /publish/abc HTTP/1.1" 200 OK
kernelci-api | INFO:     127.0.0.1:35754 - "GET /listen/abc HTTP/1.1" 200 OK
kernelci-api | INFO:     127.0.0.1:36744 - "POST /unsubscribe/abc HTTP/1.1" 200 OK
```

> **Note** The client doesn't necessarily need to be run within the `api`
Docker container, but it's a convenient way of trying things out as it already
has all the Python dependencies installed (essentially, `cloudevents`).