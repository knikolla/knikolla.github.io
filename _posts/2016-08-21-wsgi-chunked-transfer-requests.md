---
layout: post
title: Chunked Transfer Encoded Requests
tags: [Python]
---

Handling requests with Chunked Transfer Encoding in Python with Apache/mod_wsgi and uWSGI.

<!--more-->

# What is Chunked Transfer Encoding?
[Chunked Transfer Encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding) is a method introduced in HTTP 1.1 for sending data as a series of successive chunks. The `Content-Length` header is not set, therefore nor the sender nor the receiver need to know the size of the entire request beforehand. This allows the sender to start transmitting without buffering the entire request in-memory.

To tell the server that the request is being sent in chunks, the client sets the `Transfer-Encoding: chunked` header. The size of the chunk is then sent in the body alongside the chunk divided by a line separator, so the body of a request would look something like this. 

	7
	Chunked
	7
	Message


# Reading Chunked Requests
As chunked encoded is not part of the WSGI specification, dealing with such requests is quite tricky. For example: Flask reports the body of the request as empty because of the missing (or zero) `Content-Length` header.

The pieces of code below are based on Flask, but they should be fairly easy to convert to whatever framework is being used. All that needs to change is how to access the environ variables.

Also, the code is used to create a generator, as that was my use case.

## Apache and mod_wsgi
I haven't been able to get Apache with mod_wsgi to correctly read the entire body while in daemon mode, and this piece of code only works in embedded mode.

In the site configuration file, add `WSGIChunkedRequest On`.

```python
def chunked_reader():
    stream = flask.request.environ["wsgi.input"]
    try:
        while True:
            yield stream.next()
    except:
        return
```
[[Source]](http://stackoverflow.com/a/12132088)

## uWSGI
uWSGI worked generally better. Chunked input needs to be enabled either through the command line argument `--http-chunked-input` or through the configuration file by a similarly named option.

When running a python application under uWSGI, the server imports a special module called `uwsgi` which includes server-specific functions. In this case we're interested in the `chunked_read()` function. There is also a non-blocking function called `chunked_read_nb()`.

```python
import uwsgi

def chunked_reader()
    while True:
        chunk = uwsgi.chunked_read()
        if len(chunk) > 0:
            yield chunk
        else:
            return
```
[[Source]](http://uwsgi-docs.readthedocs.io/en/latest/Chunked.html)

