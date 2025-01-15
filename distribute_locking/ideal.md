# Ideal

## Mutual Exclusion

- Only one node can hold the lock at a time

## Fault Tolerance

- Lock should not available when not fails

## Performance

- Acquiring and releasing lock should be efficient

## Fairness

- Nodes should have a fair chance of acquiring the lock

<br>

## Centralized Locking
![image](./images/image.png)

- Simple to implement
- Single point of failure

## Token-Base Locking
![image](./images/Screenshot_1.png)

- Fault-tolerant
- Complex to implement

## Quorum-Base locking <RedLock>

![image](./images/Screenshot_2.png)
![image](./images/Screenshot_3.png)

- High fault tolerance and availability
- More complex, potential for issue

## Leader Electric

![image](./images/Screenshot_4.png)

- Avoid single point of failure
- Leader Electric can take time and resources
