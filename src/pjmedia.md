## pjmedia总述

转载：[pjmedia_HYQ458941968的博客-CSDN博客](https://blog.csdn.net/hyq458941968/category_11380864.html)

从对象关系来看：

**一次session，多个endpoint参与，彼此发送stream，每个stream两个channel分别收发。音频设备port对象采集播放声音，transport最终发送接收信号。**

1、 pjmedia_endpt，代表一个媒体端点，端点可以理解为一个节点，可以是服务器或者客户端，一个设备一般只会有唯一一个端点，而且在初始化的时候创建。

2、pjmedia_session，代表一次会话，一个会话可以只有两个，也可以有多个参与者会议。

3、pjmedia_stream，代表一个流，在于一方进行通讯时，一般可以有音频流、视频流。

4、pjmedia_channel，只有一个方向的媒体流，也就是说，对应音频流，要发送通道和接收通道两个channel。

以上是范围从大到小的媒体对象，下面介绍其它对象。

5、pjmedia_transport，传输对象，封装了所有从网络接收数据和发送数据到网络的操作，可以是UDP、SRTP、ICE等传输方式。

6、pjmedia_snd_port，音频设备对象，最终的音频裸流都要在设备上播放，从麦克风采集，此对象封装所有设备操作。

从数据流来看：

![img](img/20190917203551691.png)

左边为最底层，音频设备的播放和采集，通过回调接口，调用stream.c生产或消化数据，最后通过transport传输接口进行发送接收。当然，其中还夹杂着前后处理、编解码、抖动缓冲区等，后面的章节会对各个对象及相关的音频处理进行描述。

最后来分析一下实例代码simpleua.c中对pjmedia的使用方法。

### **初始化**

在main函数中，前半部分主要是对sip的初始化，对pjmedia的初始化大概从358行开始

#### 1、创建媒体端点endpoint

```cpp
static pjmedia_endpt	    *g_med_endpt;   /* Media endpoint.		*/
 
#if PJ_HAS_THREADS
    status = pjmedia_endpt_create(&cp.factory, NULL, 1, &g_med_endpt);
#else
    status = pjmedia_endpt_create(&cp.factory, 
				  pjsip_endpt_get_ioqueue(g_endpt), 
				  0, &g_med_endpt);
#endif
```

这里我们默认看有多线程的流程。

#### 2、创建udp网络接口transport

```cpp
static pjmedia_transport    *g_med_transport[MAX_MEDIA_CNT];
   
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

可以看出，虽然初始化还没有建立媒体通讯，但是预先创建了若干传输对象。

#### 3、loop处理sip事件

```cpp
/* Loop until one call is completed */
for (;!g_complete;) {
	pj_time_val timeout = {0, 10};
	pjsip_endpt_handle_events(g_endpt, &timeout);
}
```

初始化的最后是一个循环等待处理sip对象的操作。

### **建立媒体通讯**

当sip,sdp协商成功后，则要开始媒体通讯，发生在回调函数call_on_media_update

#### 1、创建并启动流对象stream

```cpp
static pjmedia_stream       *g_med_stream;  /* Call's audio stream.	*/
    
    /* Create stream info based on the media audio SDP. */
    status = pjmedia_stream_info_from_sdp(&stream_info, inv->dlg->pool,
					  g_med_endpt,
					  local_sdp, remote_sdp, 0);
    
    /* Create new audio media stream, passing the stream info, and also the
     * media socket that we created earlier.
     */
    status = pjmedia_stream_create(g_med_endpt, inv->dlg->pool, &stream_info,
				   g_med_transport[0], NULL, &g_med_stream);
 
    /* Start the audio stream */
    status = pjmedia_stream_start(g_med_stream);
```

这个实例并没有创建session，而是直接创建流对象。先从sip协商的sdp中获取媒体信息pjmedia_stream_info_from_sdp，然后根据这些信息创建流对象pjmedia_stream_create，并紧接着启动流pjmedia_stream_start。这里只有音频流，视频流暂不纳入分析范围。

#### 2、启动网络传输transport

```cpp
    /* Start the UDP media transport */
    pjmedia_transport_media_start(g_med_transport[0], 0, 0, 0, 0);
```

前面讲到，初始化的时候预创建了若干传输对象，但是并没有启动，等到协商成功后，才启动网络传输。

#### 3、创建音频设备对象Sound_device

```cpp
static pjmedia_snd_port	    *g_snd_port;    /* Sound device.		*/
 
    /* Get the media port interface of the audio stream. 
     * Media port interface is basicly a struct containing get_frame() and
     * put_frame() function. With this media port interface, we can attach
     * the port interface to conference bridge, or directly to a sound
     * player/recorder device.
     */
    pjmedia_stream_get_port(g_med_stream, &media_port);
 
    /* Create sound port */
    pjmedia_snd_port_create(inv->pool,
                            PJMEDIA_AUD_DEFAULT_CAPTURE_DEV,
                            PJMEDIA_AUD_DEFAULT_PLAYBACK_DEV,
                            PJMEDIA_PIA_SRATE(&media_port->info),/* clock rate	    */
                            PJMEDIA_PIA_CCNT(&media_port->info),/* channel count    */
                            PJMEDIA_PIA_SPF(&media_port->info), /* samples per frame*/
                            PJMEDIA_PIA_BITS(&media_port->info),/* bits per sample  */
                            0,
                            &g_snd_port);
 
    status = pjmedia_snd_port_connect(g_snd_port, media_port);
```

这里先从流对象取出媒体接口pjmedia_stream_get_port，其中的媒体接口包含了数据回调，这些后面分析数据流再重点讲，然后创建pjmedia_snd_port_create并启动pjmedia_snd_port_connect音频设备对象。

### **结束销毁**

```cpp
    /* Destroy audio ports. Destroy the audio port first
     * before the stream since the audio port has threads
     * that get/put frames to the stream.
     */
    if (g_snd_port)
			pjmedia_snd_port_destroy(g_snd_port);
 
 
    /* Destroy streams */
    if (g_med_stream)
      pjmedia_stream_destroy(g_med_stream);
 
    /* Destroy media transports */
    for (i = 0; i < MAX_MEDIA_CNT; ++i) {
      if (g_med_transport[i])
          pjmedia_transport_close(g_med_transport[i]);
    }

        /* Deinit pjmedia endpoint */
    if (g_med_endpt)
      pjmedia_endpt_destroy(g_med_endpt);

        /* Deinit pjsip endpoint */
    if (g_endpt)
      pjsip_endpt_destroy(g_endpt);

        /* Release pool */
    if (pool)
      pj_pool_release(pool);
```

当主线程退出loop时，则销毁所有创建的对象。

## pjmedia_endpt

simpleua.c在进行媒体相关初始化时，首先创建媒体端点，看看媒体端点的数据结构和创建流程。

```cpp
#if PJ_HAS_THREADS
    status = pjmedia_endpt_create(&cp.factory, NULL, 1, &g_med_endpt);
#else
    status = pjmedia_endpt_create(&cp.factory, 
				  pjsip_endpt_get_ioqueue(g_endpt), 
				  0, &g_med_endpt);
#endif
```

### **pjmedia_endpt结构体**

```cpp
/** Concrete declaration of media endpoint. */
struct pjmedia_endpt
{
    /** Pool. */
    pj_pool_t		 *pool;
 
    /** Pool factory. */
    pj_pool_factory	 *pf;
 
    /** Codec manager. */
    pjmedia_codec_mgr	  codec_mgr;
 
    /** IOqueue instance. */
    pj_ioqueue_t 	 *ioqueue;
 
    /** Do we own the ioqueue? */
    pj_bool_t		  own_ioqueue;
 
    /** Number of threads. */
    unsigned		  thread_cnt;
 
    /** IOqueue polling thread, if any. */
    pj_thread_t		 *thread[MAX_THREADS];
 
    /** To signal polling thread to quit. */
    pj_bool_t		  quit_flag;
 
    /** Is telephone-event enable */
    pj_bool_t		  has_telephone_event;
 
    /** List of exit callback. */
    exit_cb		  exit_cb_list;
};
```

ioqueue：用于绑定socket，然后异步接收数据；

thread[]：线程数组，存储工作线程。

codec_mgr：codec的管理者

exit_cb_list：退出时的回调链表

quit_flag：置位则工作线程都退出

### **创建端点**

```cpp
/**
 * Initialize and get the instance of media endpoint.
 */
PJ_DEF(pj_status_t) pjmedia_endpt_create2(pj_pool_factory *pf,
					  pj_ioqueue_t *ioqueue,
					  unsigned worker_cnt,
					  pjmedia_endpt **p_endpt)
{
    pj_pool_t *pool;
    pjmedia_endpt *endpt;
    unsigned i;
    pj_status_t status;
 
 
    pool = pj_pool_create(pf, "med-ept", 512, 512, NULL);
    if (!pool)
			return PJ_ENOMEM;
 
    endpt = PJ_POOL_ZALLOC_T(pool, struct pjmedia_endpt);
    endpt->pool = pool;
    endpt->pf = pf;
    endpt->ioqueue = ioqueue;
    endpt->thread_cnt = worker_cnt;
    endpt->has_telephone_event = PJ_TRUE;
 
 
    /* Init codec manager. */
    status = pjmedia_codec_mgr_init(&endpt->codec_mgr, endpt->pf);
    if (status != PJ_SUCCESS)
			goto on_error;
 
    /* Initialize exit callback list. */
    pj_list_init(&endpt->exit_cb_list);
 
    /* Create ioqueue if none is specified. */
    if (endpt->ioqueue == NULL) {
	
    endpt->own_ioqueue = PJ_TRUE;

    status = pj_ioqueue_create( endpt->pool, PJ_IOQUEUE_MAX_HANDLES,
              &endpt->ioqueue);
    if (status != PJ_SUCCESS)
        goto on_error;

    }
 
    /* Create worker threads if asked. */
    for (i=0; i<worker_cnt; ++i) {
      status = pj_thread_create( endpt->pool, "media", &worker_proc,
               endpt, 0, 0, &endpt->thread[i]);
      if (status != PJ_SUCCESS)
          goto on_error;
    }
 
 
    *p_endpt = endpt;
    return PJ_SUCCESS;
 
on_error:
 
    /* Destroy threads */
    for (i=0; i<endpt->thread_cnt; ++i) {
      if (endpt->thread[i]) {
          pj_thread_destroy(endpt->thread[i]);
      }
    }
 
    /* Destroy internal ioqueue */
    if (endpt->ioqueue && endpt->own_ioqueue)
			pj_ioqueue_destroy(endpt->ioqueue);
 
    pjmedia_codec_mgr_destroy(&endpt->codec_mgr);
    //pjmedia_aud_subsys_shutdown();
    pj_pool_release(pool);
    return status;
}
```

1、创建内存池med-ept

2、申请媒体端点结构体内存，并把外部的ioqueue地址存到结构体 endpt->ioqueue = ioqueue;

3、初始化编码器manager pjmedia_codec_mgr_init(&endpt->codec_mgr, endpt->pf)

4、如果初入的ioqueue为空，则创建一个新的ioqueue

5、创建工作线程组，线程函数是worker_proc

**工作线程**

```cpp
/**
 * Worker thread proc.
 */
static int PJ_THREAD_FUNC worker_proc(void *arg)
{
    pjmedia_endpt *endpt = (pjmedia_endpt*) arg;
 
    while (!endpt->quit_flag) {
        pj_time_val timeout = { 0, 500 };
        pj_ioqueue_poll(endpt->ioqueue, &timeout);
    }
 
    return 0;
}
```

工作线程就是每500ms调用一次pj_ioqueue_pool。刚创建媒体端点时，ioqueue还没有注册socket，所以不会监控到数据。

## pjmedia_transport

媒体传输封装了网络收发细节，pjmedia_transport可以是udp、srtp、ice等，这里以udp为例。

### **结构体pjmedia_transport**

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

### pjmedia_transport_op

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

这里主要看attach2，这个函数传入rtp和rtcp的回调函数指针，当从网络收到数据时，会通过该回调通知。

### **创建udp媒体传输**

在simpleua.c初始化时，创建完媒体端点pjmedia_endpt后，还会预先创建好udp媒体传输

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

#### 调用流：

pjmedia_transport_udp_create3、pjmedia_transport_udp_attach、pj_ioqueue_register_sock2

### pjmedia_transport_udp_create3

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

### transport_udp

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

### pjmedia_transport_udp_attach

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

创建transport_udp PJ_POOL_ZALLOC_T，Copy socket infos 到transport_udp、 Setup RTP/RTCP socket with the ioqueue（设置两个回调函数on_rx_rtp、on_rtp_data_sent、on_rx_rtcp）。

udp_attach先申请UDP媒体传输结构体transport_udp *tp的内存，注意，此结构体包含了媒体传输pjmedia_transport和一些回调，但是这些回调还没有设置。其中的操作集指向transport_udp_op。接着把socket注册到媒体端点中的io队列，io队列的读完成回调是on_rx_rtp。从这里可以知道，从网络读到数据时，会调用transport_udp.c中的on_rx_rtp，而在这个回调里，会再调用transport_udp中的回调rtp_cb和rtp_cb2，而这两个回调，创建的时候还没有设置，要等到调用操作集的attach才会设置。

注意，这里有两个attach的地方，一个是创建的时候，调用pjmedia_transport_udp_attach，这个attach会把socket注册到ioqueue，同时ioqueue的读完成回调为transport_udp.c中的on_rx_rtp。

### pj_ioqueue_register_sock2

注册一个套接字到I/O队列框架。当一个套接字注册到IO队列时，它可以被修改为使用非阻塞IO。如果被修改了，就不能保证在套接字取消注册后会恢复这种修改。

- **pool** – To allocate the resource for the specified handle, which must be valid until the handle/key is unregistered from I/O Queue.
- **ioque** – The I/O Queue.
- **sock** – The socket.
- **user_data** – User data to be associated with the key, which can be retrieved later.
- **cb** – Callback to be called when I/O opertion completes.
- **key** – Pointer to receive the key to be associated with this socket. Subsequent I/O queue operation will need this key.

第二个attach是操作集的attach，看看上面提到的操作集

```cpp
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

第二个attach是transport_attach/transport_attach2，这个attach会再传入rtp_cb和rtcp_cb，而这两个回调，会被on_rx_rtp调用，所以这里有两个回调。总结数据流方向，从网络收到数据，最后会进入attach传入的rtp_cb。但是这个rtp_cb什么时候设置，设置的是谁，这个是在 stream 中实现，下一篇再说。

