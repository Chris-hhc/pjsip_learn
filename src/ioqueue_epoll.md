## ioqueue_epoll

### pj_ioqueue_t

ioqueue的整体结构pj_ioqueue_t，使用epoll的在ioqueue_epoll.c  下列出ioqueue相关的`DECLARE_COMMON_IOQUEUE宏`、`pj_ioqueue_cfg`、`pj_ioqueue_t`

```c
#define DECLARE_COMMON_IOQUEUE                      \
    pj_lock_t          *lock;                       \
    pj_bool_t           auto_delete_lock;           \
    pj_ioqueue_cfg      cfg;

/**
 * Additional settings that can be given during ioqueue creation. Application
 * MUST initialize this structure with #pj_ioqueue_cfg_default().
 */
typedef struct pj_ioqueue_cfg
{
    /**
     * Specify flags to control e.g. how events are handled when epoll backend
     * is used on Linux. The values are combination of pj_ioqueue_epoll_flag.
     * The default value is PJ_IOQUEUE_DEFAULT_EPOLL_FLAGS, which by default
     * is set to PJ_IOQUEUE_EPOLL_AUTO. This setting will be ignored for other
     * ioqueue backends.
     */
    unsigned  epoll_flags;

    /**
     * Default concurrency for the handles registered to this ioqueue. Setting
     * this to non-zero enables a handle to process more than one operations
     * at the same time using different threads. Default is
     * PJ_IOQUEUE_DEFAULT_ALLOW_CONCURRENCY. This setting is equivalent to
     * calling pj_ioqueue_set_default_concurrency() after creating the ioqueue.
     */
    pj_bool_t default_concurrency;

} pj_ioqueue_cfg;

/*
 * This describes the I/O queue.
 */
struct pj_ioqueue_t
{
    DECLARE_COMMON_IOQUEUE

    unsigned            max, count;
    //pj_ioqueue_key_t  hlist;
    pj_ioqueue_key_t    active_list;    
    int                 epfd;
    //struct epoll_event *events;
    //struct queue       *queue;

#if PJ_IOQUEUE_HAS_SAFE_UNREG
    pj_mutex_t         *ref_cnt_mutex;
    pj_ioqueue_key_t    closing_list;
    pj_ioqueue_key_t    free_list;
#endif
};
```

有三个 pj_ioqueue_key_t类型的队列active_list、closing_list、free_list，

### pj_ioqueue_key_t

```c
#define DECLARE_COMMON_KEY                          \
    PJ_DECL_LIST_MEMBER(struct pj_ioqueue_key_t);   \
    pj_ioqueue_t           *ioqueue;                \
    pj_grp_lock_t          *grp_lock;               \
    pj_lock_t              *lock;                   \
    pj_bool_t               inside_callback;        \
    pj_bool_t               destroy_requested;      \
    pj_bool_t               allow_concurrent;       \
    pj_sock_t               fd;                     \
    int                     fd_type;                \
    void                   *user_data;              \
    pj_ioqueue_callback     cb;                     \
    int                     connecting;             \
    struct read_operation   read_list;              \
    struct write_operation  write_list;             \
    struct accept_operation accept_list;            \
    UNREG_FIELDS
    
/*
 * This describes each key.
 */
struct pj_ioqueue_key_t
{
    DECLARE_COMMON_KEY
    struct epoll_event ev;
};

```

pj_ioqueue_key_t中出现了epoll_event 是linux中结构

### epoll_event

```c
typedef [union](https://so.csdn.net/so/search?q=union&spm=1001.2101.3001.7020) epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;//保存触发事件的某个文件描述符相关的数据

struct epoll_event {
    __uint32_t events;   /* [epoll](https://so.csdn.net/so/search?q=epoll&spm=1001.2101.3001.7020) event */
    epoll_data_t data;   /* User data variable */
};
```

### pj_ioqueue_callback

接下来介绍I/O结束的回调函数

```c
/**
 * This structure describes the callbacks to be called when I/O operation
 * completes.
 */
typedef struct pj_ioqueue_callback
{
    /**
     * This callback is called when #pj_ioqueue_recv or #pj_ioqueue_recvfrom
     * completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param bytes_read    >= 0 to indicate the amount of data read,
     *                      otherwise negative value containing the error
     *                      code. To obtain the pj_status_t error code, use
     *                      (pj_status_t code = -bytes_read).
     */
    void (*on_read_complete)(pj_ioqueue_key_t *key,
                             pj_ioqueue_op_key_t *op_key,
                             pj_ssize_t bytes_read);

    /**
     * This callback is called when #pj_ioqueue_send or #pj_ioqueue_sendto
     * completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param bytes_sent    >= 0 to indicate the amount of data written,
     *                      otherwise negative value containing the error
     *                      code. To obtain the pj_status_t error code, use
     *                      (pj_status_t code = -bytes_sent).
     */
    void (*on_write_complete)(pj_ioqueue_key_t *key,
                              pj_ioqueue_op_key_t *op_key,
                              pj_ssize_t bytes_sent);

    /**
     * This callback is called when #pj_ioqueue_accept completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param sock          Newly connected socket.
     * @param status        Zero if the operation completes successfully.
     */
    void (*on_accept_complete)(pj_ioqueue_key_t *key,
                               pj_ioqueue_op_key_t *op_key,
                               pj_sock_t sock,
                               pj_status_t status);

    /**
     * This callback is called when #pj_ioqueue_connect completes.
     *
     * @param key           The key.
     * @param status        PJ_SUCCESS if the operation completes successfully.
     */
    void (*on_connect_complete)(pj_ioqueue_key_t *key,
                                pj_status_t status);
} pj_ioqueue_callback;
```

### pj_ioqueue_op_key_t

```c
typedef struct pj_ioqueue_op_key_t
{
    void *internal__[32];           /**< Internal I/O Queue data.   */
    void *activesock_data;          /**< Active socket data.        */
    void *user_data;                /**< Application data.          */
} pj_ioqueue_op_key_t;
```

## 初始化相关

### pj_ioqueue_create2

pj_ioqueue_create（空实现调用pj_ioqueue_create2）

```c
status = pj_ioqueue_create( endpt->pool, PJSIP_MAX_TRANSPORTS, &endpt->ioqueue);
if (status != PJ_SUCCESS) {
    goto on_error;
}
```

初始化ioqueue的空间、ioqueue->lock、ioqueue->auto_delete_lock、ioqueue->cfg（pj_ioqueue_cfg）、ioqueue->max、ioqueue->count、ioqueue->cfg.epoll_flags（epoll type）、ioqueue->ref_cnt_mutex、ioqueue->free_list（对max_fd个key初始化key->ref_count、key->lock然后加入freelist）、ioqueue->closing_list、ioqueue->epfd（epoll fd调用epoll create）

### ioqueue_init_key

```c
static pj_status_t ioqueue_init_key( pj_pool_t *pool,
                                     pj_ioqueue_t *ioqueue,
                                     pj_ioqueue_key_t *key,
                                     pj_sock_t sock,
                                     pj_grp_lock_t *grp_lock,
                                     void *user_data,
                                     const pj_ioqueue_callback *cb)
```

初始化key的key->ioqueue、key->fd、key->user_data（这里放到是transport_udp）、key->read_list、key->write_list、key->accept_list、key->connecting = 0、key->cb（callback）、key->closing 、key->allow_concurrent（pj_ioqueue_set_concurrency）、key->fd_type、key->grp_lock

### pj_ioqueue_register_sock2

pjmedia_transport_udp_create3->pjmedia_transport_udp_attach->pj_ioqueue_register_sock2

os_ioctl 设置socket to nonblocking.

从ioqueue的freelist中取得key，ioqueue_init_key，然后初始化key->ev（epoll_event类型 data.ptr 是key本身）, os_epoll_ctl注册 fd-events监听其socket事件。

```c
PJ_DEF(pj_status_t) pj_ioqueue_register_sock2(pj_pool_t *pool,
                                              pj_ioqueue_t *ioqueue,
                                              pj_sock_t sock,
                                              pj_grp_lock_t *grp_lock,
                                              void *user_data,
                                              const pj_ioqueue_callback *cb,
                                              pj_ioqueue_key_t **p_key)
  
    status = pj_ioqueue_register_sock2(pool, ioqueue, tp->rtp_sock, grp_lock,
                                       tp, &rtp_cb, &tp->rtp_key);
```

注意在`pjmedia_transport_udp_attach` 中调用pj_ioqueue_register_sock2传入的是tp->rtp_key，pj_ioqueue_register_sock2返回后，会将绑定socket后的key保存在rtp_key中

## ioqueue 读取写入流程

ioqueue 读取全流程

pj_ioqueue_poll，发现事件-》ioqueue_dispatch_read_event-》on_rx_request-》on_rx_request

### pj_ioqueue_poll

事件触发后执行主要使用os_epoll_wait，执行分发回调

一般会另开一个工作线程，不停循环，执行epoll_wait以监听读取/写入rtp包的事件请求

```c
while(1){
		pj_ioqueue_poll()
}
```

下面详细介绍pj_ioqueue_poll

先调用os_epoll_wait，事件放到events中，遍历所有events。根据read,write事件放入到queue中，queue结构如下：

```c
struct queue
{
    pj_ioqueue_key_t        *key;
    enum ioqueue_event_type  event_type;
};

enum ioqueue_event_type
{
    NO_EVENT,
    READABLE_EVENT  = 1,
    WRITEABLE_EVENT = 2,
    EXCEPTION_EVENT = 4,
};
```

最后在遍历queue，根据事件执行相应的处理函数

READABLE_EVENT：ioqueue_dispatch_read_event

WRITEABLE_EVENT：ioqueue_dispatch_write_event

EXCEPTION_EVENT：ioqueue_dispatch_exception_e

### ioqueue_dispatch_read_event 读取操作

首先要看一下read_operation

```c
struct read_operation
{
    PJ_DECL_LIST_MEMBER(struct read_operation);
    pj_ioqueue_operation_e  op;

    void                   *buf;
    pj_size_t               size;
    unsigned                flags;
    pj_sockaddr_t          *rmt_addr;
    int                    *rmt_addrlen;
};
```

```c

/**
 * Types of pending I/O Queue operation. This enumeration is only used
 * internally within the ioqueue.
 */
typedef enum pj_ioqueue_operation_e
{
    PJ_IOQUEUE_OP_NONE          = 0,    /**< No operation.          */
    PJ_IOQUEUE_OP_READ          = 1,    /**< read() operation.      */
    PJ_IOQUEUE_OP_RECV          = 2,    /**< recv() operation.      */
    PJ_IOQUEUE_OP_RECV_FROM     = 4,    /**< recvfrom() operation.  */
    PJ_IOQUEUE_OP_WRITE         = 8,    /**< write() operation.     */
    PJ_IOQUEUE_OP_SEND          = 16,   /**< send() operation.      */
    PJ_IOQUEUE_OP_SEND_TO       = 32,   /**< sendto() operation.    */
#if defined(PJ_HAS_TCP) && PJ_HAS_TCP != 0
    PJ_IOQUEUE_OP_ACCEPT        = 64,   /**< accept() operation.    */
    PJ_IOQUEUE_OP_CONNECT       = 128   /**< connect() operation.   */
#endif  /* PJ_HAS_TCP */
} pj_ioqueue_operation_e;
```



众多read_operation会挂在key中`struct read_operation   read_list;` 上通过`key_has_pending_read` 判断是否有pending的read操作

具体流程：

先看key read_list上有没有pending_read,有的话，从read_list取出，根据read_list的read_op确定读入大小，pj_sock_recvfrom接受数据的函数，将数据读入到read_op中，最后调用on_read_complete，回调函数已在pj_ioqueue_register_sock2时设置过，传入read_op

```c
(*h->cb.on_read_complete)(h, 
                                      (pj_ioqueue_op_key_t*)read_op,
                                      bytes_read);
```



### ioqueue_dispatch_write_event写操作

与ioqueue_dispatch_read_event相似，先看write_operation

```c
struct write_operation
{
    PJ_DECL_LIST_MEMBER(struct write_operation);
    pj_ioqueue_operation_e  op;

    char                   *buf;
    pj_size_t               size;
    pj_ssize_t              written;
    unsigned                flags;
    pj_sockaddr_in          rmt_addr;
    int                     rmt_addrlen;
};
```

众多write_operation会挂在key中`struct write_operation  write_list;` 上通过`key_has_pending_write` 判断是否有pending的write操作

具体流程：

先看key write_list上有没有pending_write,有的话，从write_list取出，根据write_list的 write_op确定写大小，要写入的数据，将数据写入调用pj_sock_send函数Transmit data to the socket.，最后调用on_write_complete，回调函数已在pj_ioqueue_register_sock2时设置过，传入write_op

```c
if (h->cb.on_write_complete && !IS_CLOSING(h)) {
                (*h->cb.on_write_complete)(h, 
                                           (pj_ioqueue_op_key_t*)write_op,
                                           write_op->written);
            }
```

### on_rx_rtp 读取操作

on_rx_rtp是被下面调用

```c
(*h->cb.on_read_complete)(h, 
                                      (pj_ioqueue_op_key_t*)read_op,
                                      bytes_read);
```

这里有一个很有意思的问题，就是read_op 在on_rx_rtp中竟然没有使用，下面来分析一下原因

```
udp = (struct transport_udp*) pj_ioqueue_get_user_data(key);

```

我们的read_op是从readlist中取出的，readlist对于读操作添加是依靠pj_ioqueue_recvfrom函数，在data is not immediately available时将read_op加入readlist

```c
read_op->op = PJ_IOQUEUE_OP_RECV_FROM;
read_op->buf = buffer;
read_op->size = *length;
read_op->flags = flags;
read_op->rmt_addr = addr;
read_op->rmt_addrlen = addrlen;

pj_ioqueue_lock_key(key);
/* Check again. Handle may have been closed after the previous check
 * in multithreaded app. If we add bad handle to the set it will
 * corrupt the ioqueue set. See #913
 */
if (IS_CLOSING(key)) {
    pj_ioqueue_unlock_key(key);
    return PJ_ECANCELLED;
}
pj_list_insert_before(&key->read_list, read_op);
ioqueue_add_to_set(key->ioqueue, key, READABLE_EVENT);
```

这里read_op->buf = buffer;的buffer，来自udp->rtp_pkt,相当于直接写入了rtp_pkt,所以不用read_op了。

on_rx_rtp是一个while循环，条件如下status来自pj_ioqueue_recvfrom的结果

```c
status != PJ_EPENDING && status != PJ_ECANCELLED &&
             udp->started
```

#### call_rtp_cb

在while循环里，先执行call_rtp_cb，设置pjmedia_tp_cb_param param;，调用(*cb2)(&param);cb2由transport_attach2-》tp_attach设置为stream.c ::on_rx_rtp。注意param.pkt = udp->rtp_pkt;，这里rtp_pkt其实就是ioqueue_dispatch_read_event中read_op->buf中读到的数据rtp包

```c
/* Call RTP cb. */
static void call_rtp_cb(struct transport_udp *udp, pj_ssize_t bytes_read, 
                        pj_bool_t *rem_switch)
{
    void (*cb)(void*,void*,pj_ssize_t);
    void (*cb2)(pjmedia_tp_cb_param*);
    void *user_data;

    cb = udp->rtp_cb;
    cb2 = udp->rtp_cb2;
    user_data = udp->user_data;

    if (cb2) {
        pjmedia_tp_cb_param param;

        param.user_data = user_data;
        param.pkt = udp->rtp_pkt;
        param.size = bytes_read;
        param.src_addr = &udp->rtp_src_addr;
        param.rem_switch = PJ_FALSE;
        (*cb2)(&param);
        if (rem_switch)
            *rem_switch = param.rem_switch;
    } else if (cb) {
        (*cb)(user_data, udp->rtp_pkt, bytes_read);
    }
}
```

 param.user_data = user_data;  注意这个user_data，是pjmedia_stream *stream

#### pj_ioqueue_recvfrom

接下来是调用pj_ioqueue_recvfrom，至于为什么明明ioqueue_dispatch_read_event已经读取了数据，此时还在读取数据，是因为可能有新的rtp包到达，pj_ioqueue_recvfrom查看有没有到达的包，如果有就调用pj_sock_recvfrom继续读读到udp->rtp_pkt，如果没有加到readlist中，返回PJ_EPENDING，结束on_rx_rtp中的while循环。

### on_rx_rtp::cb2 

Stream.c中的回调 tp_attach中设置该回调

该函数处理接收到的rtp包, 解析成payload和head

Put "good" packet to jitter buffer，需要先把payload解析成frame，再把frame放入jitter buffer

