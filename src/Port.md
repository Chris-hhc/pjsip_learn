# Port & Audio

## 结构体

### pjmedia_port

```c

/**
 * Port interface.
 */
typedef struct pjmedia_port
{
    pjmedia_port_info    info;              /**< Port information.  */

    /** Port data can be used by the port creator to attach arbitrary
     *  value to be associated with the port.
     */
    struct port_data {
        void            *pdata;             /**< Pointer data.      */
        long             ldata;             /**< Long data.         */
    } port_data;

    /**
     * Group lock.
     *
     * This is optional, but if this port is registered to the audio/video
     * conference bridge, the bridge will create one if the port has none.
     */
    pj_grp_lock_t       *grp_lock;

    /**
     * Get clock source.
     * This should only be called by #pjmedia_port_get_clock_src().
     */
    pjmedia_clock_src* (*get_clock_src)(struct pjmedia_port *this_port,
                                        pjmedia_dir dir);

    /**
     * Sink interface. 
     * This should only be called by #pjmedia_port_put_frame().
     */
    pj_status_t (*put_frame)(struct pjmedia_port *this_port, 
                             pjmedia_frame *frame);

    /**
     * Source interface. 
     * This should only be called by #pjmedia_port_get_frame().
     */
    pj_status_t (*get_frame)(struct pjmedia_port *this_port, 
                             pjmedia_frame *frame);

    /**
     * Called to destroy this port.
     */
    pj_status_t (*on_destroy)(struct pjmedia_port *this_port);

} pjmedia_port;
```

### pjmedia_snd_port

```c
struct pjmedia_snd_port
{
    int			 rec_id;
    int			 play_id;
    pj_uint32_t		 aud_caps;
    pjmedia_aud_param	 aud_param;
    pjmedia_aud_stream	*aud_stream;
    pjmedia_dir		 dir;
    pjmedia_port	*port;
 
    pjmedia_clock_src    cap_clocksrc,
                         play_clocksrc;
 
    unsigned		 clock_rate;
    unsigned		 channel_count;
    unsigned		 samples_per_frame;
    unsigned		 bits_per_sample;
    unsigned		 options;
    unsigned		 prm_ec_options;
 
 
    /* audio frame preview callbacks */
    void		*user_data;
    pjmedia_aud_play_cb  on_play_frame;
    pjmedia_aud_rec_cb   on_rec_frame;
};
```

### pjmedia_snd_port_param

```c
/**
 * This structure specifies the parameters to create the sound port.
 * Use pjmedia_snd_port_param_default() to initialize this structure with
 * default values (mostly zeroes)
 */
typedef struct pjmedia_snd_port_param
{
    /**
     * Base structure.
     */
    pjmedia_aud_param base;
    
    /**
     * Sound port creation options.
     */
    unsigned options;

    /**
     * Echo cancellation options/flags.
     */
    unsigned ec_options;

    /**
     * Arbitrary user data for playback and record preview callbacks below.
     */
    void *user_data;

    /**
     * Optional callback for audio frame preview right before queued to
     * the speaker.
     * Notes:
     * - application MUST NOT block or perform long operation in the callback
     *   as the callback may be executed in sound device thread
     * - when using software echo cancellation, application MUST NOT modify
     *   the audio data from within the callback, otherwise the echo canceller
     *   will not work properly.
     * - the return value of the callback will be ignored
     */
    pjmedia_aud_play_cb on_play_frame;

    /**
     * Optional callback for audio frame preview recorded from the microphone
     * before being processed by any media component such as software echo
     * canceller.
     * Notes:
     * - application MUST NOT block or perform long operation in the callback
     *   as the callback may be executed in sound device thread
     * - when using software echo cancellation, application MUST NOT modify
     *   the audio data from within the callback, otherwise the echo canceller
     *   will not work properly.
     * - the return value of the callback will be ignored
     */
    pjmedia_aud_rec_cb on_rec_frame;

} pjmedia_snd_port_param;
```

### pjmedia_aud_param

```c
/**
 * This structure specifies the parameters to open the audio stream.
 */
typedef struct pjmedia_aud_param
{
    /**
     * The audio direction. This setting is mandatory.
     */
    pjmedia_dir dir;

    /**
     * The audio recorder device ID. This setting is mandatory if the audio
     * direction includes input/capture direction.
     */
    pjmedia_aud_dev_index rec_id;

    /**
     * The audio playback device ID. This setting is mandatory if the audio
     * direction includes output/playback direction.
     */
    pjmedia_aud_dev_index play_id;

    /** 
     * Clock rate/sampling rate. This setting is mandatory. 
     */
    unsigned clock_rate;

    /** 
     * Number of channels. This setting is mandatory. 
     */
    unsigned channel_count;

    /** 
     * Number of samples per frame. This setting is mandatory. 
     */
    unsigned samples_per_frame;

    /** 
     * Number of bits per sample. This setting is mandatory. 
     */
    unsigned bits_per_sample;

    /** 
     * This flags specifies which of the optional settings are valid in this
     * structure. The flags is bitmask combination of pjmedia_aud_dev_cap.
     */
    unsigned flags;

    /** 
     * Set the audio format. This setting is optional, and will only be used
     * if PJMEDIA_AUD_DEV_CAP_EXT_FORMAT is set in the flags.
     */
    pjmedia_format ext_fmt;

    /**
     * Input latency, in milliseconds. This setting is optional, and will 
     * only be used if PJMEDIA_AUD_DEV_CAP_INPUT_LATENCY is set in the flags.
     */
    unsigned input_latency_ms;

    /**
     * Input latency, in milliseconds. This setting is optional, and will 
     * only be used if PJMEDIA_AUD_DEV_CAP_OUTPUT_LATENCY is set in the flags.
     */
    unsigned output_latency_ms;

    /**
     * Input volume setting, in percent. This setting is optional, and will 
     * only be used if PJMEDIA_AUD_DEV_CAP_INPUT_VOLUME_SETTING is set in 
     * the flags.
     */
    unsigned input_vol;

    /**
     * Output volume setting, in percent. This setting is optional, and will 
     * only be used if PJMEDIA_AUD_DEV_CAP_OUTPUT_VOLUME_SETTING is set in 
     * the flags.
     */
    unsigned output_vol;

    /** 
     * Set the audio input route/source. This setting is optional, and
     * will only be used if PJMEDIA_AUD_DEV_CAP_INPUT_ROUTE/
     * PJMEDIA_AUD_DEV_CAP_INPUT_SOURCE is set in the flags.
     */
    pjmedia_aud_dev_route input_route;

    /** 
     * Set the audio output route. This setting is optional, and will only be
     * used if PJMEDIA_AUD_DEV_CAP_OUTPUT_ROUTE is set in the flags.
     */
    pjmedia_aud_dev_route output_route;

    /**
     * Enable/disable echo canceller, if the device supports it. This setting
     * is optional, and will only be used if PJMEDIA_AUD_DEV_CAP_EC is set in
     * the flags.
     */
    pj_bool_t ec_enabled;

    /**
     * Set echo canceller tail length in milliseconds, if the device supports
     * it. This setting is optional, and will only be used if
     * PJMEDIA_AUD_DEV_CAP_EC_TAIL is set in the flags.
     */
    unsigned ec_tail_ms;

    /** 
     * Enable/disable PLC. This setting is optional, and will only be used
     * if PJMEDIA_AUD_DEV_CAP_PLC is set in the flags.
     */
    pj_bool_t plc_enabled;

    /** 
     * Enable/disable CNG. This setting is optional, and will only be used
     * if PJMEDIA_AUD_DEV_CAP_CNG is set in the flags.
     */
    pj_bool_t cng_enabled;

    /** 
     * Enable/disable VAD. This setting is optional, and will only be used
     * if PJMEDIA_AUD_DEV_CAP_VAD is set in the flags.
     */
    pj_bool_t vad_enabled;

} pjmedia_aud_param;
```

### 设备抽象

#### pjmedia_aud_stream 在设备的进一步抽象中绑定rec_cb play_cb

```c
/**
 * This structure describes the audio device stream.
 */
struct pjmedia_aud_stream
{
    /** Internal data to be initialized by audio subsystem */
    struct {
        /** Driver index */
        unsigned drv_idx;
    } sys;

    /** Operations */
    pjmedia_aud_stream_op *op;
};

```

#### pjmedia_aud_stream_op

```c
/**
 * Sound stream operations.
 */
typedef struct pjmedia_aud_stream_op
{
    /**
     * See #pjmedia_aud_stream_get_param()
     */
    pj_status_t (*get_param)(pjmedia_aud_stream *strm,
                             pjmedia_aud_param *param);

    /**
     * See #pjmedia_aud_stream_get_cap()
     */
    pj_status_t (*get_cap)(pjmedia_aud_stream *strm,
                           pjmedia_aud_dev_cap cap,
                           void *value);

    /**
     * See #pjmedia_aud_stream_set_cap()
     */
    pj_status_t (*set_cap)(pjmedia_aud_stream *strm,
                           pjmedia_aud_dev_cap cap,
                           const void *value);

    /**
     * See #pjmedia_aud_stream_start()
     */
    pj_status_t (*start)(pjmedia_aud_stream *strm);

    /**
     * See #pjmedia_aud_stream_stop().
     */
    pj_status_t (*stop)(pjmedia_aud_stream *strm);

    /**
     * See #pjmedia_aud_stream_destroy().
     */
    pj_status_t (*destroy)(pjmedia_aud_stream *strm);

} pjmedia_aud_stream_op;
```

#### factory相关

```c

/**
 * Sound device factory operations.
 */
typedef struct pjmedia_aud_dev_factory_op
{
    /**
     * Initialize the audio device factory.
     *
     * @param f         The audio device factory.
     */
    pj_status_t (*init)(pjmedia_aud_dev_factory *f);

    /**
     * Close this audio device factory and release all resources back to the
     * operating system.
     *
     * @param f         The audio device factory.
     */
    pj_status_t (*destroy)(pjmedia_aud_dev_factory *f);

    /**
     * Get the number of audio devices installed in the system.
     *
     * @param f         The audio device factory.
     */
    unsigned (*get_dev_count)(pjmedia_aud_dev_factory *f);

    /**
     * Get the audio device information and capabilities.
     *
     * @param f         The audio device factory.
     * @param index     Device index.
     * @param info      The audio device information structure which will be
     *                  initialized by this function once it returns 
     *                  successfully.
     */
    pj_status_t (*get_dev_info)(pjmedia_aud_dev_factory *f, 
                                unsigned index,
                                pjmedia_aud_dev_info *info);

    /**
     * Initialize the specified audio device parameter with the default
     * values for the specified device.
     *
     * @param f         The audio device factory.
     * @param index     Device index.
     * @param param     The audio device parameter.
     */
    pj_status_t (*default_param)(pjmedia_aud_dev_factory *f,
                                 unsigned index,
                                 pjmedia_aud_param *param);

    /**
     * Open the audio device and create audio stream. See
     * #pjmedia_aud_stream_create()
     */
    pj_status_t (*create_stream)(pjmedia_aud_dev_factory *f,
                                 const pjmedia_aud_param *param,
                                 pjmedia_aud_rec_cb rec_cb,
                                 pjmedia_aud_play_cb play_cb,
                                 void *user_data,
                                 pjmedia_aud_stream **p_aud_strm);

    /**
     * Refresh the list of audio devices installed in the system.
     *
     * @param f         The audio device factory.
     */
    pj_status_t (*refresh)(pjmedia_aud_dev_factory *f);

} pjmedia_aud_dev_factory_op;


/**
 * This structure describes an audio device factory. 
 */
struct pjmedia_aud_dev_factory
{
    /** Internal data to be initialized by audio subsystem. */
    struct {
        /** Driver index */
        unsigned drv_idx;
    } sys;

    /** Operations */
    pjmedia_aud_dev_factory_op *op;
};
```



## 相关方法

### 初始化方法

#### （snd_port_param）pjmedia_snd_port_param_default

```c
/* Initialize with default values (zero) */
PJ_DEF(void) pjmedia_snd_port_param_default(pjmedia_snd_port_param *prm)
{
    pj_bzero(prm, sizeof(*prm));
}
```

#### pjmedia_stream_get_port

```
PJ_DEF(pj_status_t) pjmedia_stream_get_port( pjmedia_stream *stream,
                                             pjmedia_port **p_port )
{
    *p_port = &stream->port;
    return PJ_SUCCESS;
}
```

stream->port，返回p_port

#### （aud_dev_default_param）pjmedia_aud_dev_default_param

```c
/* API: Initialize the audio device parameters with default values for the
 * specified device.
 */
PJ_DEF(pj_status_t) pjmedia_aud_dev_default_param(pjmedia_aud_dev_index id,
                                                  pjmedia_aud_param *param)
{
    pjmedia_aud_dev_factory *f;
    unsigned index;
    pj_status_t status;

    PJ_ASSERT_RETURN(param && id!=PJMEDIA_AUD_INVALID_DEV, PJ_EINVAL);
    PJ_ASSERT_RETURN(aud_subsys.pf, PJMEDIA_EAUD_INIT);

    status = lookup_dev(id, &f, &index);  //Internal: lookup valid device id
    if (status != PJ_SUCCESS)
        return status;

    status = f->op->default_param(f, index, param);
    if (status != PJ_SUCCESS)
        return status;

    /* Normalize device IDs */
    make_global_index(f->sys.drv_idx, &param->rec_id);
    make_global_index(f->sys.drv_idx, &param->play_id);

    return PJ_SUCCESS;
}
```



### Create port

#### pjmedia_snd_port_create

初始化pjmedia_snd_port_param的各种信息，调用pjmedia_snd_port_create2

```c
pj_status_t pjmedia_snd_port_create_player(pj_pool_t *pool, int dev_id, unsigned int clock_rate, unsigned int channel_count, unsigned int samples_per_frame, unsigned int bits_per_sample, unsigned int options, pjmedia_snd_port **p_port)
```

Create unidirectional sound device port for playing audio streams with the specified parameters.

**参数:**
`pool` – Pool to allocate sound port structure.
`index` – Device index, or -1 to let the library choose the first available device.
`clock_rate` – Sound device's clock rate to set.
`channel_count` – Set number of channels, 1 for mono, or 2 for stereo. The channel count determines the format of the frame.
`samples_per_frame` – Number of samples per frame.
`bits_per_sample` – Set the number of bits per sample. The normal value for this parameter is 16 bits per sample.
`options` – Options flag.
`p_port` – Pointer to receive the sound device port instance.

#### pjmedia_snd_port_create2

用pjmedia_snd_port_creat中传入的pjmedia_snd_port_param，初始化pjmedia_snd_port（此时声音port创建），最后调用start_sound_device，Start the sound stream.

#### start_sound_device

- Get device caps：获取dev_id，根据id调用函数pjmedia_aud_dev_get_info获取pjmedia_aud_dev_info

- Process EC settings 

- Open the device

  设置了两个回调snd_rec_cb、snd_play_cb分别为 rec_cb、play_cb

  ```c
      status = pjmedia_aud_stream_create(&param_copy,
                                         snd_rec_cb,
                                         snd_play_cb,
                                         snd_port,
                                         &snd_port->aud_stream);
  ```

- Start sound stream.：pjmedia_aud_stream_start(snd_port->aud_stream);

#### pjmedia_aud_stream_create

先通过lookup_dev搜索rec_id、play_id设备对应的工厂，然后通过工厂创建设备f->op->create_stream并设置给设备两个回调函数 rec_cb、play_cb给设备抽象pjmedia_aud_stream的ca_cb、pb_cb，lookup_dev中aud_subsys已经存储了所有的音频设备是在创建媒体端点endpoint时初始化举例来说：初始化的时候创建媒体端点pjmedia_endpt，同时初始化了音频子系统aud_subsys，把各种类型的音频设备工厂添加到全局变量[static](https://so.csdn.net/so/search?q=static&spm=1001.2101.3001.7020) pjmedia_aud_subsys aud_subsys;。这样当创建设备时，就可以遍历这些工厂，寻找合适的工厂，通过工厂创建设备实例。比如alsa类型的设备在alsa_dev.c

#### pjmedia_aud_stream_start

以alsa为例，在alsa_dev.c中，调用alsa_stream_start函数，在alsa_dev.c alsa_stream_start中，会创建播放和采集两条线程。

```c
static pj_status_t alsa_stream_start (pjmedia_aud_stream *s)
{
    struct alsa_stream *stream = (struct alsa_stream*)s;
    pj_status_t status = PJ_SUCCESS;
 
    stream->quit = 0;
    if (stream->param.dir & PJMEDIA_DIR_PLAYBACK) {
	status = pj_thread_create (stream->pool,
				   "alsasound_playback",
				   pb_thread_func,
				   stream,
				   0, //ZERO,
				   0,
				   &stream->pb_thread);
 
 
    if (stream->param.dir & PJMEDIA_DIR_CAPTURE) {
	status = pj_thread_create (stream->pool,
				   "alsasound_playback",
				   ca_thread_func,
				   stream,
				   0, //ZERO,
				   0,
				   &stream->ca_thread);
 
    }
 
    return status;
}
```

以播放线程为例

```c
static int pb_thread_func (void *arg)
{
    struct alsa_stream* stream = (struct alsa_stream*) arg;
    snd_pcm_t* pcm             = stream->pb_pcm;
    int size                   = stream->pb_buf_size;
    snd_pcm_uframes_t nframes  = stream->pb_frames;
    void* user_data            = stream->user_data;
    char* buf 		       = stream->pb_buf;
    pj_timestamp tstamp;
    int result;
 
    pj_bzero (buf, size);
    tstamp.u64 = 0;
 
 
    snd_pcm_prepare (pcm);
 
    while (!stream->quit) {
        pjmedia_frame frame;

        frame.type = PJMEDIA_FRAME_TYPE_AUDIO;
        frame.buf = buf;
        frame.size = size;
        frame.timestamp.u64 = tstamp.u64;
        frame.bit_info = 0;

        result = stream->pb_cb (user_data, &frame);
        if (result != PJ_SUCCESS || stream->quit)
            break;

        if (frame.type != PJMEDIA_FRAME_TYPE_AUDIO)
            pj_bzero (buf, size);

        result = snd_pcm_writei (pcm, buf, nframes);
        if (result == -EPIPE) {
            PJ_LOG (4,(THIS_FILE, "pb_thread_func: underrun!"));
            snd_pcm_prepare (pcm);
        } else if (result < 0) {
            PJ_LOG (4,(THIS_FILE, "pb_thread_func: error writing data!"));
        }

        tstamp.u64 += nframes;
    }
 
    snd_pcm_drain (pcm);
    TRACE_((THIS_FILE, "pb_thread_func: Stopped"));
    return PJ_SUCCESS;
}
```

播放线程先通过回调拿到待播放的音频数据stream->pb_cb ，然后写到声卡snd_pcm_writei。pb_cb就是sound_port.c中的play_cb，来看下play_cb的流程。

```c
static pj_status_t play_cb(void *user_data, pjmedia_frame *frame)
{
    pjmedia_snd_port *snd_port = (pjmedia_snd_port*) user_data;
    pjmedia_port *port;
    const unsigned required_size = (unsigned)frame->size;
    pj_status_t status;
 
    port = snd_port->port;
    status = pjmedia_port_get_frame(port, frame);
 
    /* Invoke preview callback */
    if (snd_port->on_play_frame)
	(*snd_port->on_play_frame)(snd_port->user_data, frame);
 
    return PJ_SUCCESS;
}
```

通过pjmedia_port* port获取一帧数据

