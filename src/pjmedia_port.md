## pjmedia_port

媒体端口（用pjmedia_port“类”表示）提供了一个通用和可扩展的框架，用于实现媒体元素。媒体元素本身可以是媒体源、接收端或处理元素。媒体端口接口基本上具有以下属性：

- 媒体端口信息（pjmedia_port_info）用于描述媒体端口的属性（采样率、通道数等），
- 可选的指向从端口获取帧的函数指针（get_frame()接口），将被pjmedia_port_get_frame()公共API调用，
- 以及可选的指向将帧存储到端口的函数指针（put_frame()接口），将被pjmedia_port_put_frame()公共API调用。



get_frame()和put_frame()接口当然只需要在媒体端口发出和/或接收媒体帧时才需要实现。

媒体端口是被动的“对象”。默认情况下，没有工作线程来运行媒体流。应用程序（或其他PJMEDIA组件，如时钟/定时中所述）必须积极地从媒体端口调用pjmedia_port_get_frame()或pjmedia_port_put_frame()来检索/存储媒体帧。

一些媒体端口（如会议桥和重采样端口）可能与其他端口相互连接（或封装），以执行端口的组合任务，而另一些代表媒体的最终源/汇终止。互连意味着上游媒体端口将调用其下游媒体端口的get_frame()和put_frame()。为了实现这一点，媒体端口需要具有相同的格式，其中格式被定义为音频媒体的采样格式、时钟速率、通道数、每样本位数和每帧样本数的组合。

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

## pjmedia_port_info

```c
typedef struct pjmedia_port_info
{
    pj_str_t        name;               /**< Port name.                     */
    pj_uint32_t     signature;          /**< Port signature.                */
    pjmedia_dir     dir;                /**< Port direction.                */
    pjmedia_format  fmt;                /**< Format.                        */
} pjmedia_port_info;
```

