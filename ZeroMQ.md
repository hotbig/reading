###CH01 Basics
---
这章最重要的一点是大概描述了ZeroMQ有哪些特性，这些特性是使用ZeroMQ的理由：   
```
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
```
###CH04 Reliable Request-Reply Pattern
---
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

