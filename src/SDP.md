## SDP

[SDP协议简介](https://www.jianshu.com/p/94b118b8fd97)

### pjmedia_sdp_session

```c
/**
 * This structure describes SDP session description. A SDP session descriptor
 * contains complete information about a session, and normally is exchanged
 * with remote media peer using signaling protocol such as SIP.
 */
struct pjmedia_sdp_session
{
    /** Session origin (o= line) */
    struct
    {
        pj_str_t    user;           /**< User                           */
        pj_uint_t   id;             /**< Session ID                     */
        pj_uint_t   version;        /**< Session version                */
        pj_str_t    net_type;       /**< Network type ("IN")            */
        pj_str_t    addr_type;      /**< Address type ("IP4", "IP6")    */
        pj_str_t    addr;           /**< The address.                   */
    } origin;

    pj_str_t           name;        /**< Subject line (s=)              */
    pjmedia_sdp_conn  *conn;        /**< Connection line (c=)           */
    unsigned           bandw_count; /**< Number of bandwidth info (b=)  */
    pjmedia_sdp_bandw *bandw[PJMEDIA_MAX_SDP_BANDW];
                                    /**< Bandwidth info array (b=)      */
    
    /** Session time (t= line)  */
    struct
    {
        pj_uint_t start;            /**< Start time.                    */
        pj_uint_t stop;             /**< Stop time.                     */
    } time;

    unsigned           attr_count;              /**< Number of attributes.  */
    pjmedia_sdp_attr  *attr[PJMEDIA_MAX_SDP_ATTR]; /**< Attributes array.   */

    unsigned           media_count;             /**< Number of media.       */
    pjmedia_sdp_media *media[PJMEDIA_MAX_SDP_MEDIA];    /**< Media array.   */

};

```

### pjmedia_sdp_attr

```c
/** 
 * Generic representation of attribute.
 */
struct pjmedia_sdp_attr
{
    pj_str_t            name;       /**< Attribute name.    */
    pj_str_t            value;      /**< Attribute value.   */
};

```

### pjmedia_sdp_media

```c
/**
 * This structure describes SDP media descriptor. A SDP media descriptor
 * starts with "m=" line and contains the media attributes and optional
 * connection line.
 */
struct pjmedia_sdp_media
{
    /** Media descriptor line ("m=" line) */
    struct
    {
        pj_str_t    media;              /**< Media type ("audio", "video")  */
        pj_uint16_t port;               /**< Port number.                   */
        unsigned    port_count;         /**< Port count, used only when >2  */
        pj_str_t    transport;          /**< Transport ("RTP/AVP")          */
        unsigned    fmt_count;          /**< Number of formats.             */
        pj_str_t    fmt[PJMEDIA_MAX_SDP_FMT];       /**< Media formats.     */
    } desc;

    pjmedia_sdp_conn   *conn;           /**< Optional connection info.      */
    unsigned            bandw_count;    /**< Number of bandwidth info.      */
    pjmedia_sdp_bandw  *bandw[PJMEDIA_MAX_SDP_BANDW]; /**< Bandwidth info.  */
    unsigned            attr_count;     /**< Number of attributes.          */
    pjmedia_sdp_attr   *attr[PJMEDIA_MAX_SDP_ATTR];   /**< Attributes.      */

};
```

### pjmedia_sdp_conn

```c
/**
 * This structure describes SDP connection info ("c=" line). 
 */
struct pjmedia_sdp_conn
{
    pj_str_t    net_type;       /**< Network type ("IN").               */
    pj_str_t    addr_type;      /**< Address type ("IP4", "IP6").       */
    pj_str_t    addr;           /**< The address.                       */
    pj_uint8_t  ttl;            /**< Multicast address TTL              */
    pj_uint8_t  no_addr;        /**< Multicast number of addresses      */
};
```

### pjmedia_sdp_bandw

```c
/**
 * This structure describes SDP bandwidth info ("b=" line). 
 */
typedef struct pjmedia_sdp_bandw
{
    pj_str_t    modifier;       /**< Bandwidth modifier.                */
    pj_uint32_t value;          /**< Bandwidth value.                   */
} pjmedia_sdp_bandw;
```

