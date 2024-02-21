## Stream

### pjmedia_stream_info

对应sdp中的 m=字段 （媒体名称和传输地址）

```makefile
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 126
//m=audio说明本会话包含音频，9代表音频使用端口9来传输，但是在webrtc中一现在一般不使用，如果设置为0，代表不
//传输音频,UDP/TLS/RTP/SAVPF是表示用户来传输音频支持的协议，udp，tls,rtp代表使用udp来传输rtp包，并使用tls加密
//SAVPF代表使用srtcp的反馈机制来控制通信过程,后台111 103 104 9 0 8 106 105 13 126表示本会话音频支持的编码，后台几行会有详细补充说明
```



```c
/**
 * This structure describes media stream information. Each media stream
 * corresponds to one "m=" line in SDP session descriptor, and it has
 * its own RTP/RTCP socket pair.
 */
typedef struct pjmedia_stream_info
{
    pjmedia_type        type;       /**< Media type (audio, video)          */
    pjmedia_tp_proto    proto;      /**< Transport protocol (RTP/AVP, etc.) */
    pjmedia_dir         dir;        /**< Media direction.                   */
    pj_sockaddr         local_addr; /**< Local RTP address                  */
    pj_sockaddr         rem_addr;   /**< Remote RTP address                 */
    pj_sockaddr         rem_rtcp;   /**< Optional remote RTCP address. If
                                         sin_family is zero, the RTP address
                                         will be calculated from RTP.       */
    pj_bool_t           rtcp_mux;   /**< Use RTP and RTCP multiplexing.     */
#if defined(PJMEDIA_HAS_RTCP_XR) && (PJMEDIA_HAS_RTCP_XR != 0)
    pj_bool_t           rtcp_xr_enabled;
                                    /**< Specify whether RTCP XR is enabled.*/
    pj_uint32_t         rtcp_xr_interval; /**< RTCP XR interval.            */
    pj_sockaddr         rtcp_xr_dest;/**<Additional remote RTCP XR address.
                                         This is useful for third-party (e.g:
                                         network monitor) to monitor the 
                                         stream. If sin_family is zero, 
                                         this will be ignored.              */
#endif
    pjmedia_rtcp_fb_info loc_rtcp_fb; /**< Local RTCP-FB info.              */
    pjmedia_rtcp_fb_info rem_rtcp_fb; /**< Remote RTCP-FB info.             */
    pjmedia_codec_info  fmt;        /**< Incoming codec format info.        */
    pjmedia_codec_param *param;     /**< Optional codec param.              */
    unsigned            tx_pt;      /**< Outgoing codec paylaod type.       */
    unsigned            rx_pt;      /**< Incoming codec paylaod type.       */
    unsigned            tx_maxptime;/**< Outgoing codec max ptime.          */
    int                 tx_event_pt;/**< Outgoing pt for telephone-events.  */
    int                 rx_event_pt;/**< Incoming pt for telephone-events.  */
    pj_uint32_t         ssrc;       /**< RTP SSRC.                          */
    pj_str_t            cname;      /**< RTCP CNAME.                        */
    pj_bool_t           has_rem_ssrc;/**<Has remote RTP SSRC?               */
    pj_uint32_t         rem_ssrc;   /**< Remote RTP SSRC.                   */
    pj_str_t            rem_cname;  /**< Remote RTCP CNAME.                 */
    pj_uint32_t         rtp_ts;     /**< Initial RTP timestamp.             */
    pj_uint16_t         rtp_seq;    /**< Initial RTP sequence number.       */
    pj_uint8_t          rtp_seq_ts_set;
                                    /**< Bitmask flags if initial RTP sequence 
                                         and/or timestamp for sender are set.
                                         bit 0/LSB : sequence flag 
                                         bit 1     : timestamp flag         */
    int                 jb_init;    /**< Jitter buffer init delay in msec.  
                                         (-1 for default).                  */
    int                 jb_min_pre; /**< Jitter buffer minimum prefetch
                                         delay in msec (-1 for default).    */
    int                 jb_max_pre; /**< Jitter buffer maximum prefetch
                                         delay in msec (-1 for default).    */
    int                 jb_max;     /**< Jitter buffer max delay in msec.   */
    pjmedia_jb_discard_algo jb_discard_algo;
                                    /**< Jitter buffer discard algorithm.   */

#if defined(PJMEDIA_STREAM_ENABLE_KA) && PJMEDIA_STREAM_ENABLE_KA!=0
    pj_bool_t           use_ka;     /**< Stream keep-alive and NAT hole punch
                                         (see #PJMEDIA_STREAM_ENABLE_KA)
                                         is enabled?                        */
    pjmedia_stream_ka_config ka_cfg;
                                    /**< Stream send kep-alive settings.    */
#endif
    pj_bool_t           rtcp_sdes_bye_disabled; 
                                    /**< Disable automatic sending of RTCP
                                         SDES and BYE.                      */
} pjmedia_stream_info;
```

### pjmedia_stream

```c
/**
 * This structure describes media stream.
 * A media stream is bidirectional media transmission between two endpoints.
 * It consists of two channels, i.e. encoding and decoding channels.
 * A media stream corresponds to a single "m=" line in a SDP session
 * description.
 */
struct pjmedia_stream
{
    pjmedia_endpt           *endpt;         /**< Media endpoint.            */
    pjmedia_codec_mgr       *codec_mgr;     /**< Codec manager instance.    */
    pjmedia_stream_info      si;            /**< Creation parameter.        */
    pjmedia_port             port;          /**< Port interface.            */
    pjmedia_channel         *enc;           /**< Encoding channel.          */
    pjmedia_channel         *dec;           /**< Decoding channel.          */

    pj_pool_t               *own_pool;      /**< Only created if not given  */

 
    pjmedia_dir              dir;           /**< Stream direction.          */
    void                    *user_data;     /**< User data.                 */
    pj_str_t                 cname;         /**< SDES CNAME                 */

    pjmedia_transport       *transport;     /**< Stream transport.          */

    pjmedia_codec           *codec;         /**< Codec instance being used. */
    pjmedia_codec_param      codec_param;   /**< Codec param.               */
    pj_int16_t              *enc_buf;       /**< Encoding buffer, when enc's
                                                 ptime is different than dec.
                                                 Otherwise it's NULL.       */

    unsigned                 enc_samples_per_pkt;
    unsigned                 enc_buf_size;  /**< Encoding buffer size, in
                                                 samples.                   */
    unsigned                 enc_buf_pos;   /**< First position in buf.     */
    unsigned                 enc_buf_count; /**< Number of samples in the
                                                 encoding buffer.           */

    pj_int16_t              *dec_buf;       /**< Decoding buffer.           */
    unsigned                 dec_buf_size;  /**< Decoding buffer size, in
                                                 samples.                   */
    unsigned                 dec_buf_pos;   /**< First position in buf.     */
    unsigned                 dec_buf_count; /**< Number of samples in the
                                                 decoding buffer.           */

    pj_uint16_t              dec_ptime;     /**< Decoder frame ptime in ms. */
    pj_uint8_t               dec_ptime_denum;/**< Decoder ptime denum.      */
    pj_bool_t                detect_ptime_change;
                                            /**< Detect decode ptime change */

    unsigned                 plc_cnt;       /**< # of consecutive PLC frames*/
    unsigned                 max_plc_cnt;   /**< Max # of PLC frames        */

    unsigned                 vad_enabled;   /**< VAD enabled in param.      */
    unsigned                 frame_size;    /**< Size of encoded base frame.*/
    pj_bool_t                is_streaming;  /**< Currently streaming?. This
                                                 is used to put RTP marker
                                                 bit.                       */
    pj_uint32_t              ts_vad_disabled;/**< TS when VAD was disabled. */
    pj_uint32_t              tx_duration;   /**< TX duration in timestamp.  */

    pj_mutex_t              *jb_mutex;
    pjmedia_jbuf            *jb;            /**< Jitter buffer.             */
    char                     jb_last_frm;   /**< Last frame type from jb    */
    unsigned                 jb_last_frm_cnt;/**< Last JB frame type counter*/
    unsigned                 soft_start_cnt;/**< Stream soft start counter */

    pjmedia_rtcp_session     rtcp;          /**< RTCP for incoming RTP.     */

    pj_uint32_t              rtcp_last_tx;  /**< RTCP tx time in timestamp  */
    pj_uint32_t              rtcp_interval; /**< Interval, in timestamp.    */
    pj_bool_t                initial_rr;    /**< Initial RTCP RR sent       */
    pj_bool_t                rtcp_sdes_bye_disabled;/**< Send RTCP SDES/BYE?*/
    void                    *out_rtcp_pkt;  /**< Outgoing RTCP packet.      */
    unsigned                 out_rtcp_pkt_size;
                                            /**< Outgoing RTCP packet size. */
    pj_int16_t              *zero_frame;    /**< Zero frame buffer.         */

    /* RFC 2833 DTMF transmission queue: */
    unsigned                 dtmf_duration; /**< DTMF duration(in timestamp)*/
    int                      tx_event_pt;   /**< Outgoing pt for dtmf.      */
    int                      tx_dtmf_count; /**< # of digits in tx dtmf buf.*/
    struct dtmf              tx_dtmf_buf[32];/**< Outgoing dtmf queue.      */

    /* Incoming DTMF: */
    int                      rx_event_pt;   /**< Incoming pt for dtmf.      */
    int                      last_dtmf;     /**< Current digit, or -1.      */
    pj_uint32_t              last_dtmf_dur; /**< Start ts for cur digit.    */
    pj_bool_t                last_dtmf_ended;
    unsigned                 rx_dtmf_count; /**< # of digits in dtmf rx buf.*/
    char                     rx_dtmf_buf[32];/**< Incoming DTMF buffer.     */

    /* DTMF callback */
    void                    (*dtmf_cb)(pjmedia_stream*, void*, int);
    void                     *dtmf_cb_user_data;

    void                    (*dtmf_event_cb)(pjmedia_stream*, void*,
                                             const pjmedia_stream_dtmf_event*);
    void                     *dtmf_event_cb_user_data;

#if defined(PJMEDIA_HANDLE_G722_MPEG_BUG) && (PJMEDIA_HANDLE_G722_MPEG_BUG!=0)
    /* Enable support to handle codecs with inconsistent clock rate
     * between clock rate in SDP/RTP & the clock rate that is actually used.
     * This happens for example with G.722 and MPEG audio codecs.
     */
    pj_bool_t                has_g722_mpeg_bug;
                                            /**< Flag to specify whether
                                                 normalization process
                                                 is needed                  */
    unsigned                 rtp_tx_ts_len_per_pkt;
                                            /**< Normalized ts length per packet
                                                 transmitted according to
                                                 'erroneous' definition     */
    unsigned                 rtp_rx_ts_len_per_frame;
                                            /**< Normalized ts length per frame
                                                 received according to
                                                 'erroneous' definition     */
    unsigned                 rtp_rx_last_cnt;/**< Nb of frames in last pkt  */
    unsigned                 rtp_rx_check_cnt;
                                            /**< Counter of remote timestamp
                                                 checking */
#endif


    pj_sockaddr              rem_rtp_addr;     /**< Remote RTP address      */
    unsigned                 rem_rtp_flag;     /**< Indicator flag about
                                                    packet from this addr.
                                                    0=no pkt, 1=good ssrc,
                                                    2=bad ssrc pkts         */
    unsigned                 rtp_src_cnt;      /**< How many pkt from
                                                    this addr.              */
    pj_uint32_t              rtp_rx_last_ts;        /**< Last received RTP
                                                         timestamp          */
    pj_uint32_t              rtp_tx_err_cnt;        /**< The number of RTP
                                                         send() error       */
    pj_uint32_t              rtcp_tx_err_cnt;       /**< The number of RTCP
                                                         send() error       */

    /* RTCP Feedback */
    pj_bool_t                send_rtcp_fb_nack;     /**< Send NACK?         */
    pjmedia_rtcp_fb_nack     rtcp_fb_nack;          /**< TX NACK state.     */
    int                      rtcp_fb_nack_cap_idx;  /**< RX NACK cap idx.   */


};

```

## 初始化pjmedia_stream_create

```c
PJ_DEF(pj_status_t) pjmedia_stream_create( pjmedia_endpt *endpt,
                                           pj_pool_t *pool,
                                           const pjmedia_stream_info *info,
                                           pjmedia_transport *tp,
                                           void *user_data,
                                           pjmedia_stream **p_stream)
```

创建stream对象stream = PJ_POOL_ZALLOC_T(pool, pjmedia_stream);

好的，以下是所有变量以及它们的类型和描述的表格：

| 变量                      | 类型                                                         | 描述                                            | 初始化                                                       |
| ------------------------- | ------------------------------------------------------------ | ----------------------------------------------- | :----------------------------------------------------------- |
| `endpt`                   | `pjmedia_endpt*`                                             | 媒体端点。                                      | endpt 函数参数                                               |
| `codec_mgr`               | `pjmedia_codec_mgr*`                                         | 编解码器管理器实例。                            | pjmedia_endpt_get_codec_mgr(endpt);                          |
| `si`                      | `pjmedia_stream_info`                                        | 流的创建参数。                                  | create参数info                                               |
| `port`                    | `pjmedia_port`                                               | 端口接口。                                      | port.info初始化pjmedia_port_info_init ，stream->port.info.fmt |
| `enc`                     | `pjmedia_channel*`                                           | 编码通道。                                      | f                                                            |
| `dec`                     | `pjmedia_channel*`                                           | 解码通道。                                      |                                                              |
| `own_pool`                | `pj_pool_t*`                                                 | 只有在未给定时才会创建。                        |                                                              |
| `dir`                     | `pjmedia_dir`                                                | 流的方向。                                      | info->dir                                                    |
| `user_data`               | `void*`                                                      | 用户数据。                                      | user_data参数                                                |
| `cname`                   | `pj_str_t`                                                   | SDES CNAME。                                    |                                                              |
| `transport`               | `pjmedia_transport*`                                         | 流传输。                                        |                                                              |
| `codec`                   | `pjmedia_codec*`                                             | 正在使用的编解码器实例。                        | pjmedia_codec_mgr_alloc_codec( stream->codec_mgr, &info->fmt, &stream->codec); |
| `codec_param`             | `pjmedia_codec_param`                                        | 编解码器参数。                                  | stream->codec_param = *stream->si.param;或pjmedia_codec_mgr_get_default_param(stream->codec_mgr,&info>fmt,&stream>codec_param); |
| `enc_buf`                 | `pj_int16_t*`                                                | 编码缓冲区，当编码的 ptime 与解码的不同时有效。 |                                                              |
| `enc_samples_per_pkt`     | `unsigned`                                                   | 每个包的编码样本数。                            |                                                              |
| `enc_buf_size`            | `unsigned`                                                   | 编码缓冲区大小，以样本为单位。                  |                                                              |
| `enc_buf_pos`             | `unsigned`                                                   | 缓冲区中的第一个位置。                          |                                                              |
| `enc_buf_count`           | `unsigned`                                                   | 编码缓冲区中的样本数。                          |                                                              |
| `dec_buf`                 | `pj_int16_t*`                                                | 解码缓冲区。                                    |                                                              |
| `dec_buf_size`            | `unsigned`                                                   | 解码缓冲区大小，以样本为单位。                  |                                                              |
| `dec_buf_pos`             | `unsigned`                                                   | 缓冲区中的第一个位置。                          |                                                              |
| `dec_buf_count`           | `unsigned`                                                   | 解码缓冲区中的样本数。                          |                                                              |
| `dec_ptime`               | `pj_uint16_t`                                                | 解码器帧的时间，以毫秒为单位。                  |                                                              |
| `dec_ptime_denum`         | `pj_uint8_t`                                                 | 解码器帧的时间分母。                            |                                                              |
| `detect_ptime_change`     | `pj_bool_t`                                                  | 检测解码 ptime 的变化。                         |                                                              |
| `plc_cnt`                 | `unsigned`                                                   | 连续 PLC 帧的数量。                             |                                                              |
| `max_plc_cnt`             | `unsigned`                                                   | 最大的 PLC 帧数量。                             |                                                              |
| `vad_enabled`             | `unsigned`                                                   | 参数中是否启用了 VAD。                          |                                                              |
| `frame_size`              | `unsigned`                                                   | 编码基本帧的大小。                              |                                                              |
| `is_streaming`            | `pj_bool_t`                                                  | 当前是否正在流式传输？                          |                                                              |
| `ts_vad_disabled`         | `pj_uint32_t`                                                | 禁用 VAD 时的时间戳。                           |                                                              |
| `tx_duration`             | `pj_uint32_t`                                                | 时间戳中的 TX 时长。                            |                                                              |
| `jb_mutex`                | `pj_mutex_t*`                                                | 抖动缓冲区互斥锁。                              | status = pj_mutex_create_simple(pool, NULL, &stream->jb_mutex); |
| `jb`                      | `pjmedia_jbuf*`                                              | 抖动缓冲区。                                    |                                                              |
| `jb_last_frm`             | `char`                                                       | 最后一帧的类型。                                |                                                              |
| `jb_last_frm_cnt`         | `unsigned`                                                   | 上一个 JB 帧类型的计数器。                      |                                                              |
| `soft_start_cnt`          | `unsigned`                                                   | 流软启动计数器。                                |                                                              |
| `rtcp`                    | `pjmedia_rtcp_session`                                       | 传入 RTP 的 RTCP。                              |                                                              |
| `rtcp_last_tx`            | `pj_uint32_t`                                                | RTCP 的上次发送时间戳。                         |                                                              |
| `rtcp_interval`           | `pj_uint32_t`                                                | 间隔时间戳。                                    |                                                              |
| `initial_rr`              | `pj_bool_t`                                                  | 是否已发送初始的 RTCP RR。                      |                                                              |
| `rtcp_sdes_bye_disabled`  | `pj_bool_t`                                                  | 是否发送 RTCP SDES/BYE？                        |                                                              |
| `out_rtcp_pkt`            | `void*`                                                      | 出站 RTCP 数据包。                              |                                                              |
| `out_rtcp_pkt_size`       | `unsigned`                                                   | 出站 RTCP 数据包大小。                          |                                                              |
| `zero_frame`              | `pj_int16_t*`                                                | 零帧缓冲区。                                    |                                                              |
| `dtmf_duration`           | `unsigned`                                                   | DTMF 时长（时间戳）。                           |                                                              |
| `tx_event_pt`             | `int`                                                        | DTMF 的传输事件 PT。                            |                                                              |
| `tx_dtmf_count`           | `int`                                                        | 发送 DTMF 缓冲区中的数字数量。                  |                                                              |
| `tx_dtmf_buf`             | `struct dtmf[32]`                                            | 发送 DTMF 队列。                                |                                                              |
| `rx_event_pt`             | `int`                                                        | 接收 DTMF 的事件 PT。                           |                                                              |
| `last_dtmf`               | `int`                                                        | 当前数字，或 -1。                               |                                                              |
| `last_dtmf_dur`           | `pj_uint32_t`                                                | 当前数字的开始时间戳。                          |                                                              |
| `last_dtmf_ended`         | `pj_bool_t`                                                  | 上一个 DTMF 是否结束。                          |                                                              |
| `rx_dtmf_count`           | `unsigned`                                                   | 接收 DTMF 缓冲区中的数字数量。                  |                                                              |
| `rx_dtmf_buf`             | `char[32]`                                                   | 接收 DTMF 缓冲区。                              |                                                              |
| `dtmf_cb`                 | `void (*)(pjmedia_stream*, void*, int)`                      | DTMF 回调函数。                                 |                                                              |
| `dtmf_cb_user_data`       | `void*`                                                      | DTMF 回调函数的用户数据。                       |                                                              |
| `dtmf_event_cb`           | `void (*)(pjmedia_stream*, void*, const pjmedia_stream_dtmf_event*)` | DTMF 事件回调函数。                             |                                                              |
| `dtmf_event_cb_user_data` | `void*`                                                      | DTMF 事件回调函数的用户数据。                   |                                                              |
| `has_g722_mpeg_bug`       | `pj_bool_t`                                                  | 是否存在 G722 MPEG Bug。                        |                                                              |
| `rtp_tx_ts_len_per_pkt`   | `unsigned`                                                   | 每个发送包的标准化 TS 长度。                    |                                                              |
| `rtp_rx_ts_len_per_frame` | `unsigned`                                                   | 每个接收帧的标准化 TS 长度。                    |                                                              |
| `rtp_rx_last_cnt`         | `unsigned`                                                   | 上一个包中的帧数。                              |                                                              |
| `rtp_rx_check_cnt`        | `unsigned`                                                   | 远程时间戳检查计数器。                          |                                                              |
| `rtcp_xr_last_tx`         | `pj_uint32_t`                                                | 上次发送 RTCP XR 的时间戳。                     |                                                              |
| `rtcp_xr_interval`        | `pj_uint32_t`                                                | RTCP XR 的间隔时间戳。                          |                                                              |
| `rtcp_xr_dest`            | `pj_sockaddr`                                                | 附加的远程 RTCP XR 目标。                       |                                                              |
| `rtcp_xr_dest_len`        | `unsigned`                                                   | RTCP XR 目标地址的长度。                        |                                                              |
| `use_ka`                  | `pj_bool_t`                                                  | 是否启用了流的保活机制。                        |                                                              |
| `ka_interval`             | `unsigned`                                                   | 发送保活的间隔。                                |                                                              |
| `last_frm_ts_sent`        | `pj_time_val`                                                | 上次发送包的时间。                              |                                                              |
| `start_ka_count`          | `unsigned`                                                   | 创建后要发送的保活数量。                        |                                                              |
| `start_ka_interval`       | `unsigned`                                                   | 流创建后的保活发送间隔。                        |                                                              |
| `rem_rtp_addr`            | `pj_sockaddr`                                                | 远程 RTP 地址。                                 |                                                              |
| `rem_rtp_flag`            | `unsigned`                                                   | 来自该地址的数据包指示标志。                    |                                                              |
| `rtp_src_cnt`             | `unsigned`                                                   | 来自该地址的数据包数量。                        |                                                              |
| `trace_jb_fd`             | `pj_oshandle_t`                                              | 抖动跟踪文件句柄。                              |                                                              |
| `trace_jb_buf`            | `char*`                                                      | 抖动跟踪缓冲区。                                |                                                              |
| `rtp_rx_last_ts`          | `pj_uint32_t`                                                | 上次接收的 RTP 时间戳。                         |                                                              |
| `rtp_tx_err_cnt`          | `pj_uint32_t`                                                | RTP 发送错误计数。                              |                                                              |
| `rtcp_tx_err_cnt`         | `pj_uint32_t`                                                | RTCP 发送错误计数。                             |                                                              |
| `send_rtcp_fb_nack`       | `pj_bool_t`                                                  | 是否发送 RTCP 反馈 NACK？                       |                                                              |
| `rtcp_fb_nack`            | `pjmedia_rtcp_fb_nack`                                       | TX NACK 状态。                                  |                                                              |
| `rtcp_fb_nack_cap_idx`    | `int`                                                        | RX NACK 能力索引。                              |                                                              |

```c
    stream->endpt = endpt;
    stream->codec_mgr = pjmedia_endpt_get_codec_mgr(endpt);
    stream->dir = info->dir;
    stream->user_data = user_data;
    stream->rtcp_interval = (PJMEDIA_RTCP_INTERVAL-500 + (pj_rand()%1000)) *
                            info->fmt.clock_rate / 1000;
    stream->rtcp_sdes_bye_disabled = info->rtcp_sdes_bye_disabled;

    stream->tx_event_pt = info->tx_event_pt ? info->tx_event_pt : -1;
    stream->rx_event_pt = info->rx_event_pt ? info->rx_event_pt : -1;
    stream->last_dtmf = -1;
    stream->jb_last_frm = PJMEDIA_JB_NORMAL_FRAME;
    stream->rtcp_fb_nack.pid = -1;
    stream->soft_start_cnt = PJMEDIA_STREAM_SOFT_START;
		stream->cname = info->cname;
```

Create mutex to protect jitter buffer:

codec： **Create and initialize codec**: pjmedia_codec_mgr_alloc_codec   **Get codec param:** pjmedia_codec_mgr_get_default_param **Init the codec** pjmedia_codec_init(stream->codec, pool); **Open the codec.**

pjmedia_codec_open(stream->codec, &stream->codec_param);

```c
    stream->dec_ptime = stream->codec_param.info.frm_ptime;
    stream->dec_ptime_denum = PJ_MAX(stream->codec_param.info.frm_ptime_denum,
                                     1);
    afd->bits_per_sample = 16;
    afd->frame_time_usec = stream->codec_param.info.frm_ptime *
                           stream->codec_param.setting.frm_per_pkt * 1000 /
                           stream->codec_param.info.frm_ptime_denum;
    stream->port.info.fmt.id = stream->codec_param.info.fmt_id;
```

重要stream对应port的回调

stream->port.put_frame = &put_frame;

stream->port.get_frame = &get_frame;



 /* Init jitter buffer parameters: */

/* Create jitter buffer */

```
status = pjmedia_jbuf_create(pool, &stream->port.info.name,
                                 stream->frame_size,
                                 stream->codec_param.info.frm_ptime,
                                 jb_max, &stream->jb);
```

Create decoder channel

Create encoder channel

```c
status = create_channel( pool, stream, PJMEDIA_DIR_ENCODING,
                             info->tx_pt, info, &stream->enc);
```



Init RTCP session:



Only attach transport when stream is ready.在attach之前初始化att_param 看两个回调

```c
    att_param.rtp_cb2 = &on_rx_rtp;
    att_param.rtcp_cb = &on_rx_rtcp;
```

```c
    stream->transport = tp;
    status = pjmedia_transport_attach2(tp, &att_param);
    if (status != PJ_SUCCESS)
        goto err_cleanup;
```

