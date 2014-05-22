## Changes

TODO: 
batch delete
peek
webhook


Changes from v2:

- Per-message expirations turn into per-queue expirations
- Timed out and released messages go to the front of the queue. (This
is not an API change, but it is a behavior change that will likely
cause some tests to fail.)
- Push queues must be explicitly created. There's no changing a queue's type.
- All json objects are wrapped at the root level.
- All object structures changed a bit, please review json.
- Clear messages endpoint changed to be part of delete messages.
- Can no longer set timeout when posting a message, only when reserving one.
- webhook url is no longer /queues/{queue_name}/messages/webhook, it's now /queues/{queue_name}/webhook

## Contents

1. [Global Stuff](#global-stuff)
2. [Queues](#queues)
  1. [Create Queue](#create-queue)
  2. [Get Queue](#get-queue)
  3. [Update Queue](#update-queue)
  4. [Delete Queue](#delete-queue)
  4. [List Queues](#list-queues)
  5. [Add Subscriber](#add-subscriber)
  6. [Remove Subscriber](#remove-subscriber)
  5. [Add Alert](#add-alert)
  6. [Remove Alert](#remove-alert)
1. [Messages](#messages)
  1. [Post Message](#post-messages) - Core operation to add messages to a queue
  2. [Post Message via Webhook](#post-message-via-webhook)
  2. [Reserve/Get Messages](#reserve-messages) - Core operation to get message(s) off the queue.
  2. [Get Message by Id](#get-message-by-id)
  2. [Peek Messages](#peek-messages) - View first messages in queue without reserving them
  3. [Delete Message](#delete-message) - Core operation to delete a message after it's been processed
  3. [Delete Messages](#delete-messages) - Batch delete
  2. [Release Message](#release-message) - Makes a message available for another process
  3. [Touch Message](#touch-message) - Extends the timeout period so process can finish processing message
  3. [Clear Messages](#clear-messages) - Removes all messages from a queue


## Global Stuff

Base path: `/3/projects/{project_id}`

All requests:

Headers:

- Content-type: application/json

Authentication

Headers:

- Authorization: OAuth TOKEN

## Queues

### Create Queue

POST `/queues/{queue_name}`

Request:

All fields are optional.

If `push` field is defined, this queue will be created as a push queue and must contain at least one subscriber. Everything else in the push map is optional.

A `push` queue cannot have alerts.

```json
{
  "queue": {
    "timeout": 60,
    "push":{
      "subscribers": [
        {
          "url": "http://mysterious-brook-1807.herokuapp.com/ironmq_push_1",
          "headers": {"Content-Type": "application/json"}
        }
      ],
      "retries": 3,
      "retries_delay": 60,
      "error_queue": "error_queue_name"
    },
    "alerts": [
      {
       "type": "fixed",
       "trigger": 100,
       "direction": "asc",
       "queue": "target_queue_name",
       "snooze": 60
      }
    ]
  }
}
```

Response: 201 Created

SAME AS GET QUEUE INFO

### Get Queue Info

GET `/queues/{queue_name}`

Response: 200 or 404

Some fields will not be included if they are not applicable like `push` if it's not a push queue and `alerts` if
there are no alerts.

```json
{
  "queue": {
    "id": 123,
    "name": "my_queue",
    "created_at": "2014-12-19T16:39:57-08:00",
    "updated_at": "2014-12-19T16:39:57-08:00",
    "timeout": 60,
    "type": "pull/unicast/multicast",
    "expire_time": 604800,
    "push":{
      "subscribers": [
        {
          "url": "http://mysterious-brook-1807.herokuapp.com/ironmq_push_1",
          "headers": {"Content-Type": "application/json"}
        }
      ],
      "retries": 3,
      "retries_delay": 60,
      "error_queue": "error_queue_name"
    },
    "alerts": [
      {
        "type": "fixed",
        "trigger": 100,
        "direction": "asc",
        "queue": "target_queue_name",
        "snooze": 60
      }
    ]
  }
}
```


### Update Queue

PATCH `/queues/{queue_name}`

Request:

SAME AS CREATE QUEUE

Response: 200 or 404

Some fields will not be included if they are not applicable like `push` if it's not a push queue and `alerts` if
there are no alerts.

SAME AS GET QUEUE INFO

### Delete Queue

DELETE `/queues/{queue_id}`

Request:

```json
{}
```

Response: 200 or 404

```json
{
  "msg": "Queue deleted."
}
```


### List Queues

GET `/queues`

Lists queues in alphabetical order.

Request:

- per_page - number of elements in response, default is 30.
- previous - this is the last queue on the previous page, it will start from the next one.

Response: 200 or 404

Some fields will not be included if they are not applicable like `push` if it's not a push queue and `alerts` if
there are no alerts.

```json
{
  "queues": [ "list of queues with same info as get queue info" ]
}
```

SAME AS GET QUEUE INFO







## Messages

### Post Messages

POST `/queues/{queue_name}/messages`

Request:

```json
{
  "messages": [
    {
      "body": "This is my message 1.",
      "delay": 0
    }
  ]
}
```

Response: 201 Created

Returns a list of message ids in the same order as they were sent in.

```json
{
  "ids": [
    "2605601678238811215"
  ],
  "msg": "Messages put on queue."
}
```


### Reserve Messages

POST `/queues/{queue_name}/reservations`

Request:

All fields are optional.

- n: The maximum number of messages to get. Default is 1. Maximum is 100. Note: You may not receive all n messages on every request, the more sparse the queue, the less likely you are to receive all n messages.
- timeout:  After timeout (in seconds), item will be placed back onto queue. You must delete the message from the queue to ensure it does not go back onto the queue. If not set, value from queue is used. Default is 60 seconds, minimum is 30 seconds, and maximum is 86,400 seconds (24 hours).

```json
{
  "n": 1,
  "timeout": 60
}
```

Response: 200

```json
{
  "messages": [
    {
       "TODO": "SAME AS GET MESSAGE BY ID plus:",
       "reservation_id": "def456"
    }
  ]
}
```

Will return an empty array if no messages are available in queue.

### Get Message by Id

GET `/queues/{queue_name}/messages/{message_id}`

Response: 200

Some fields will not be included if they are not applicable like `push` if it's not a push queue and `alerts` if
there are no alerts.


```json
{
  "message": {
    "id": 123,
    "created_at": "2014-12-19T16:39:57-08:00",
    "updated_at": "2014-12-19T16:39:57-08:00",
    "body": "This is my message 1.",
    "delay": 0,
    "reserved_until": "2014-12-19T16:39:57-08:00",
    "reserved_count": 1,
    "timeout": 60,
    "todo": "push related info"
  }
}
```

### Peek Messages

GET `/queues/{queue_name}/messages`

Request:

- n: The maximum number of messages to peek. Default is 1. Maximum is 100. Note: You may not receive all n messages on every request, the more sparse the queue, the less likely you are to receive all n messages.

Response: 200

Some fields will not be included if they are not applicable like `push` if it's not a push queue and `alerts` if
there are no alerts.

```json
{
  "messages": [
    {
       "TODO": "SAME AS GET MESSAGE BY ID"
  ]
}
```

### Delete Message

DELETE `/queues/{queue_name}/messages/{message_id}`

Request:

- reservation_id: This id is returned when you [reserve a message](#reserve-messages) and must be provided to delete a message that is reserved.
If a reservation times out, this will return an error when deleting so the consumer knows that some other consumer will be processing this message and can rollback or react accordingly.
If the message isn't reserved, it can be deleted without the reservation_id.

```json
{
  "reservation_id": "def456"
}
```

Response: 200 or 404

```json
{
  "msg": "Message deleted."
}
```

### Delete Messages

DELETE `/queues/{queue_name}/messages`

This is for batch deleting messages. Maximum number of messages you can delete at once is 100.

Request:

- reservation_id: This id is returned when you [reserve a message](#reserve-messages) and must be provided to delete a message that is reserved. If a reservation times out, this will return an error when deleting so the worker knows that some other worker will be processing this message and can rollback or react accordingly.

TODO: should reservation_id be optional? if not included, a message can be deleted as long as it's not reserved. This is how marconi works.

```json
{
  "ids": [
    "id": 123,
    "reservation_id": "abc"
  ]
}
```

Response: 200 or 404

```json
{
  "msg": "Messages deleted."
}
```

### Touch Message

DELETE `/queues/{queue_name}/messages/{message_id}/touch`

Request:

```json
{}
```

Response: 200 or 404

```json
{
  "msg": "Message touched."
}
```


### Release Message

DELETE `/queues/{queue_name}/messages/{message_id}/release`

Request:

```json
{
  "delay": 60
}
```

Response: 200 or 404

```json
{
  "msg": "Message released."
}
```



### Clear Messages

DELETE `/queues/{queue_name}/messages`

This will remove all messages from a queue.

Request:

```json
{

}
```

Response: 200 or 404

```json
{
  "msg": "Queue cleared."
}
```
