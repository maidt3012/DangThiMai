Provide your solution here:
This is design for below feature similar to Binance trading platform:
    1. Buy/Sell stock
    2. Order status updates
The design also need to align with the given requirements that ensure the system is resilient to failures, scalable, and cost-effective.

- An overview diagram of the services used and what role they play in the system.

Message format
{
  "orderId": "12345",
  "type": "SELL",
  "userID": "user1",
  "coinID": "BTC",
  "quantity": 5,
  "price": 100
}

API gateway 1:
entry point to handle request buy/sell from client and receive order status from the service and send it back to client via websocket

lambda 2: classify incoming request to add them into buy queue or sell queue, save the connection with order id to database to send order update status when it complete processing.
SQS queue 3: store all sell/buy order into 2 sqs queue : sell/buy
EKS 4: together with redis installed help procesing order matching fast, this service will pickup the message from sell/buy queues to process. 
if order matched, it will add message to Match topic in SNS 5, 
if order is no match it will add message to No Match topic in SNS 5, 
if order is expire or invalid.. it will add message to Other topic in SNS 5, 
SNS 5: manage topic for order to send to subscribe to continue step to handle order.
Lambda 6: subcribe to sns topic in SNS 5 to 
add back to queues SELL/BUY the message of match, no match item.
for the matched order, lambda 6_1 will send it status to API gateway with orderId and connection string and update to database


- Elaboration on why each cloud service is used and what are the alternatives considered. 

API gateway: is use for single entry point and single exit point.
Reason to use API gateway: 
- Secure API Management: Authentication & Authorization, DDoS Protection, AWS Shield & WAF help protect against attacks, Rate Limiting & Throttling, Prevents API abuse by limiting requests per user/IP
- Traffic Management & Load Balancing: Handles Millions of Requests per Second, Efficiently routes traffic to backend services (Lambda, EC2, Fargate, etc.), Caching for Performance
Built-in caching reduces API response times and backend load 
- Logging, Monitoring & Analytics: AWS CloudWatch Logging & Metrics, Tracks API usage, error rates, and latencies, AWS X-Ray for Tracing, Provides end-to-end visibility into request flow
- Cost-Effective & Scalable: Pay-Per-Use Pricing ,Only pay for API requests & data transfe,  Auto-Scales on Demand, Handles traffic spikes without manual scaling


Alternative is Kong API gateway with ec2, Nginx with API gateway with ec2

lambda 2: I used lambda to use benefit of serverless and cost per time of usage (because in case low throughput we don't need an persistent host.) But when the system become larger, we will change it to ecs or eks with load balancing.
Reason to use:
No Server Management (Fully Serverless)
Auto-Scalability
Cost-Effective (Pay-Per-Use)
High Availability & Fault Tolerance

Alternative : AWS Fargate, EKS, ECS with load balancing when high request volume


SQS 3: 
Reason to use SQS: 
Ease of Use, fully managed, auto-scaling queue.
a simple queue with no persistent connection.
prioritize simplicity, reliability, and scalability over real-time processing.
decouple microservices without managing a broker.
latency : (10-100ms)
pricing model: Pay-per-request

Alternative is  AWS MQ (low Latency < 10ms>)


EKS 4: 
Reason to use EKS: The order matching process requires significant processing time, making an hourly pricing model a cost-effective choice for handling high-throughput messaging. Additionally, EKS provides a scalable, production-grade environment for Kubernetes workloads.
Alternative is  ECS, 

SNS 5: 
Reason to use SNS: we need publish result of order soon to multiple parties at once. Low Latency & High Throughput, Cost-Effective & Serverless
Alternative is  AWS EventBridge

Lambda 6: handle simple task for adding to queue or update db or send to api gateway so we use lambda for easy and cheap.


Plans for scaling when the product grows beyond your current setup.

When product grow we can change
all lambda to EKS,
lambda 6_1, 6_2, 6_3 can deloy into 1 service deploy in eks.
add load balancing for EKS


