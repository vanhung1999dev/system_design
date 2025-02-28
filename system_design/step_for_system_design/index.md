# Step for system design

1. Clarify Requirement
2. Back of the envelope Estimation
3. APIs Design
4. Data Model Design
5. High Level Design
6. Detailed Design
7. Identify & Discuss Bottlenecks

# Interview Time Allocation

45 minutes including introduction (Q&A -> Only 40 minute for real interview) <br>

- Clarify Requirement: 5m
- Estimate: 5m
- API Design: 5m
- DB model Design: 5m
- High level design + detail design + Bottleneck discussion: 20m

## Clarify Requirement (5m)

- focus on core function

  ![](./images/Screenshot_11.png)

## Estimation (5m )

- `no need to calculate correctness`
- `prefer use 10 to easy calculate`
- `example 1 day => 24 \* 3600 , but we will convert it to 10000 to easy calculate`
- `100 M DAU, 10 post message / Day -> 10^9 write per day -> 10^9 / 10^5  ~ 10^4 QPS`
- `10^6 DAU, 2 transaction -> 10^6 * 2 / 10^5 = 2 * 10^2 = 200 TPS `
- `Peak QPS => x2, x3, x4 QPS`

![](./images/Screenshot_11.png)

<br>

![](./images/Screenshot_2.png)

<br>

### Example

![](./images/Screenshot_3.png)

## Api Design (5m)

![](./images/Screenshot_4.png)

### Design API

![](./images/Screenshot_5.png)

## Data Model Design (5m)

![](./images/Screenshot_6.png)

## High Level Design

![](./images/Screenshot_7.png)

## Detail Design

![](./images/Screenshot_8.png)

## Identify & Discuss Bottleneck

![](./images/Screenshot_9.png)
