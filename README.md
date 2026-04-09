# Arista-Assignment-2.14-Serverless-Architecture-2---SNS-DLQ
Arista Assignment 2.14 Serverless Architecture 2 - SNS DLQ


# SNS, DLQ, and Notification Setup

## 1. Does SNS guarantee exactly once delivery to subscribers?

**No**, Amazon SNS (Simple Notification Service) does **not** guarantee exactly-once delivery.  
It guarantees **at-least-once** delivery. This means a message may be delivered more than once to a subscriber (e.g., due to retries or network issues). Subscribers should be designed to be idempotent.

## 2. What is the purpose of the Dead-letter Queue (DLQ)? This is a feature available to SQS/SNS/EventBridge.

A **Dead-letter Queue (DLQ)** is used to capture messages that cannot be successfully processed after a specified number of retries.  
- **Purpose**: Isolate failed messages for debugging, analysis, or manual reprocessing without blocking the main queue or losing data.  
- **For SQS**: After maxReceiveCount is exceeded, messages go to DLQ.  
- **For SNS**: Messages that fail delivery to subscribed endpoints (after retry policy) can be sent to a DLQ (SQS).  
- **For EventBridge**: Failed invocations can be sent to a DLQ (SQS or SNS).  

## 3. How would you enable a notification to your email when messages are added to the DLQ?

**Step-by-step (AWS Console):**

1. **Create an SQS DLQ** (if not exists).  
2. **Create an SNS Topic** (e.g., `DLQ-Alerts`).  
3. **Subscribe your email** to the SNS topic (confirm subscription).  
4. **Configure CloudWatch Alarm** on the DLQ:  
   - Go to CloudWatch → Metrics → SQS → Choose your DLQ.  
   - Metric: `ApproximateNumberOfMessagesVisible` (or `NumberOfMessagesSent`).  
   - Set threshold: e.g., > 0 for 1 datapoint within 1 minute.  
5. **Add action to alarm**:  
   - Select SNS topic created in step 2.  
6. **Test**: Send a failing message to the main queue → it goes to DLQ → alarm triggers → email notification sent.

**Alternative**: Use AWS EventBridge rule on SQS DLQ (if supported in future) or Lambda function triggered by DLQ messages that sends email via SNS.
