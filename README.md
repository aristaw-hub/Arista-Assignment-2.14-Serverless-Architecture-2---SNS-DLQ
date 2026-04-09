# Arista-Assignment-2.14-Serverless-Architecture-2---SNS-DLQ
Arista Assignment 2.14 Serverless Architecture 2 - SNS DLQ

# Serverless Architecture - Advanced Concepts

## Task: In-Class Activity Discussion

### 1. Does SNS guarantee exactly once delivery to subscribers?

**No, Amazon SNS does not guarantee exactly-once delivery.**  
It guarantees **at-least-once** delivery. This means a message may be delivered more than once to a subscriber endpoint. Possible reasons include:
- Network timeouts or retries
- Subscriber endpoint returning transient errors  
- SNS retry policies  

Therefore, subscribers must be designed to be **idempotent** (handling duplicate messages gracefully).

---

### 2. What is the purpose of the Dead-letter Queue (DLQ)? This is a feature available to SQS/SNS/EventBridge.

A **Dead-letter Queue (DLQ)** is a destination (typically an SQS queue) where messages are sent after they fail to be processed successfully a specified number of times.

**Purpose:**
- **Isolate failed messages** for analysis without blocking the main queue or workflow.
- **Prevent message loss** when a subscriber or consumer cannot process a message.
- **Debug and troubleshoot** by inspecting problematic messages.
- **Manual reprocessing** after fixing the root cause.

**How it works per service:**
- **SQS:** After `maxReceiveCount` exceeded, message moves to DLQ.
- **SNS:** After delivery retries to subscriber endpoints fail, message sent to DLQ.
- **EventBridge:** Failed invocations (target errors) can be routed to a DLQ.

---

### 3. How would you enable a notification to your email when messages are added to the DLQ?

**Step-by-step solution (AWS Console):**

1. **Create an SQS Dead-letter Queue** (if not already configured for your source queue).

2. **Create an SNS Topic** named `DLQ-Alerts`:
   - Type: Standard
   - Create subscription: Protocol = Email, Endpoint = your-email@example.com
   - Confirm subscription from the email received.

3. **Create a CloudWatch Alarm** on the DLQ:
   - Go to CloudWatch → All metrics → SQS → Queue Metrics
   - Select your DLQ
   - Metric: `ApproximateNumberOfMessagesVisible`
   - Condition: **> 0** for **1 consecutive datapoint** within **1 minute**

4. **Configure alarm action:**
   - Under “Actions”, select “Send notification to SNS topic”
   - Choose the `DLQ-Alerts` topic created earlier

5. **Test the setup:**
   - Send a failing message to your source queue → after retries, it goes to DLQ
   - Alarm triggers → SNS publishes to your email → you receive an email notification

**Alternative advanced pattern:**  
Use a Lambda function triggered by the DLQ (via SQS trigger) that formats a custom message and sends it via SNS to email or Slack.

---

## References

- AWS Documentation: SNS Delivery Retries and DLQ  
- AWS Documentation: CloudWatch Alarms with SQS  
- Class discussion (Zoom session), NTU, April 2026
