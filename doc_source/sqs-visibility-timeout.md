# Visibility Timeout<a name="sqs-visibility-timeout"></a>

When a consumer receives and processes a message from a queue, the message remains in the queue\. Amazon SQS doesn't automatically delete the message\. Because Amazon SQS is a distributed system, there's no guarantee that the consumer actually receives the message \(for example, due to a connectivity issue, or due to an issue in the consumer application\)\. Thus, the consumer must delete the message from the queue after receiving and processing it\.

Immediately after the message is received, it remains in the queue\. To prevent other consumers from processing the message again, Amazon SQS sets a *visibility timeout*, a period of time during which Amazon SQS prevents other consumers from receiving and processing the message\. The default visibility timeout for a message is 30 seconds\. The maximum is 12 hours\.

![\[Visibility Timeout\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/Visibility_Timeout.png)

**Note**  
For standard queues, the visibility timeout isn't a guarantee against receiving a message twice\. For more information, see [At\-Least\-Once Delivery](standard-queues.md#standard-queues-at-least-once-delivery)\.  
FIFO queues allow the producer or consumer to attempt multiple retries:  
If the producer detects a failed `SendMessage` action, it can retry sending as many times as necessary, using the same message deduplication ID\. Assuming that the producer receives at least one acknowledgement before the deduplication interval expires, multiple retries neither affect the ordering of messages nor introduce duplicates\.
If the consumer detects a failed `ReceiveMessage` action, it can retry as many times as necessary, using the same receive request attempt ID\. Assuming that the consumer receives at least one acknowledgement before the visibility timeout expires, multiple retries don't affect the ordering of messages\.
When you receive a message with a message group ID, no more messages for the same message group ID are returned unless you delete the message or it becomes visible\.


+ [Inflight Messages](#inflight-messages)
+ [Setting the Visibility Timeout](#configuring-visibility-timeout)
+ [Changing the Visibility Timeout for a Message](#changing-message-visibility-timeout)
+ [Terminating the Visibility Timeout for a Message](#terminating-message-visibility-timeout)
+ [Visibility Timeout API Actions](#visibility-timeout-api-actions)

## Inflight Messages<a name="inflight-messages"></a>

A message is considered to be *in flight* after it's received from a queue by a consumer, but not yet deleted from the queue\.

For standard queues, there can be a maximum of 120,000 inflight messages per queue\. If you reach this limit, Amazon SQS returns the `OverLimit` error message\. To avoid reaching the limit, you should delete messages from the queue after they're processed\. You can also increase the number of queues you use to process your messages\.

For FIFO queues, there can be a maximum of 20,000 inflight messages per queue\. If you reach this limit, Amazon SQS returns no error messages\.

## Setting the Visibility Timeout<a name="configuring-visibility-timeout"></a>

The visibility timeout begins when Amazon SQS returns a message\. During this time, the consumer processes and deletes the message\. However, if the consumer fails before deleting the message and your system doesn't call the `[DeleteMessage](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_DeleteMessage.html)` action for that message before the visibility timeout expires, the message becomes visible to other consumers and the message is received again\. If a message must be received only once, your consumer should delete it within the duration of the visibility timeout\.

Every Amazon SQS queue has the default visibility timeout setting of 30 seconds\. You can change this setting for the entire queue\. Typically, you should set the visibility timeout to the maximum time that it takes your application to process and delete a message from the queue\. When receiving messages, you can also set a special visibility timeout for the returned messages without changing the overall queue timeout\. For more information, see the best practices in the [Processing Messages in a Timely Manner](general-recommendations.md#processing-messages-timely-manner) section\.

If you don't know how long it takes to process a message, specify the initial visibility timeout \(for example, 2 minutes\) and the period of time after which you can check whether the message is processed \(for example, 1 minute\)\. If the message isn't processed, extend the visibility timeout \(for example, to 3 minutes\)\.

## Changing the Visibility Timeout for a Message<a name="changing-message-visibility-timeout"></a>

When you receive a message from a queue and begin to process it, the visibility timeout for the queue may be insufficient \(for example, you might need to process and delete a message\)\. You can shorten or extend a message's visibility by specifying a new timeout value using the `[ChangeMessageVisibility](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibility.html)` API action\.

For example, if the default timeout for a queue is 60 seconds, 15 seconds have elapsed since you received the message, and you send a `ChangeMessageVisibility` call with `VisibilityTimeout` set to 10 seconds, the 10 seconds begin to count from the time that you make the `ChangeMessageVisibility` call\. Thus, any attempt to change the visibility timeout or to delete that message 10 seconds after you initially change the visibility timeout \(a total of 25 seconds\) might result in an error\.

**Note**  
The new timeout period takes effect from the time you call the `ChangeMessageVisibility` API action\. In addition, the new timeout period applies only to the particular receipt of the message\. `ChangeMessageVisibility` doesn't affect the timeout of later receipts of the message or later queues\.

## Terminating the Visibility Timeout for a Message<a name="terminating-message-visibility-timeout"></a>

When you receive a message from a queue, you might find that you actually don't want to process and delete that message\. Amazon SQS allows you to terminate the visibility timeout for a specific message\. This makes the message immediately visible to other components in the system and available for processing\. 

To terminate a message's visibility timeout after calling `ReceiveMessage`, call [http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibility.html](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibility.html) with `VisibilityTimeout` set to 0 seconds\. 

## Visibility Timeout API Actions<a name="visibility-timeout-api-actions"></a>

The following table lists the API actions you can use to manipulate the visibility timeout\. Use each action's `VisibilityTimeout` parameter to set or get the value\.


| Task | API Action | 
| --- | --- | 
|  Set the visibility timeout for a queue  |  `[SetQueueAttributes](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html)`  | 
|  View the visibility timeout for a queue  |  `[GetQueueAttributes](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_GetQueueAttributes.html)`  | 
|  Set the visibility timeout for received messages without affecting the visibility timeout of the entire queue  |  `[ReceiveMessage](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ReceiveMessage.html)`  | 
|  Extend or terminate a message's visibility timeout  |  `[ChangeMessageVisibility](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibility.html)`  | 
|  Extend or terminate the visibility timeout for up to 10 messages  |  `[ChangeMessageVisibilityBatch](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibilityBatch.html)`  | 