+++
author = "Kevin Norman"
categories = ["redis", "pipelining", "scripts", "ruby", "optimization"]
date = 2019-04-16T19:59:23Z
description = "Without Redis Pipelining, when making many requests to a Redis, we are at the mercy of the latency between ourselves and the Redis server in question, or we're forced to use threading. To avoid this, we can pipeline our requests."
draft = false
image = "/images/redis-pipelining-diagram.PNG"
slug = "redis-pipelining"
tags = ["Redis", "Stories"]
title = "Beating Round-Trip Latency With Redis Pipelining"

+++

I worked on a task which involved checking every single value in a Redis instance, and modifying ones which matched a certain format. This script needed to run in a fairly timely fashion, as it would need to be run frequently. Making requests sequentially didn't cut it, so I learned about pipelining. This simple optimization meant that a script which I originally calculated to take over 50 hours to run ran in under 4 minutes.

The two approaches for doing this in a timely fashion I considered are as follows:

1. Write some script that makes use of pipelining in order to stream requests to the Redis instance without waiting for the response
2. Writing some Lua script which redis can execute itself, an amazing ability that I should learn to take advantage of more often! This approach would likely have been much more efficient, but there were time constraints on this and my knowledge of Lua is nada, so I decided to go with the script approach.

# Life without pipelining

Without pipelining, when making many requests to a Redis, we are at the mercy of the latency between ourselves and the server in question, or are forced to do threading of some sort. Let's look at some code. In this code, we want to get the ttl, or time to live for each key. If you're unsure what a ttl is, the [Redis docs](https://redis.io/commands/ttl) do a good job of explaining it, but the long and short of it is that you can set an expiry on a key that Redis stores. The Redis ttl is the number of seconds before the key is deleted.

```ruby
keys = ['apple', 'orange', 'banana', 'pineapple']
values = []
keys.each { |key|
    values << redis.ttl(key)
}
```

In this code, we are getting the ttl for 4 keys. Every time we call `redis.ttl()`, our code blocks. This is because we need to wait for Redis to respond to our request for the ttl. This is usually fine for small numbers of keys, but when you've got millions, and the Redis you're talking to is not on your local machine, every single key takes _at least_ the network latency between you and the Redis to retrieve.

In the above example, we are retrieving 4 keys ttls. Lets assume that I, in London am talking to a Redis in the US somewhere. Let's assume a minimum network latency of 100ms. This means, to retrieve these 4 keys, it will take _at least_ 400ms. 40 keys will take at least 4000ms, and so on. You can see how this doesn't scale.

# Enter pipelining

Lets look at the code changes required to pipeline our requests.

```ruby
keys = ['apple', 'orange', 'banana', 'pineapple']
values = []
redis.pipelined do
  keys.each{ |key|
    values << redis.ttl(key)
  }
end
```

That's simpler than expected right? What is actually different here? Instead of sequentially doing the requests, we **pipeline** them. This means we send the requests without waiting on the responses. We send as many requests as our machine can, and the responses come in whenever they're ready from the server. This means if we take the same latency figures as above, we can assume the time to retrieve all these ttls is going to be around 100ms! for 40 keys, the latency will probably be a bit higher, but closer to 100ms than 400ms for sure! 400 keys? Much closer to 100ms than 4000ms!

Here's a diagram I stole from Wikipedia that does a great job of illustrating the difference this makes to how the requests are made.

{{< figure src="/images/redis-pipelining-diagram.PNG" >}}

In this library, the `redis.pipelined` block will block until all the responses come back, and the `values` array will be populated with Redis::futures. You can wait on these in a separate thread and do work as soon as results come in, but if you don't care about that and just want to start your work once you've gotten all your responses, you can just let the `redis.pipelined` block continue to block until all the responses have come back, and then call `value()` on each element in the values array. For example:

```ruby
puts keys[0].value # returns the ttl, ie 3600
```

# How does this work, though?

At a fairly high level, what happens is the server buffers these requests we're sending to it in the order that we sent them. This is why we can depend on the order they're returned in. It processes the requests as it normally does and returns them to our client as normal. There are some important caveats which might matter to your use case though:

* Pipelining does not make your requests atomic. If you're making millions of requests for the ttls of all your keys, then we can assume it's going to take longer than a few seconds. This means that if keys you request expire in the time it takes for Redis to process all these requests, you're not going to be able to retrieve them again. If you need atomicity, you're going to need to do a [transaction](https://redis.io/topics/transactions).
* The buffering of requests on the server takes up memory. If your requests each take a non-trivial amount of time to process by Redis, this buffer can grow very large quickly. If you're making millions of queries, you might want to consider batching your requests. For example, you might pipeline 10000 calls to the ttl method, and once that lot succeeds pipeline another 10000 calls. This means your buffer will never exceed 10000 requests, and you've ensured you're not going to OOM your Redis. The size of your batches will decide how many times you'll need to pay the RTT penalty.
* Redis has lots of lovely methods which somewhat negate the need for pipelining. For example, if you're just getting the values of keys, consider using `mget` instead of pipelining.

# Conclusion

Redis pipelining allows us to make many requests without waiting for the responses or worrying about the order of the responses. It offers huge performance benefits if your service that talks to Redis is far away from the Redis instance. Regarding my use of it, even with this optimization, the majority of the time my script executed was spent on the machine running the script making the modifications to the keys. I am certain more performance could be gotten, but this was more than fast enough for my purposes - and was a very significant improvement.
