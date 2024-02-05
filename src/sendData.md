## Data structure for sending outgoing message

```c
/**
 * Data structure for sending outgoing message. Application normally creates
 * this buffer by calling #pjsip_endpt_create_tdata.
 *
 * The lifetime of this buffer is controlled by the reference counter in this
 * structure, which is manipulated by calling #pjsip_tx_data_add_ref and
 * #pjsip_tx_data_dec_ref. When the reference counter has reached zero, then
 * this buffer will be destroyed.
 *
 * A transaction object normally will add reference counter to this buffer
 * when application calls #pjsip_tsx_send_msg, because it needs to keep the
 * message for retransmission. The transaction will release the reference
 * counter once its state has reached final state.
 */
struct pjsip_tx_data
{
    /** This is for transmission queue; it's managed by transports. */
    PJ_DECL_LIST_MEMBER(struct pjsip_tx_data);

    /** Memory pool for this buffer. */
    pj_pool_t           *pool;

    /** A name to identify this buffer. */
    char                 obj_name[PJ_MAX_OBJ_NAME];

    /** Short information describing this buffer and the message in it. 
     *  Application should use #pjsip_tx_data_get_info() instead of
     *  directly accessing this member.
     */
    char                *info;

    /** For response message, this contains the reference to timestamp when 
     *  the original request message was received. The value of this field
     *  is set when application creates response message to a request by
     *  calling #pjsip_endpt_create_response.
     */
    pj_time_val          rx_timestamp;

    /** The transport manager for this buffer. */
    pjsip_tpmgr         *mgr;

    /** Ioqueue asynchronous operation key. */
    pjsip_tx_data_op_key op_key;

    /** Lock object. */
    pj_lock_t           *lock;

    /** The message in this buffer. */
    pjsip_msg           *msg;

    /** Strict route header saved by #pjsip_process_route_set(), to be
     *  restored by #pjsip_restore_strict_route_set().
     */
    pjsip_route_hdr     *saved_strict_route;

    /** Buffer to the printed text representation of the message. When the
     *  content of this buffer is set, then the transport will send the content
     *  of this buffer instead of re-printing the message structure. If the
     *  message structure has changed, then application must invalidate this
     *  buffer by calling #pjsip_tx_data_invalidate_msg.
     */
    pjsip_buffer         buf;

    /** Reference counter. */
    pj_atomic_t         *ref_cnt;

    /** Being processed by transport? */
    int                  is_pending;

    /** Transport manager internal. */
    void                *token;

    /** Callback to be called when this tx_data has been transmitted.   */
    void               (*cb)(void*, pjsip_tx_data*, pj_ssize_t);

    /** Destination information, to be used to determine the network address
     *  of the message. For a request, this information is  initialized when
     *  the request is sent with #pjsip_endpt_send_request_stateless() and
     *  network address is resolved. For CANCEL request, this information
     *  will be copied from the original INVITE to make sure that the CANCEL
     *  request goes to the same physical network address as the INVITE
     *  request.
     */
    struct
    {
        /** Server name. 
         */
        pj_str_t                 name;

        /** Server addresses resolved. 
         */
        pjsip_server_addresses   addr;

        /** Current server address being tried. 
         */
        unsigned cur_addr;

    } dest_info;

    /** Transport information, only valid during on_tx_request() and 
     *  on_tx_response() callback.
     */
    struct
    {
        pjsip_transport     *transport;     /**< Transport being used.  */
        pj_sockaddr          dst_addr;      /**< Destination address.   */
        int                  dst_addr_len;  /**< Length of address.     */
        char                 dst_name[PJ_INET6_ADDRSTRLEN]; /**< Destination address.   */
        int                  dst_port;      /**< Destination port.      */
    } tp_info;

    /** 
     * Transport selector, to specify which transport to be used. 
     * The value here must be set with pjsip_tx_data_set_transport(),
     * to allow reference counter to be set properly.
     */
    pjsip_tpselector        tp_sel;

    /**
     * Special flag to indicate that this transmit data is a request that has
     * been updated with proper authentication response and is ready to be
     * sent for retry.
     */
    pj_bool_t               auth_retry;

    /**
     * Arbitrary data attached by PJSIP modules.
     */
    void                    *mod_data[PJSIP_MAX_MODULE];

    /**
     * If via_addr is set, it will be used as the "sent-by" field of the
     * Via header for outgoing requests as long as the request uses via_tp
     * transport. Normally application should not use or access these fields.
     */
    pjsip_host_port          via_addr;      /**< Via address.           */
    const void              *via_tp;        /**< Via transport.         */
};
```

