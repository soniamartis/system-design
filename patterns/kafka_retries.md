## Reliable Reprocessing with Kafka


### Goal
To achieve decoupled, observable error handling without affecting live traffic with Kafka

### Use-case
- Order events from live traffic arrive at the topic and can be consumed by various services(kafka consumers)
- If there is a failure in one service, it shouldnt impact other services
- If the error is retriable, it shold be retried, but at the same time shouldnt affect the ive traffic so that app can serve the live traffic with efficiency
- Non-retriable errors should not be retried
- We should have good visibility into the failed events, along with functions to list, purge and replay the events back for processing
- If there is a bug in the service, it will affect all incoming events, so we should be able to reprocess such events with ease


### Design approach 1
- Use the default kafka retry logic, that retries an event on the main service consumer thread, for a certain number of times, and then moves them into DLQ,
- Also increment consumer offset after moving message into DLQ so that next message is processed
- Pros:
  - Simple to implement
- Cons:
  - Blocks live traffic, as the failed event retries prevent the consumption of other messages on topic
 

## Preferred design approach: Delayed processing queues
- Create multiple topics for retry per consumer service
- Use a DLQ at the very last retry consumer, so that if the event was not successfully processed by any of the intermediate retry consumers, we can just move it to DLQ
- Use exception classification to determine whether an exception is retriable(500 for upstream service) vs non-retriable(NPE)
- If retriable, keep pushing event into retry queues along with a delay time, this event may get processed successfully by one of the retry consumers eventually
- If the event failed on all attempts, or there is a non-retriable exception, push it into DLQ
- Once issue is resolved, these messages can be replayed back onto the first retry topic(note: not the live traffic topic, so that we do not block live traffic)

![image](https://github.com/user-attachments/assets/162542a9-4d56-49c8-ab1e-9a4e16b1229d)



### Kafka nuances
- To introduce a delay on the consumer, we should not use sleep, as that will potentially render the consumer dead, resulting in consumer group rebalances
- This is due to the max.poll.interval config, if there is no poll within this time interval, the consumer is marked as dead
- We rather should use the poll - pause - resume mechanism of consumer
- When message is received, look at the delay time, pause the consumer for that time interval, and then resume
- When a container is paused, it continues polling, but the poll does not return any records

### Reading material
- https://www.uber.com/en-IN/blog/reliable-reprocessing/
- https://medium.com/naukri-engineering/retry-mechanism-and-delay-queues-in-apache-kafka-528a6524f722
