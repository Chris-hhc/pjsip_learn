## Transport

媒体传输封装了网络收发细节，pjmedia_transport可以是udp、srtp、ice等，这里以udp为例。

### 相关结构体

#### **结构体pjmedia_transport**

```cpp
/**
 * This structure declares media transport. A media transport is called
 * by the stream to transmit a packet, and will notify stream when
 * incoming packet is arrived.
 */
struct pjmedia_transport
{
    /** Transport name (for logging purpose). */
    char		     name[PJ_MAX_OBJ_NAME];
 
    /** Transport type. */
    pjmedia_transport_type   type;
 
    /** Transport's "virtual" function table. */
    pjmedia_transport_op    *op;
 
    /** Application/user data */
    void		    *user_data;
};
```

type：传输类型，上面讲过，这里以udp为例。

op：操作集，每种传输类型实现了同一组接口。

user_data：应用层用户数据

#### pjmedia_transport_op

操作集是核心，这里列举重要的一些函数。[pjmedia_transport_op](https://docs.pjsip.org/en/latest/api/generated/pjmedia/group/group__PJMEDIA__TRANSPORT.html#_CPPv420pjmedia_transport_op)

```cpp
/**
 * This structure describes the operations for the stream transport.
 */
struct pjmedia_transport_op
{
 
    /**
     * This function is called by the stream when the transport is about
     * to be used by the stream for the first time, and it tells the transport
     * about remote RTP address to send the packet and some callbacks to be 
     * called for incoming packets. This function exists for backwards
     * compatibility. Transports should implement attach2 instead.
     *
     * Application should call #pjmedia_transport_attach() instead of 
     * calling this function directly.
     */
    pj_status_t (*attach)(pjmedia_transport *tp,
			  void *user_data,
			  const pj_sockaddr_t *rem_addr,
			  const pj_sockaddr_t *rem_rtcp,
			  unsigned addr_len,
			  void (*rtp_cb)(void *user_data,
					 void *pkt,
					 pj_ssize_t size),
			  void (*rtcp_cb)(void *user_data,
					  void *pkt,
					  pj_ssize_t size));
 
    /**
     * This function is called by the stream to send RTP packet using the 
     * transport.
     *
     * Application should call #pjmedia_transport_send_rtp() instead of 
     * calling this function directly.
     */
    pj_status_t (*send_rtp)(pjmedia_transport *tp,
			    const void *pkt,
			    pj_size_t size);
 
    /**
     * Prepare the transport for a new media session.
     *
     * Application should call #pjmedia_transport_media_create() instead of 
     * calling this function directly.
     */
    pj_status_t (*media_create)(pjmedia_transport *tp,
				pj_pool_t *sdp_pool,
				unsigned options,
				const pjmedia_sdp_session *remote_sdp,
				unsigned media_index);
 
    /**
     * This function is called by application to start the transport
     * based on local and remote SDP.
     *
     * Application should call #pjmedia_transport_media_start() instead of 
     * calling this function directly.
     */
    pj_status_t (*media_start) (pjmedia_transport *tp,
			        pj_pool_t *tmp_pool,
			        const pjmedia_sdp_session *sdp_local,
			        const pjmedia_sdp_session *sdp_remote,
				unsigned media_index);
 
    /**
     * This function is called by application to stop the transport.
     *
     * Application should call #pjmedia_transport_media_stop() instead of 
     * calling this function directly.
     */
    pj_status_t (*media_stop)  (pjmedia_transport *tp);
 
 
    /**
     * This function can be called to destroy this transport.
     *
     * Application should call #pjmedia_transport_close() instead of 
     * calling this function directly.
     */
    pj_status_t (*destroy)(pjmedia_transport *tp);
 
    /**
     * This function is called by the stream when the transport is about
     * to be used by the stream for the first time, and it tells the transport
     * about remote RTP address to send the packet and some callbacks to be
     * called for incoming packets.
     *
     * Application should call #pjmedia_transport_attach2() instead of
     * calling this function directly.
     */
    pj_status_t (*attach2)(pjmedia_transport *tp,
			   pjmedia_transport_attach_param *att_param);
};
```

op的默认初始化方法

```c
static pjmedia_transport_op transport_udp_op = 
{
    &transport_get_info,
    &transport_attach,
    &transport_detach,
    &transport_send_rtp,
    &transport_send_rtcp,
    &transport_send_rtcp2,
    &transport_media_create,
    &transport_encode_sdp,
    &transport_media_start,
    &transport_media_stop,
    &transport_simulate_lost,
    &transport_destroy,
    &transport_attach2
};
```

这里主要看attach2，这个函数传入rtp和rtcp的回调函数指针，当从网络收到数据时，会通过该回调通知。

#### pjmedia_transport_attach_param

```c
/**
 * This structure describes the data passed when calling
 * #pjmedia_transport_attach2().
 */
struct pjmedia_transport_attach_param
{
    /**
     * The media stream.
     */
    void *stream;

    /**
     * Indicate the stream type, either it's audio (PJMEDIA_TYPE_AUDIO) 
     * or video (PJMEDIA_TYPE_VIDEO).
     */
    pjmedia_type media_type;

    /**
     * Remote RTP address to send RTP packet to.
     */
    pj_sockaddr rem_addr;

    /**
     * Optional remote RTCP address. If the argument is NULL
     * or if the address is zero, the RTCP address will be
     * calculated from the RTP address (which is RTP port plus one).
     */
    pj_sockaddr rem_rtcp;

    /**
     * Length of the remote address.
     */
    unsigned addr_len;

    /**
     * Arbitrary user data to be set when the callbacks are called.
     */
    void *user_data;

    /**
     * Callback to be called when RTP packet is received on the transport.
     */
    void (*rtp_cb)(void *user_data, void *pkt, pj_ssize_t);

    /**
     * Callback to be called when RTCP packet is received on the transport.
     */
    void (*rtcp_cb)(void *user_data, void *pkt, pj_ssize_t);

    /**
     * Callback to be called when RTP packet is received on the transport.
     */
    void (*rtp_cb2)(pjmedia_tp_cb_param *param);

};

```

#### transport_udp

```c
struct transport_udp
{
    pjmedia_transport   base;           /**< Base transport.                */

    pj_pool_t          *pool;           /**< Memory pool                    */
    unsigned            options;        /**< Transport options.             */
    unsigned            media_options;  /**< Transport media options.       */
    void               *user_data;      /**< Only valid when attached       */
    //pj_bool_t         attached;       /**< Has attachment?                */
    pj_bool_t           started;        /**< Has started?                   */
    pj_sockaddr         rem_rtp_addr;   /**< Remote RTP address             */
    pj_sockaddr         rem_rtcp_addr;  /**< Remote RTCP address            */
    int                 addr_len;       /**< Length of addresses.           */
    void  (*rtp_cb)(    void*,          /**< To report incoming RTP.        */
                        void*,
                        pj_ssize_t);
    void  (*rtp_cb2)(pjmedia_tp_cb_param*); /**< To report incoming RTP.    */
    void  (*rtcp_cb)(   void*,          /**< To report incoming RTCP.       */
                        void*,
                        pj_ssize_t);

    unsigned            tx_drop_pct;    /**< Percent of tx pkts to drop.    */
    unsigned            rx_drop_pct;    /**< Percent of rx pkts to drop.    */
    pj_ioqueue_t        *ioqueue;       /**< Ioqueue instance.              */

    pj_sock_t           rtp_sock;       /**< RTP socket                     */
    pj_sockaddr         rtp_addr_name;  /**< Published RTP address.         */
    pj_ioqueue_key_t   *rtp_key;        /**< RTP socket key in ioqueue      */
    pj_ioqueue_op_key_t rtp_read_op;    /**< Pending read operation         */
    unsigned            rtp_write_op_id;/**< Next write_op to use           */
    pending_write       rtp_pending_write[MAX_PENDING];  /**< Pending write */
    pj_sockaddr         rtp_src_addr;   /**< Actual packet src addr.        */
    int                 rtp_addrlen;    /**< Address length.                */
    char                rtp_pkt[RTP_LEN];/**< Incoming RTP packet buffer    */

    pj_bool_t           enable_rtcp_mux;/**< Enable RTP & RTCP multiplexing?*/
    pj_bool_t           use_rtcp_mux;   /**< Use RTP & RTCP multiplexing?   */
    pj_sock_t           rtcp_sock;      /**< RTCP socket                    */
    pj_sockaddr         rtcp_addr_name; /**< Published RTCP address.        */
    pj_sockaddr         rtcp_src_addr;  /**< Actual source RTCP address.    */
    unsigned            rtcp_src_cnt;   /**< How many pkt from this addr.   */
    int                 rtcp_addr_len;  /**< Length of RTCP src address.    */
    pj_ioqueue_key_t   *rtcp_key;       /**< RTCP socket key in ioqueue     */
    pj_ioqueue_op_key_t rtcp_read_op;   /**< Pending read operation         */
    pj_ioqueue_op_key_t rtcp_write_op;  /**< Pending write operation        */
    char                rtcp_pkt[RTCP_LEN];/**< Incoming RTCP packet buffer */
};

```

#### pj_ioqueue_callback

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



### **创建udp media transport**

在simpleua.c初始化时，创建完媒体端点pjmedia_endpt后，还会预先创建好udp媒体传输在main.c在sdp和invite session之前

```cpp
    /* 
     * Create media transport used to send/receive RTP/RTCP socket.
     * One media transport is needed for each call. Application may
     * opt to re-use the same media transport for subsequent calls.
     */
    for (i = 0; i < PJ_ARRAY_SIZE(g_med_transport); ++i) {
      status = pjmedia_transport_udp_create3(g_med_endpt, AF, NULL, NULL, 
                     RTP_PORT + i*2, 0, 
                     &g_med_transport[i]);
```

pjmedia_transport_udp_create最终调用pjmedia_transport_udp_create3，这个函数先创建rtp和rtcp两个socket，然后调用pjmedia_transport_udp_attach。

#### transport_udp_create调用流

pjmedia_transport_udp_create3 (Create & Bind RTP & RTCP socket)->pjmedia_transport_udp_attach(创建transport_udp,初始化socket infos，pj_ioqueue_register_sock2：注册socket到 I/O queue同时设置回调函数，以便异步接受消息)

##### pjmedia_transport_udp_create3

```cpp
/**
 * Create UDP stream transport.
 */
PJ_DEF(pj_status_t) pjmedia_transport_udp_create3(pjmedia_endpt *endpt,
						  int af,
						  const char *name,
						  const pj_str_t *addr,
						  int port,
						  unsigned options,
						  pjmedia_transport **p_tp)
{
    pjmedia_sock_info si;
    pj_status_t status;
 
    
    /* Sanity check */
    PJ_ASSERT_RETURN(endpt && port && p_tp, PJ_EINVAL);
 
 
    pj_bzero(&si, sizeof(pjmedia_sock_info));
    si.rtp_sock = si.rtcp_sock = PJ_INVALID_SOCKET;
 
    /* Create RTP socket */
    status = pj_sock_socket(af, pj_SOCK_DGRAM(), 0, &si.rtp_sock);
    if (status != PJ_SUCCESS)
			goto on_error;
 
    /* Bind RTP socket */
    status = pj_sockaddr_init(af, &si.rtp_addr_name, addr, (pj_uint16_t)port);
    if (status != PJ_SUCCESS)
			goto on_error;
 
    status = pj_sock_bind(si.rtp_sock, &si.rtp_addr_name, 
			  pj_sockaddr_get_len(&si.rtp_addr_name));
    if (status != PJ_SUCCESS)
			goto on_error;
 
 
    /* Create RTCP socket */
    status = pj_sock_socket(af, pj_SOCK_DGRAM(), 0, &si.rtcp_sock);
    if (status != PJ_SUCCESS)
			goto on_error;
 
    /* Bind RTCP socket */
    status = pj_sockaddr_init(af, &si.rtcp_addr_name, addr, 
			      (pj_uint16_t)(port+1));
    if (status != PJ_SUCCESS)
			goto on_error;
 
    status = pj_sock_bind(si.rtcp_sock, &si.rtcp_addr_name,
			  pj_sockaddr_get_len(&si.rtcp_addr_name));
    if (status != PJ_SUCCESS)
			goto on_error;
 
    
    /* Create UDP transport by attaching socket info */
    return pjmedia_transport_udp_attach( endpt, name, &si, options, p_tp);
 
 
on_error:
    if (si.rtp_sock != PJ_INVALID_SOCKET)
			pj_sock_close(si.rtp_sock);
    if (si.rtcp_sock != PJ_INVALID_SOCKET)
			pj_sock_close(si.rtcp_sock);
    return status;
}
```

Create & Bind RTP & RTCP socket、Create UDP transport by attaching socket info

##### pjmedia_transport_udp_attach

[pjmedia_transport_udp_attach](html/pjmedia_transport_udp_attach.html)

```c
/**
 * Create UDP stream transport from existing socket info.
 */
PJ_DEF(pj_status_t) pjmedia_transport_udp_attach( pjmedia_endpt *endpt,
                                                  const char *name,
                                                  const pjmedia_sock_info *si,
                                                  unsigned options,
                                                  pjmedia_transport **p_tp)
{
    struct transport_udp *tp;
    pj_pool_t *pool;
    pj_ioqueue_t *ioqueue;
    pj_ioqueue_callback rtp_cb, rtcp_cb;
    pj_grp_lock_t *grp_lock;
    pj_status_t status;


    /* Sanity check */
    PJ_ASSERT_RETURN(endpt && si && p_tp, PJ_EINVAL);

    /* Get ioqueue instance */
    ioqueue = pjmedia_endpt_get_ioqueue(endpt);

    if (name==NULL)
        name = "udp%p";

    /* Create transport structure */
    pool = pjmedia_endpt_create_pool(endpt, name, 512, 512);
    if (!pool)
        return PJ_ENOMEM;
		
    tp = PJ_POOL_ZALLOC_T(pool, struct transport_udp);
    tp->pool = pool;
    tp->options = options;
    pj_memcpy(tp->base.name, pool->obj_name, PJ_MAX_OBJ_NAME);
    tp->base.op = &transport_udp_op;
    tp->base.type = PJMEDIA_TRANSPORT_TYPE_UDP;

    /* Copy socket infos */
    tp->rtp_sock = si->rtp_sock;
    tp->rtp_addr_name = si->rtp_addr_name;
    tp->rtcp_sock = si->rtcp_sock;
    tp->rtcp_addr_name = si->rtcp_addr_name;

    /* If address is 0.0.0.0, use host's IP address */
    if (!pj_sockaddr_has_addr(&tp->rtp_addr_name)) {
        pj_sockaddr hostip;

        status = pj_gethostip(tp->rtp_addr_name.addr.sa_family, &hostip);
        if (status != PJ_SUCCESS)
            goto on_error;

        pj_memcpy(pj_sockaddr_get_addr(&tp->rtp_addr_name), 
                  pj_sockaddr_get_addr(&hostip),
                  pj_sockaddr_get_addr_len(&hostip));
    }

    /* Same with RTCP */
    if (!pj_sockaddr_has_addr(&tp->rtcp_addr_name)) {
        pj_memcpy(pj_sockaddr_get_addr(&tp->rtcp_addr_name),
                  pj_sockaddr_get_addr(&tp->rtp_addr_name),
                  pj_sockaddr_get_addr_len(&tp->rtp_addr_name));
    }

    /* Create group lock */
    status = pj_grp_lock_create(pool, NULL, &grp_lock);
    if (status != PJ_SUCCESS)
        goto on_error;

    pj_grp_lock_add_ref(grp_lock);
    tp->base.grp_lock = grp_lock;

    /* Setup RTP socket with the ioqueue */
    pj_bzero(&rtp_cb, sizeof(rtp_cb));
    rtp_cb.on_read_complete = &on_rx_rtp;
    rtp_cb.on_write_complete = &on_rtp_data_sent;

    status = pj_ioqueue_register_sock2(pool, ioqueue, tp->rtp_sock, grp_lock,
                                       tp, &rtp_cb, &tp->rtp_key);
    if (status != PJ_SUCCESS)
        goto on_error;
    
    /* Disallow concurrency so that detach() and destroy() are
     * synchronized with the callback.
     *
     * Note that we still need this even after group lock is added to
     * maintain the above behavior.
     */
    status = pj_ioqueue_set_concurrency(tp->rtp_key, PJ_FALSE);
    if (status != PJ_SUCCESS)
        goto on_error;
        
    /* Setup RTCP socket with ioqueue */
    pj_bzero(&rtcp_cb, sizeof(rtcp_cb));
    rtcp_cb.on_read_complete = &on_rx_rtcp;

    status = pj_ioqueue_register_sock2(pool, ioqueue, tp->rtcp_sock, grp_lock,
                                       tp, &rtcp_cb, &tp->rtcp_key);
    if (status != PJ_SUCCESS)
        goto on_error;

    status = pj_ioqueue_set_concurrency(tp->rtcp_key, PJ_FALSE);
    if (status != PJ_SUCCESS)
        goto on_error;

    tp->ioqueue = ioqueue;

    /* Done */
    *p_tp = &tp->base;
    return PJ_SUCCESS;


on_error:
    transport_destroy(&tp->base);
    return status;
}


```

创建transport_udp ：PJ_POOL_ZALLOC_T，Copy socket infos 到transport_udp、 Setup RTP/RTCP socket with the ioqueue（设置回调函数on_rx_rtp、on_rtp_data_sent、on_rx_rtcp）。

udp_attach先申请UDP媒体传输结构体transport_udp *tp的内存，注意，此结构体包含了媒体传输pjmedia_transport和一些回调，但是这些回调还没有设置。其中的操作集指向transport_udp_op。接着把socket注册到媒体端点中的io队列，io队列的读完成回调是on_rx_rtp。从这里可以知道，从网络读到数据时，会调用transport_udp.c中的on_rx_rtp，而在这个回调里，会再调用transport_udp中的回调rtp_cb和rtp_cb2，而这两个回调，创建的时候还没有设置，要等到调用操作集的attach才会设置。

注意，这里有两个attach的地方，一个是创建的时候，调用pjmedia_transport_udp_attach，这个attach会把socket注册到ioqueue，同时ioqueue的读完成回调为transport_udp.c中的on_rx_rtp。

##### pj_ioqueue_register_sock2

注册一个套接字到I/O队列框架。当一个套接字注册到IO队列时，它可以被修改为使用非阻塞IO。如果被修改了，就不能保证在套接字取消注册后会恢复这种修改。

- **pool** – To allocate the resource for the specified handle, which must be valid until the handle/key is unregistered from I/O Queue.
- **ioque** – The I/O Queue.
- **sock** – The socket.
- **user_data** – User data to be associated with the key, which can be retrieved later.
- **cb** – Callback to be called when I/O opertion completes.
- **key** – Pointer to receive the key to be associated with this socket. Subsequent I/O queue operation will need this key.

第二个attach是transport_attach/transport_attach2，这个attach会再传入rtp_cb和rtcp_cb，而这两个回调，会被on_rx_rtp调用，所以这里有两个回调。总结数据流方向，从网络收到数据，最后会进入attach传入的rtp_cb。但是这个rtp_cb什么时候设置，设置的是谁，这个是在 stream 中实现，下一篇再说。

#### pjmedia_transport_attach2调用流

pjmedia_transport_attach2-》transport_attach2-》tp_attach

tp_attach最重要就是设置了pjmedia_transport 的cb2 为on_rx_request，还有Copy remote RTP address





