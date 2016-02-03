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
  struct _mdcli_t {
  zctx_t *ctx; // Our context
  char *broker;
  void *client; // Socket to broker
  int verbose; // Print activity to stdout
  int timeout; // Request timeout
  int retries; // Request retries
  };
接着定义这个class对应构造和析构方法用来construct、destruct对象，以及用来接收消息和发送消息的send方法等
  void s_mdcli_connect_to_broker (mdcli_t *self)  
  mdcli_t *mdcli_new (char *broker, int verbose)  
  void mdcli_destroy (mdcli_t **self_p)  
  void mdcli_set_timeout (mdcli_t *self, int timeout)  
  void mdcli_set_retries (mdcli_t *self, int retries)  
  zmsg_t *mdcli_send (mdcli_t *self, char *service, zmsg_t **request_p)
很好的面向对象的例子！  

