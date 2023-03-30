# aws-telegram-bot
Documentations about Telegram bot on AWS

Telegram Bot may receive updates via Getupdates or Webhook. For now, Bow uses Webhook cause this way is cheaper.

## Receiving messages

Telegram sends updates to Webhook one by one synchronously. Until one update is not handled next one will not be sent.
To allow ongoing updates, we will save all incoming Webhook requests to AWS SQS. The solution for receiving Telegram updates is based on [this article](https://medium.com/wearewaes/how-to-build-a-reliable-scalable-and-cost-effective-telegram-bot-58ae2d6684b1). 

### Around queue properties

It's planned that one message should be processed not more than 20 seconds in total. Our consumer will receive only one messages maximum as a batch request. Let's have a visibility timeout for 1 minutes.

Unprocessable messages have to be skipped, it will be sent to dead-letter queue. This one should be monitored.

It's not planned to guarantee to process all messages. If there are some troubles with processing - old messages could be dropped. Let's have a message retention period of 1 hour.

### Around template properties

We assume that our API Gateway receives only valid updates from Telegram. In this case, to duplicate, let's have MessageDeduplicationId as the Telegram update id. 

Messages from one channel should be parsed one by one. To implement it, let's have the same MessageGroupId for all messages from all channels. As the ideal way, we should set MessageGroupId as the Telegram chat id.

```
Action=SendMessage&MessageGroupId=1&MessageDeduplicationId=$input.path('$.update_id')&MessageBody=$util.base64Encode($input.body)
```

### One more thing

Also, the article doesn't mention that you have to publish the resource.

## Processing updates

All updates are being processed synchronously. 

All steps have timeouts. If a timeout expires, it means an error appears.

All errors are saved in the log. All errors should be sent as a reply if it's possible.
