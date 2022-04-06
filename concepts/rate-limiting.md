
In this article, I will introduce the concept of rate limiting,
in addition to 4 ways to implement it.

What is rate limiting in the context of APIs?

Rate limiting is decreasing the threshold of allowed traffic to APIs/services.

When rate limiting an API, some kind of API gateway could be involved to handle the transmission of requests to their destination.

For example, an API (A) may be configured to only accept 100 requests per second, while another (B) can accept 500 requests per second.

In this example, the architect may want to adjust the rate limit of (B) to the limit of (A), because the inconsistency of scalibility can be detrimental in the future to the system.

Currently (04/23/2022), rate limiting is implementing in one of 4 ways.

    1. Token bucket
    2. Leaky bucket
    3. Fixed window
    4. Sliding window

Each method can be applied at the network level; however, this post will explain at the highest-level and not mention "bytes" and "packets".

### Token bucket

The token bucket implementation involves tokens associated with a client and applies logic on each request, using the number of tokens in the bucket.

Here is the general conceptual algorithm for a token bucket implementation:

    1. 


