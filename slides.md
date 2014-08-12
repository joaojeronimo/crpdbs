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



### First prototypes:

## MongoDB


### Wow this is cool


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


## The Redis Phase