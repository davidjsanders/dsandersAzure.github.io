---
layout: post
title: Redis for session management
date: 2017-10-12 10:23:00 +0000
categories:
- Microservics
- Pet Project
- Gaming
- Redis
---

**tl&dr : Use Redis for session state across highly scalable microservices apps** 

My current pet project is a microservices based implementation
of the CowBull game - you know, the one where my code generates
random numbers and you get so many guesses and then you're told
if the guess is a bull (it's the right digit in the right place),
a cow (the right digit in the wrong place), or a miss. For me, this is a fun project because it lets me do lots
of things that are focused on my mantra of explore, try, learn,
master.

As the game is based on a microservices model, it's
highly scalable (from 1 to n servers or containers) and 
portable across Google App Engine, Google Container Engine, 
minikube, Docker swarm, or any Python and/or container 
solution. The architecture looks like this:

![game microservice architecture image](/uploads/2017/10/12/myarch.jpg)

The game uses a game object to store session information
about the game in progress:

```
    {
        "guesses_made": int,
        "key": "A UUID",
        "status": ["playing" | "won" | "lost"],
        "mode": {
            "digits": int,
            "digit_type": DigitWord.DIGIT | DigitWord.HEXDIGIT,
            "mode": GameMode(),
            "priority": int,
            "help_text": str,
            "instruction_text": str,
            "guesses_allowed": int
        },
        "ttl": int,
        "answer": [int|str0, int|str1, ..., int|strN]
    }
```

And logic within the app handles all serialization and
reforming of game objects based on a JSON representation
of this object.

Because the game is intended to be highly scalable and
stateless, using a local persistence engine (like 
sqlite) would have forced me to ensure sticky-ness 
and make sure that a game was served by one and only
one instance. I didn't want to do this because I wanted
to achieve statelessness and treat my instances like 
cattle - I care that the game is available but not that
server 123 is.

So I decided I wanted to run a persistence store that 
every instance could access at any time. Within my
app, abstracted calls to the persistence controller
handle all loading and saving:

```
persister = PersistenceEngine(
    host=redis_host, 
    port=redis_port, 
    db=redis_db
)
_persisted_response = persister.load(key=_key)
```

Now for my game, it no longer cares what the persistence
engine is, simply that it's available. I have abstracted
the choice of storage engine away from the game. Any
instance (whether 1 or 1,000) can call the persistence
engine, pass a key (a 4 word UUID), and retrieve the game.

For my app, I actually selected Redis (1) as the storage engine
because of its low latency, persistence, ease of use, 
high availability options, and the ease of running Redis 
in Docker:

```
docker run --name redis --rm -p 6379:6379 redis
```

Then, because my app follows the 12 factor rules (2) 
I can simply tell my instance where to find the persistence
store using env. vars.:

```
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
```

Using env. vars. makes it really easy to swap my local 
Redis container to a cloud service, such as Redis Labs, 
or tell the app how to find my local HA Kubernetes service:

```
REDIS_HOST=myendpoint.us-east-1-2.ec2.cloud.redislabs.com
REDIS_PORT=12345
```

This works really well, especially when running unit and
integration tests, when simply changing the config allows
rapid testing in seconds across multiple storage locations.

#### References
1. Redis Labs (n.d.), *Homepage*. Available at: [https://redislabs.com](https://redislabs.com)
(Accessed: 12 October 2017)
2. Wiggins, A. (2017) *The Twelve Factor App*. Available at: [https://12factor.net/](https://12factor.net/)
(Accessed: 12 October 2017)

David

david@davidjsanders.com 