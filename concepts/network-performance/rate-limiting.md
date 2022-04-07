
In this article, I will introduce the concept of rate limiting, in addition to 4 ways to implement it.

What is rate limiting in the context of APIs?

Rate limiting is decreasing the threshold of allowed traffic to APIs/services.

When rate limiting an API, some kind of API gateway could be involved to handle the transmission of requests to their destination.

For example, an API (A) may be configured to only accept 100 requests per second, while another (B) can accept 500 requests per second.

In this example, the architect may want to adjust the rate limit of (B) to the limit of (A), because the inconsistency of scalibility can be detrimental in the future to the system.

Currently (04/02/2022), rate limiting is implemented in one of 4 ways.

    1. Token bucket
    2. Leaky bucket
    3. Fixed window
    4. Sliding window

Each method can be applied at the network level; however, this post will explain at the highest-level and not mention "bytes" and "packets".

### Token bucket

The token bucket implementation involves tokens associated with a client and applies logic on each request, using the number of tokens in the bucket.

Here is the general conceptual algorithm for a token bucket implementation:

    1. A token is added
    2. The bucket has a limit on how much tokens can be stored
    3. The bucket has a limit on how much tokens can be forwarded every second (burstiness)
    4. If the bucket is full, the request is not forwarded
    5. If the bucket is not full, the request is forwarded, and a token is removed from the bucket

### Leaky bucket

The leaky bucket implementation uses a fixed capacity bucket which "leaks" its contents over time.

Here is the general conceptual algorithm for a leaky bucket implementation:

    1. A fixed capcacity bucket leaks at a fixed rate
    2. If the bucket is empty, it stops leaking
    3. If a specific amount of "water" can be added to the bucket, the request is forwarded
    4. If a specific amount of "water" cannot be added to the bucket, the request is not forwarded

A leaky bucket can be thought of and implemented as a FIFO queue. When a request arrives, if there is room on the queue it is appended to the queue, otherwise it is discarded.

![Leaky bucket](/assets/images/leaky-bucket-as-a-queue.png "Leaky bucket as a FIFO queue")

### Fixed window

The fixed window implementation divides the time into fixed windows and places the request in the windows, and makes the clients requests at a later, specified time interval if the window is full.

An analogy:

Consider a train moving with people on it, and assume people can board even if it is moving. If the train is full, people wanting to get on will have to try to board again, but only after one minute.

Here is a code implementation:

```python
import time

class FixedWindow:

    def __init__(self, max_capacity):
        self.space_left = max_capacity
        self.max_capacity = max_capacity
        self.timestamp = time.time()

    def reject_request(self, request):
        return None

    def accept_request(self, request):
        return request

    def handle_request(self, request):
        current_timestamp = time.time()

        if self.timestamp != current_timestamp:
            self.timestamp = current_timestamp
            self.space_left = self.max_capacity

        if self.space_left < 1:
            return self.reject_request(request)
        
        self.space_left -= 1
        return self.accept_request(request)

```

Fixed window has issues with bursts of traffic, due to how waiting traffic could all attempt to fill a window around the same time.

### Sliding window

The sliding window implementation is based on the fixed window implementation but, it does not limit requests based on a fixed time interval/unit.

Sliding window is able to handle burstiness better than fixed window.

It smooths the flow of traffic by taking into account the previous number of requests in the last window when calculating capacity.

Here is a code implementation:

```python
import time

class SlidingWindow:

    def __init__(self, max_capacity, time_unit):
        self.space_filled = 0
        self.previous_space_filled = max_capacity
        self.max_capacity = max_capacity
        self.time_unit = time_unit
        self.timestamp = time.time()

    def reject_request(self, request):
        return None

    def accept_request(self, request):
        return request

    def handle_request(self, request):
        current_timestamp = time.time()

        if current_timestamp - self.timestamp > self.time_unit:
            self.timestamp = time.time()
            self.previous_space_filled = self.space_filled
            self.space_filled = 0

        estimated_space_taken = (self.previous_space_filled * (self.time_unit - (current_timestamp - self.timestamp) / self.time_unit)) + self.space_filled

        if estimated_space_taken > self.max_capacity:
            return self.reject_request(request)
        
        self.space_filled += 1

        return self.accept_request(request)

sliding_window = SlidingWindow(capacity=100, time_unit=1)

```





