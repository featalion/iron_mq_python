IronMQ Python Client Library
-------------

Python language binding for IronMQ. [IronMQ](http://www.iron.io/products/mq) is an elastic message queue for managing data and event flow within cloud applications and between systems. [See How It Works](http://www.iron.io/products/mq/how)

## Getting Started

1\. Install iron_mq_python

```sh
pip install iron-mq
```

or just copy `iron_mq.py` and include it in your script:

```python
from iron_mq import *
```

2\. [Setup your Iron.io credentials](http://dev.iron.io/mq/reference/configuration/)

3\. Create an IronMQ client object:

```python
ironmq = IronMQ()
# or pass credentials
ironmq = IronMQ(project_id='500f7b....b0f302e9', token='Et1En7.....0LuW39Q')
```


## The Basics

### Get Queues List

```python
queues = ironmq.queues() # ['queue1 name', 'queue2 name', ...]
```

returns list of `Queue` objects.

--

### Get a Queue Object

You can have as many queues as you want, each with their own unique set of messages.

```python
queue = ironmq.get('my_queue')
# aliases
queue = ironmq.queue('my_queue')
queue = ironmq.get_queue('my_queue')
```

Now you can use it.

--

### Post a Message on a Queue

Messages are placed on the queue in a FIFO arrangement.
If a queue does not exist, it will be created upon the first posting of a message.

```python
queue.post('hello world!')
```

--

### Retrieve Queue Information

```python
q_info = queue.info()

q_id = queue.id()
q_size = queue.size()
q_name = queue.name
```

--

### Get a Message off a Queue

```python
msg = queue.get()

data = msg.body
```

When you pop/get a message from the queue, it is no longer on the queue but it still exists within the system.
You have to explicitly delete the message or else it will go back onto the queue after the `timeout`.
The default `timeout` is 60 seconds. Minimal `timeout` is 30 seconds.

--

### Delete a Message from a Queue

```python
msg.delete()
```

Be sure to delete a message from the queue when you're done with it.

--


## Important Notes on Backward Compatibility

New client library (version 0.4) includes partial backward compatibility with previous version (0.3).
Please, read this notes before you will update your client library.

* `queue.get()` instantiates `Message` object instead returning a raw response body.
To have the same behavior as in version 0.3 set `instantiate` keyword argument to `False`.

* `queue.post()` accepts list of messages' bodies by default in new version.
To pass prepared `dict` for messages set `raw` keyword argument to `True`.
Example:
```python
message = {'body': 'my body', 'timeout': 300, 'delay': 120}
queue.post(message, raw=True)
```

* `queue.delete(message_id)` is deprecated.
Because `queue.delete()` method is used for queue deletion (not supported in version 0.3).

* List of unsupported methods (all of them were deprecated in version 0.3):
  * `getQueues()`
  * `getQueueDetails()`
  * `deleteMessage()`
  * `postMessage()`
  * `getMessage()`
  * `clearQueue()`

We are highly recomend to use new syntax explained hereinafter.


## IronMQ

`IronMQ` class uses `IronClient` from [iron_core_python](https://github.com/iron-io/iron_core_python) and provides easy access to the queues.

```python
ironmq = IronMQ(config='config.ini')
# or
ironmq = IronMQ(project_id='PROJECT_ID', token='TOKEN')
```

### List Queues

Get queues' names list:

```python
queues = ironmq.queues()
```

Get list of instantiated queues:

```python
queues = ironmq.queues(instantiate=True)
```

**Optional parameters:**

* `page`: The 0-based page to view. The default is 0.
* `per_page`: The number of queues to return per page. The default is 30, the maximum is 100.
* `instantiate`: Instantiate queues instead of returning names list. The default is `False`.

```python
queues = ironmq.queues(page=2, per_page=5, instantiate=True)
```

The method returns list of queues

--

### Get Queue by Name

```python
queue = ironmq.get('my_queue')
# aliases
queue = ironmq.queue('my_queue')
queue = ironmq.get_queue('my_queue')
```

**Note:** if queue with desired name does not exist it returns fake queue.
Queue will be created automatically on post of first message or queue configuration update.

--

## Queues

### Retrieve Queue Information

```python
info = queue.info() # {'id': '5127bf043264140e863e2283', 'name': 'my_queue', ...}

q_id = queue.id() # "5127bf043264140e863e2283"
# Does queue exists on server? Alias for `queue.id() is None`
is_new = queue.is_new() # False

size = queue.size() # 7
name = queue.name # 'my_queue'
overall_messages = queue.total_messages() # 13
subscribers = queue.subscribers() # [Subscription, ...]

push_type = queue.push_type() # 'multicast'
# Does queue Push Queue? Alias for `queue.push_type() is not None`
is_push_queue = queue.is_push_queue() # True
```

**Warning:** to be sure configuration information is up-to-date
client library call IronMQ API each time you request for any parameter except `queue.name`.
In this case you may prefer to use `queue.info()` to have `dict` with all available info parameters.

If you want to have new `Queue` object as response set `instantiate` parameter to `True`:

```python
updated_queue = queue.info(instantiate=True)
```

--

### Delete a Message Queue

```python
queue.delete() # True
```

Returns `True` if queue is deleted. If queue is not found the method returns `False`.
Otherwise raises exception for HTTP status.

--

### Post Messages to a Queue

**Single message:**

```python
queue.post('something helpful', timeout=300)

# or
my_msg = ['my', 'first', 'message']

# post as 1 message
queue.post(my_msg, timeout=300)
```

**Multiple messages:**

```python
messages = ['my', 'three', 'messages']

# post as 3 messages
queue.post(*messages, timeout=120, delay=42)
```

**Optional parameters for messages:**

* `timeout`: After timeout (in seconds), item will be placed back onto queue.
You must delete the message from the queue to ensure it does not go back onto the queue.
 Default is 60 seconds. Minimum is 30 seconds. Maximum is 86,400 seconds (24 hours).

* `delay`: The item will not be available on the queue until this many seconds have passed.
Default is 0 seconds. Maximum is 604,800 seconds (7 days).

* `expires_in`: How long in seconds to keep the item on the queue before it is deleted.
Default is 604,800 seconds (7 days). Maximum is 2,592,000 seconds (30 days).

--

### Get Messages from a Queue

```python
message = queue.get() # Message

# or N messages
messages = queue.get(count=7, timeout=300) # [Message, ...]
```

**Optional parameters:**

* `count`: The maximum number of messages to get. Default is 1. Maximum is 100.

* `timeout`: timeout: After timeout (in seconds), item will be placed back onto queue.
You must delete the message from the queue to ensure it does not go back onto the queue.
If not set, value from POST is used. Default is 60 seconds. Minimum is 30 seconds.
Maximum is 86,400 seconds (24 hours).

When `count` parameter is specified and greater than 1 method returns `list` of `Queue`s.
Otherwise, `Queue` instance will be returned.

--

### Touch a Message on a Queue

Touching a reserved message extends its timeout by the duration specified when the message was created, which is 60 seconds by default.

```python
message = queue.get() # Message

message.touch()
```

--

### Release Message

```python
message = queue.get() # Message

message.release() # True
# or
message.release(delay=42) # True
```

**Optional parameters:**

* `delay`: The item will not be available on the queue until this many seconds have passed.
Default is 0 seconds. Maximum is 604,800 seconds (7 days).

--

### Delete a Message from a Queue

```python
message = queue.get()

message.delete() # True
```

--

### Peek Messages from a Queue

Peeking at a queue returns the next messages on the queue, but it does not reserve them.

```python
message = queue.peek() # Message

# or multiple messages
messages = queue.peek(count=13) # [Message, ...]
```

**Optional parameters:**

* `count`: The maximum number of messages to peek. Default is 1. Maximum is 100.

--

### Clear a Queue

```python
queue.clear() # True
```


## Push Queues

IronMQ push queues allow you to setup a queue that will push to an endpoint, rather than having to poll the endpoint. 
[Here's the announcement for an overview](http://blog.iron.io/2013/01/ironmq-push-queues-reliable-message.html). 

### Update a Message Queue

```python
queue.update(
    subscribers=[
        'http://endpoint.com/first',
        'http://endpoint.com/second'
    ],
    push_type='multicast',
    retries=5,
    retries_delay=120
)
```

**The following parameters are all related to Push Queues:**

* `subscribers`: An array of subscribers dicts containing a “url” field.
This set of subscribers will replace the existing subscribers.
To add or remove subscribers, see the add subscribers endpoint or the remove subscribers endpoint.
See below for example json.
* `push_type`: Either `'multicast'` to push to all subscribers or `'unicast'` to push to one and only one subscriber. Default is `multicast`.
* `retries`: How many times to retry on failure. Default is 3.
* `retries_delay`: Delay between each retry in seconds. Default is 60.

--

### Set Subscribers on a Queue

Subscribers can be any HTTP endpoint. `push_type` is one of:

* `multicast`: will push to all endpoints/subscribers
* `unicast`: will push to one and only one endpoint/subscriber

```python
ptype = 'multicast'
subscribers = [
    'http://rest-test.iron.io/code/200?store=key1',
    'http://rest-test.iron.io/code/200?store=key2'
]

queue.update(subscribers=subscribers, push_type=ptype)
```

--

### Add/Remove Subscribers on a Queue

```python
queue.subscribe('http://nowhere.com')

queue.subscribe([
    'http://first.endpoint.com/process',
    {'http://second.endpoint.com/process'
])

queue.unsubscribe('http://nowhere.com')

queue.unsubscribe([
    'http://first.endpoint.com/process',
    'http://second.endpoint.com/process'
])
```

--

### Get Message Push Status

When post a message to Push Queue set `instantiate` keyword argument to `True`.
In this case `queue.post()` returns fake instance of `Message` (for single object post) or
`list` of fake `Message`s (for multiple object post). This fake instances are only usable for Push Queues
because stores only ID. It provides easy access to check push status.

If you want to check push status later you can get fake message by ID. 
To use the feature call `queue.get_push_message(message_id)`.

```python
# check right after post
msg = queue.post('push me!', instantiate=True)
subscriptions = msg.get_push_status()

# delayed check
response = queue.post('push me!')
# store ID somewhere
store_message_id_your_way(response['ids'][0])
# get the message
msg_id = get_message_id_your_way()
message = queue.get_push_message(msg_id)
subscriptions = message.get_push_status()
```

Returns an array of subscribers with status.

--

### Acknowledge / Delete Message Push Status

```python
subscriptions = queue.get_push_message(msg.id).push_status()

for subscription in subscriptions:
    subscription.acknowledge()
    # subscription.delete() # is alias for `acknowledge()`
```

--


## Further Links

* [IronMQ Overview](http://dev.iron.io/mq/)
* [IronMQ REST/HTTP API](http://dev.iron.io/mq/reference/api/)
* [Push Queues](http://dev.iron.io/mq/reference/push_queues/)
* [Other Client Libraries](http://dev.iron.io/mq/libraries/)
* [Live Chat, Support & Fun](http://get.iron.io/chat)

-------------
© 2011 - 2013 Iron.io Inc. All Rights Reserved.
