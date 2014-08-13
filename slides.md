![CrowdProcess](https://crowdprocess.com/img/crowdprocess-logo-symbol.svg)
# DBs @ CrowdProcess



## CrowdProcess ?

* HTML5 Distributed Computing Platform
* Uses WebSockets to connect to WebWorkers

![how it works](https://crowdprocess.com/img/dev-animation.gif)


## CrowdProcess is IO heavy

* Lots of data goes in;
* We ship it to the Web Workers;
* The workers compute the result;
* We send the result to the user;


## The workload is mainly RW 1/1

* meaning we do as much reads as we do writes


## When it started, I knew nothing

![jon snow](http://www.quickmeme.com/img/2f/2fbfaf0c8356b6ec8fb02a3c0e51a48e7dd44d036c3e82384aac53ea059c1abb.jpg)


## but who cares!

![I know how to do it](http://i.imgur.com/6Mb3oI6.gif)



## Before we start


### "I'll deal with scalability problems when I have scalability problems"


### "I want to focus on my application rather than my infrastructure"



### First prototypes:

## MongoDB


### Wow this is cool

![young again](http://www.troll.me/images/the-most-interesting-man-in-the-world/i-dont-always-use-nosql-but-when-i-do-i-feel-young-again.jpg)


### Damn Indexes > RAM


### no acks ?!


* at least 3 servers for a replica set
* at least 2 shards
* at least 3 config servers

![](http://docs.mongodb.org/manual/_images/sharded-cluster-production-architecture.png)


## No fun with less than 8 servers


## very messy deployment

![messi](http://38.media.tumblr.com/7f1ad54ef1b3e185e46bb47a309ccac4/tumblr_mi95tlc8Vn1s5gv1ro1_500.gif)

###### ba dum tssss
###### I'll show myself out


### sum it up

* pretty cool API clients
* easy to dive into
* gets messy when you want to scale



## The Redis Phase

* key-value is dead-simple
* redis has awesome data structures like Hashes, Lists, Sorted Sets
* handy operations like `ZREMRANGEBYLEX SDIFFSTORE BRPOPLPUSH`
* Pub/Sub is sweet
* Simple APIs

Not your usual Key-Value store


## Far easier to manage

Very lightweight on resources, common to have lots of redis processes running.


## Robust

![no restart](http://cdn.memegenerator.net/instances/500x/43304700.jpg)


## But there was only master-slave replication at the time :(


## Did someone say multi-master?
![master](http://www.quickmeme.com/img/d8/d89721941b0de9ed181cae87b1f647e14f60e7c13ac9e4ca645fef0a4ae08164.jpg)


## ![github](https://cdn4.iconfinder.com/data/icons/ionicons/512/icon-social-github-128.png) joaojeronimo/node_redis_cluster


## Redis Sharding before it was cool

or even possible

actually still not possible


## but... persistence!

* AOF (persistence logs)
* RDB (point in time snapshots)

Choose one, tune it, ship it to S3 every now and then
If writes become slow, get a slave to persist for you


## So yes, redis is a pretty good primary data store


## Just ask [these guys](https://muut.com/blog/technology/redis-as-primary-datastore-wtf.html)

![](https://muut.com/m/img/blog/redis/chart.png)

###### "We launched a couple days ago and had a decent spike in traffic initially. We were servicing some 50 API calls per second and the CPU of our main API server sat completely idle."

###### "can fulfill API requests under load at around 2ms"

###### "8 Days. 8000 forums."


## I <3 Redis


## But you cannot run a company on pre-alpha software.



## Riak

* Distributed key-value store
* Dynamo based (R, C)

### The good

...

### The bad

* No decent drivers (pgte made the best one in JS)
* Update = read, modify and write (not atomic)
* Not very fast on comodity hardware
* No query language at all