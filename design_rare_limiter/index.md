# Step 1 - Understand the problem and establish design scope

- **Candidate**: What kind of rate limiter are we going to design? Is it a client-side rate limiter or server-side API rate limiter?
- **Interviewer**: Great question. We focus on the server-side API rate limiter.
- **Candidate**: Does the rate limiter throttle API requests based on IP, the user ID, or other properties?
- **Interviewer**: The rate limiter should be flexible enough to support different sets of throttle rules.
- **Candidate**: What is the scale of the system? Is it built for a startup or a big company with a large user base?
- **Interviewer**: The system must be able to handle a large number of requests.
- **Candidate**: Will the system work in a distributed environment?
- **Interviewer**: Yes.
- **Candidate**: Is the rate limiter a separate service or should it be implemented in application code?
- **Interviewer**: It is a design decision up to you.
- **Candidate**: Do we need to inform users who are throttled?
- **Interviewer**: Yes.

# Requirements

Here is a summary of the requirements for the system: <br>

- Accurately limit excessive requests.
- Low latency. The rate limiter should not slow down HTTP response time.
- Use as little memory as possible.
- Distributed rate limiting. The rate limiter can be shared across multiple servers or
  processes.
- Exception handling. Show clear exceptions to users when their requests are throttled.
- High fault tolerance. If there are any problems with the rate limiter (for example, a cache server goes offline), it does not affect the entire system.

# Step 2 - Propose high-level design and get buy-in

## Where to put the rate limiter?

- Client side (no recommend)
- Server side (can be put in middleware, vv)

### Guideline

- While designing a rate limiter, an important question to ask ourselves is: where should the rater limiter be implemented, on the server-side or in a gateway? There is no absolute answer.It depends on your company’s current technology stack, engineering resources, priorities,goals, etc. Here are a few general guidelines:
  - Evaluate your current technology stack, such as programming language, cache service,
    etc. Make sure your current programming language is efficient to implement rate limiting
    on the server-side.
  - Identify the rate limiting algorithm that fits your business needs. When you implement
    everything on the server-side, you have full control of the algorithm. However, your
    choice might be limited if you use a third-party gateway.
  - If you have already used microservice architecture and included an API gateway in the
    design to perform authentication, IP whitelisting, etc., you may add a rate limiter to the API gateway.
  - Building your own rate limiting service takes time. If you do not have enough
    engineerin

## Algorithms for rate limiting

### Token bucket algorithm

- A token bucket is a container that has pre-defined capacity. Tokens are put in the bucket
  at preset rates periodically. Once the bucket is full, no more tokens are added. The re-filler puts 2 tokens into the bucket every second. Once the bucket is full, extra tokens will overflow.
- Each request consumes one token. When a request arrives, we check if there are enough
  tokens in the bucket. Figure 4-5 explains how it works.
  - If there are enough tokens, we take one token out for each request, and the request
    goes through.
  - If there are not enough tokens, the request is dropped.
    ![](./images/2025-02-01_15-39.png)
- The token bucket algorithm takes two parameters:
  - **Bucket size**: the maximum number of tokens allowed in the bucket
  - **Refill rate**: number of tokens put into the bucket every second

#### How It Works:

- Imagine a bucket that holds tokens.

- The bucket has a maximum capacity of tokens.

- Tokens are added to the bucket at a fixed rate (e.g., 10 tokens per second).

- When a request arrives, it must obtain a token from the bucket to proceed.

- If there are enough tokens, the request is allowed and tokens are removed.

- If there aren't enough tokens, the request is dropped.

#### How many bucket we need

- It is usually necessary to have different buckets for different API endpoints.
  - if a user is allowed to make 1 post per second, add 150 friends per day, and like 5 posts per second, 3 buckets are required for each user.
  - If we need to throttle requests based on IP addresses, each IP address requires a bucket.
  - If the system allows a maximum of 10,000 requests per second, it makes sense to have a global bucket shared by all requests.

#### Pros:

- The algorithm is easy to implement.
- Memory efficient.
- Token bucket allows a burst of traffic for short periods. A request can go through as long as there are tokens left.

#### Cons:

- Two parameters in the algorithm are bucket size and token refill rate.
- Does not handle sudden bursts of requests well; excess requests are immediately dropped.
- Slightly more complex to implement compared to Token Bucket.

### Leaking bucket algorithm

#### How it works

- The leaking bucket algorithm is similar to the token bucket except that requests are processed at a fixed rate. It is usually implemented with a first-in-first-out (FIFO) queue. The algorithm works as follows:
  - When a request arrives, the system checks if the queue is full. If it is not full, the request
    is added to the queue.
  - Otherwise, the request is dropped.
  - Requests are pulled from the queue and processed at regular intervals.
- ![](./images/2025-02-01_16-07.png)
- Leaking bucket algorithm takes the following two parameters:
  - **Bucket size**: it is equal to the queue size. The queue holds the requests to be processed at a fixed rate.
  - **Outflow rate**: it defines how many requests can be processed at a fixed rate, usually in seconds.
- **Shopify, an ecommerce company**, uses leaky buckets for rate-limiting

### Fixed window counter algorithm
