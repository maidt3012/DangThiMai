# Trading Platform System Design (Similar to Binance)

## Features
This system is designed to support the following features:
1. **Buy/Sell Stock**
2. **Order Status Updates**

The architecture should be **resilient to failures**, **scalable**, and **cost-effective** while meeting the given requirements.

---

## System Overview
### Architecture Diagram
![System Diagram](https://github.com/maidt3012/DangThiMai/blob/main/src/problem2/problem2-Page-1.jpg)

### Message Format
Below is the JSON message format for an order request:
```json
{
  "orderId": "12345",
  "type": "SELL",
  "userID": "user1",
  "coinID": "BTC",
  "quantity": 5,
  "price": 100
}
```


Cloud Services Used and Their Roles (no. 1, 2, 3, 4, 5, 6 below is marked in diagram)
1. API Gateway 1
Purpose: Acts as the entry point for handling buy/sell requests from clients and receiving order status updates via WebSockets.
Reasons for Choosing AWS API Gateway:
Secure API Management: Authentication & Authorization, DDoS Protection, AWS WAF
Traffic Management: Load Balancing, Caching, Rate Limiting
Auto-Scaling to handle high request volumes
Logging & Monitoring: AWS CloudWatch, AWS X-Ray
Alternatives Considered: Kong API Gateway (EC2), Nginx with API Gateway on EC2.
2. Lambda 2 for Request Classification
Purpose: Classifies incoming requests into buy or sell queues and stores order connection details for status updates.
Reasons for Choosing AWS Lambda:
Serverless & Cost-Effective (Pay-Per-Use)
Auto-Scalability to handle unpredictable loads
No Infrastructure Management
Alternatives Considered: AWS Fargate, ECS, EKS (for high-volume scenarios).
3. SQS 3 for Order Queues
Purpose: Stores buy and sell orders in separate queues for processing.
Reasons for Choosing AWS SQS:
Fully Managed, Auto-Scaling Queue
Decouples Microservices without requiring a broker
High Reliability & Scalability
Alternatives Considered: AWS MQ (Lower Latency, <10ms).
4. EKS 4 for Order Matching
Purpose: Matches buy and sell orders efficiently using Redis for fast processing.
Reasons for Choosing AWS EKS:
Handles high-throughput order processing efficiently
Cost-Effective hourly pricing model for high workloads
Scalability for future growth
Alternatives Considered: AWS ECS.
5. SNS 5 for Order Event Handling
Purpose: Manages notifications for matched, unmatched, and invalid orders.
Reasons for Choosing AWS SNS:
Low Latency & High Throughput
Fan-Out Model for multiple consumers
Serverless & Cost-Effective
Alternatives Considered: AWS EventBridge.
6. Lambda 6 for Post-Processing
Purpose: Handles tasks such as:
Adding unmatched orders back to the queue.
Sending matched order status updates via API Gateway.
Reasons for Choosing AWS Lambda:
Ideal for Simple, Event-Driven Tasks
Serverless, Cost-Effective, and Auto-Scaling
Alternatives Considered: Deploying a microservice on EKS.

Scaling Strategies for Future Growth

As the product scales, the system can be optimized:
Move from Lambda to EKS for long-running tasks.
Consolidate multiple Lambda functions into a single microservice on EKS.
Implement Load Balancing for EKS to distribute processing load.
Optimize Redis Caching to speed up order matching.
