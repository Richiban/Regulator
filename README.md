# Richiban.Regulator

Richiban.Regulator is a library that you can use to rate limit outbound requests (or, in fact, any async operation).

Let's say you're calling a third party API and it's very important that you don't exceed their rate limits, use this library and configure it with the same parameters the third party has given and your code will automatically obey their rate limits.

Currently supported limits:

- `{count}` per `{timeframe}`. Sliding window.
  - e.g. 500 per minute
- maximum `{count}` concurrent requests at any time
  - e.g. at most 10 concurrent requests

## Usage

```csharp
async Task MakeRequest() { ... }

var rateLimiter = new RateLimiter(
    new SlidingWindowRateLimit(
        count: 500,
        timePeriod: TimeSpan.FromSeconds(60)),
    new ConcurrencyRateLimit(maxConcurrentRequests: 2))
{
    WaitTimeout = TimeSpan.FromSeconds(30)
};

await rateLimiter.WhenReady(() => MakeRequest());
```

**Note** It's up to you to manage the lifetimes of these rate limiter objects; there are no statics built in so you will need to share the relevant instances to ensure that rate limits get obeyed.

## Token pooling

Some APIs will apply their rate limits per authentication token, therefore adding more tokens to use will increase the number of requests you can make.

To make use of this feature instantiate a `RateLimiterPool`.
