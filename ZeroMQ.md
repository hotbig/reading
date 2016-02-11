###CH01 Basics
---
这章最重要的一点是大概描述了ZeroMQ有哪些特性，这些特性是使用ZeroMQ的理由：   

And this is ØMQ: an efficient, embeddable library that solves most of the problems an 
application needs to become nicely elastic across a network, without much cost.
Specifically:  
* It handles I/O asynchronously, in background threads. These communicate with application
threads using lock-free data structures, so concurrent ØMQ applications need no locks, 
semaphores, or other wait states.  
* Components can come and go dynamically, and ØMQ will automatically reconnect. This means
you can start components in any order. You can create “service-oriented architectures” 
(SOAs) where services can join and leave the network at any time.  
* It queues messages automatically when needed. It does this intelligently, pushing
messages as close as possible to the receiver before queuing them.  
* It has ways of dealing with over-full queues (called the “high-water mark”). When
a queue is full, ØMQ automatically blocks senders, or throws away messages, depending
on the kind of messaging you are doing (the so-called “pattern”).  
* It lets your applications talk to each other over arbitrary transports: TCP, multicast,
in-process, inter-process. You don’t need to change your code to use a different
transport.  
* It handles slow/blocked readers safely, using different strategies that depend on the
messaging pattern.  
* It lets you route messages using a variety of patterns, such as **request-reply** and
**publish-subscribe**. These patterns are how you create the topology, the structure of
your network.  
* It lets you create **proxies** to queue, forward, or capture messages with a single call.
Proxies can reduce the interconnection complexity of a network.
* It delivers whole messages exactly as they were sent, using a simple framing on the
wire. If you write a 10KB message, you will receive a 10KB message.
* It does not impose any format on messages. They are blobs of zero bytes to gigabytes
large. When you want to represent data you choose some other product on top,
such as Google’s protocol buffers, XDR, and others.
* It handles network errors intelligently. Sometimes it retries, sometimes it tells you
an operation failed.
* It reduces your carbon footprint. **Doing more with less CPU means** your boxes use
less power, and you can keep your old boxes in use for longer. Al Gore would love
ØMQ.
Actually, ØMQ does rather more than this. It has a subversive effect on how you develop
network-capable applications. Superficially, it’s a socket-inspired API on which you do
zmq_msg_recv() and zmq_msg_send().  


###CH02 Sockets and Patterns
---
这章的内容也是ZeroMq最核心的内容。讲了ZMQ Socket API和展现核心设计思想Message Pattern。  
Like a favorite dish, ØMQ sockets are easy to digest. These sockets have a life in four
parts, just like BSD sockets:  
* We can create and destroy them, which go together to form a karmic circle of socket
life (see zmq_socket(), zmq_close()).  
* We can configure them by setting options on them and checking them if necessary
(see zmq_setsockopt(), zmq_getsockopt()).  
* We can plug them into the network topology by creating ØMQ connections to and
from them (see zmq_bind(), zmq_connect()).  
* We can use them to carry data by writing and receiving messages on them (see
zmq_msg_send(), zmq_msg_recv()).  

下面这几个是ZeroMQ最核心的几个Message Pattern：  
* Request-reply, which connects a set of clients to a set of services. This is a remote
procedure call and task distribution pattern.  
* Publish-subscribe, which connects a set of publishers to a set of subscribers. This is
a data distribution pattern.  
* Pipeline, which connects nodes in a fan-out/fan-in pattern that can have multiple
steps and loops. This is a parallel task distribution and collection pattern.  

以及另外的一些组合：
These are the socket combinations that are valid for a connect-bind pair (either side can bind):  
* PUB and SUB  
* REQ and REP  
* REQ and ROUTER  
* DEALER and REP  
* DEALER and ROUTER  
* DEALER and DEALER  
* ROUTER and ROUTER  
* PUSH and PULL  
* PAIR and PAIR  

###CH03 Advanced Request-Reply Pattern
---
这一章继续讲REQ-REP模式的高级topic，除了如何封装数据帧，如果组合这些不同额pattern，最重要的是几个example。
* How the request-reply mechanisms work
* How to combine REQ, REP, DEALER, and ROUTER sockets
* How ROUTER sockets work, in detail
* The load-balancing pattern
* Building a simple load-balancing message broker
* Designing a high-level API for ØMQ
* Building an asynchronous request-reply server
* A detailed inter-broker routing example，**如何从一个cluster扩展到多个cluster（cloud），非常好的一个例子**

###CH04 Reliable Request-Reply Pattern
---
一些介绍到的reliable pattern:  
* The Lazy Pirate pattern: reliable request-reply from the client side  
* The Simple Pirate pattern: reliable request-reply using load balancing  
* The Paranoid Pirate pattern: reliable request-reply with heartbeating  
* The Majordomo pattern: service-oriented reliable queuing，还讲到了如何建立contract。  
* The Titanic pattern: disk-based/disconnected reliable queuing，不错的一个pattern。
* The Binary Star pattern: primary backup server failover  
* The Freelance pattern: brokerless reliable request-reply  

Service-Oriented Reliable Queuing (Majordomo Pattern)部分很好的用C来写面向对象的例子。  
比如对于一个MD的client来说，他需要有：  
* 一个zmq的context属性  
* 一个与之连接的broker属性  
* 一个zmq socket属性  
* 用于的verbose属性  
* 和保证可靠性的timeout/和retry属性  

所以对于MQ Client对象的class类型可以定义成为下面这样的格式：  
```
  struct _mdcli_t {
    zctx_t *ctx; // Our context
    char *broker;
    void *client; // Socket to broker
    int verbose; // Print activity to stdout
    int timeout; // Request timeout
    int retries; // Request retries
  };
```
接着定义这个class对应构造和析构方法用来construct、destruct对象，以及用来接收消息和发送消息的send方法等  
```
  void s_mdcli_connect_to_broker (mdcli_t *self)  
  mdcli_t *mdcli_new (char *broker, int verbose)  
  void mdcli_destroy (mdcli_t **self_p)  
  void mdcli_set_timeout (mdcli_t *self, int timeout)  
  void mdcli_set_retries (mdcli_t *self, int retries)  
  zmsg_t *mdcli_send (mdcli_t *self, char *service, zmsg_t **request_p)
```
**从第四章的开始，作者从面向过程的例子写到此处进化成面向对象的例子，很妙！**

###CH05 Adanced Publish-Subscrible Patterns
---
没有细看

###CH06 The ZeroMQ Community
---
这一章内容其实不错。其中有个关于innovation的例子特别生动：
```
Two old engineers were talking of their lives and boasting of their greatest projects. One
of the engineers explained how he had designed one of the greatest bridges ever made.
“We built it across a river gorge,” he told his friend. “It was wide and deep. We spent two
years studying the land and choosing designs and materials. We hired the best engineers
and spent another five years designing the bridge. We contracted the largest engineering
firms to build the structures, the towers, the tollbooths, and the roads that would connect
the bridge to the main highways. Dozens died during the construction. Under the road
level we had trains, and a special path for cyclists. That bridge represented years of my
life.”  
The second man reflected for a while, then spoke. “One evening me and a friend got
drunk on vodka, and we threw a rope across a gorge,” he said. “Just a rope, tied to two
trees. There were two villages, one at each side. At first, people pulled packages across
that rope with a pulley and string. Then someone threw a second rope, and built a foot
walk. It was dangerous, but the kids loved it. A group of men then rebuilt that, made it
solid, and women started to cross, every day, with their produce. A market grew up on
one side of the bridge, and slowly that became a large town, since there was a lot of space
for houses. The rope bridge got replaced with a wooden bridge, to allow horses and
carts to cross. Then the town built a real stone bridge, with metal beams. Later, they
replaced the stone part with steel, and today there’s a suspension bridge standing in that
same spot.”  
The first engineer was silent. “Funny thing,” he said, “my bridge was demolished about
10 years after we built it. Turns out it was built in the wrong place and no one wanted
to use it. Some guys had thrown a rope across the gorge, a few miles further downstream,
and that’s where everyone went.”  
```
###CH07 Advanced Architecture Using ØMQ
---
FileMQ，下面是如果完成这个application的五个步骤，这就是软件的演进最终形成架构！  
We can turn this into five real steps:  
1. Internalize the ØMQ semantics.  
2. Draw a rough architecture.  
3. Decide on the contracts.  
4. Make a minimal end-to-end solution.  
5. Solve one problem and repeat.  
