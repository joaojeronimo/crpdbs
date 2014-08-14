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


## The workload is mainly RW 1/2

meaning we do 2 writes for every 1 read


## The typical data path:

### user sends input data

* **write** input to the DB
* send input to worker
* get result from worker
* **write** result to the DB

### user retrieves the results

* **read** result from the DB
* send it to user


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


### oops, Indexes > RAM


### Default Write Concern Acknowledged ?


## No fun with less than 8 servers

![](http://docs.mongodb.org/manual/_images/sharded-cluster-production-architecture.png)


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

* No decent drivers (then pgte made the best one for node)
* Update = read, modify and write
* No query language at all


### The worst

Running Riak on 3 c1.xlarge EC2 instances was very slow

###### not using EBS of course, if we did we would be dead


so we had a **cache layer** in front of Riak

###### about 6 cache servers


using


redis!



## Our infrastructure was complicated


we had 50 private repositories on github


about 7 different services running

###### not counting metrics and telemetry services


the platform crashed often

###### too many things to manage, small team


I have a theory on this


## p < 2 * s

###### p: number of people in your team
###### s: number of services you're running

you're in trouble



## So then came the revolution


### ditched AWS for Hetzner

* No virtualization
* Latest waswell 4 core CPU, 32GB of fast RAM
* Most important, 2*240GB of very fast SSD


databases <3 SSDs


### Also ditched Riak for Cassandra


Cassandra was about 2x faster for our use case, on both reads and writes


didn't need a cache layer anymore!


Went back to two monoliths, api-server and browser-server

![](http://michaeltougher.com/wp-content/uploads/2014/02/2001space037.jpg)

Massive simplification of the infrastructure


## huge rewrite

also, node.js -> Go


So now CrowdProcess was simple, easy to manage and more performant


The DB situation:

* 3 node cluster
* about 138k writes per second
* about 114k reads per second
* relational data is on PostgreSQL
* overly written and accessed data is on Cassandra
* rarely fails



# Let's compare


* write an object
* retrieve object by it's key
* retrieve an object by an attribute


## MongoDB

#### insert
    db.foo.update({
      _id: 'one',
      attr: 2
    });

#### get by ID
    db.find({_id: 'one'})

#### get by attribute
    db.find({attr: 2})


## Redis

#### insert
    HSET one attr 2
    HSET attrIndex 2 one # index it

### get by ID
    HGETALL one # attr: 2

### get by attribute
    HGET attrIndex 2 # => one
    HGETALL one # => attr: 2


## Riak

    var modelOptions = {
      bucket: 'numbers',
      indexes: [
        { key: 'attr' }
      ]
    };

    var numbers = RiakModel(modelOptions);

### create

    numbes.save({_id: 'one', attr: 2}, callback);

### get by ID

    numbers.get('one', callback)

### get by attribute

    numbers.findAllByAttr(2, cb)


## Cassandra

### create
    CREATE KEYSPACE everything;
    CREATE TABLE everything.numbers (id VARCHAR, attr INT, PRIMARY KEY (id));
    CREATE INDEX ON everything.numbers (attr); 

    INSERT INTO everything.numbers (id, attr) VALUES ("one", 2);

### get by ID

    SELECT * FROM everything.numbers WHERE id = "one";

### get by attribute

    SELECT * FROM everything.numbers WHERE attr = 2; // fine under 1000 rows



## My newest toy

# n1ql.io


hosted CouchBase with N1QL queries


## API

```bash
> curl -X POST https://n1ql.io/one -d '{"attr": 2}'
> curl -X GET https://n1ql.io/one
{
  "attr": 2
}
```

### And of course, N1QL

```bash
> curl https://n1ql.io/?query='SELECT * FROM free WHERE attr = 2;'
{
  "attr": 2
}
```


## but it's not ready
http://www.couchbase.com/communities/q-and-a/how-query-curl



## my advice on choosing DBs for startups


# Listen to your data


if your project is a website or something **not IO intensive**, use a free 10.000 row PostgreSQL table from Heroku


## if it's relational

stick with **MySQL** or **PostgreSQL**

They're fast, proven and have SQL


## if it's key-value data

**redis** until you can scale up

**cassandra** after

skip Riak and use Cassandra as a key-value store


## for objects

CouchDB

MongoDB will take too much of your time when you have to scale


## Metrics, Lucene Querying

ElasticSearch



# That's all folks!