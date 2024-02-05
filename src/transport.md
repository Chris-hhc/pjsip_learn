## transport

```c
/**
 * This structure represent the "public" interface of a SIP transport.
 * Applications normally extend this structure to include transport
 * specific members.
 */
struct pjsip_transport
{
    char                    obj_name[PJ_MAX_OBJ_NAME];  /**< Name. */

    pj_pool_t              *pool;           /**< Pool used by transport.    */
    pj_atomic_t            *ref_cnt;        /**< Reference counter.         */
    pj_lock_t              *lock;           /**< Lock object.               */
    pj_grp_lock_t          *grp_lock;       /**< Group lock for sync with
                                                 ioqueue and timer.         */
    pj_bool_t               tracing;        /**< Tracing enabled?           */
    pj_bool_t               is_shutdown;    /**< Being shutdown?            */
    pj_bool_t               is_destroying;  /**< Destroy in progress?       */

    /** Key for indexing this transport in hash table. */
    pjsip_transport_key     key;

    char                   *type_name;      /**< Type name.                 */
    unsigned                flag;           /**< #pjsip_transport_flags_e   */
    char                   *info;           /**< Transport info/description.*/

    int                     addr_len;       /**< Length of addresses.       */
    pj_sockaddr             local_addr;     /**< Bound address.             */
    pjsip_host_port         local_name;     /**< Published name (eg. STUN). */
    pjsip_host_port         remote_name;    /**< Remote address name.       */
    pjsip_transport_dir     dir;            /**< Connection direction.      */
    
    pjsip_endpoint         *endpt;          /**< Endpoint instance.         */
    pjsip_tpmgr            *tpmgr;          /**< Transport manager.         */
    pjsip_tpfactory        *factory;        /**< Factory instance. Note: it
                                                 may be invalid/shutdown.   */
    pj_timer_entry          idle_timer;     /**< Timer when ref cnt is zero.*/

    pj_timestamp            last_recv_ts;   /**< Last time receiving data.  */
    pj_size_t               last_recv_len;  /**< Last received data length. */

    void                   *data;           /**< Internal transport data.   */
    unsigned                initial_timeout;/**< Initial timeout interval
                                                 to be applied to incoming
                                                 TCP/TLS transports when no
                                                 valid data received after
                                                 a successful connection.   */

    /**
     * Function to be called by transport manager to send SIP message.
     *
     * @param transport     The transport to send the message.
     * @param packet        The buffer to send.
     * @param length        The length of the buffer to send.
     * @param op_key        Completion token, which will be supplied to
     *                      caller when pending send operation completes.
     * @param rem_addr      The remote destination address.
     * @param addr_len      Size of remote address.
     * @param callback      If supplied, the callback will be called
     *                      once a pending transmission has completed. If
     *                      the function completes immediately (i.e. return
     *                      code is not PJ_EPENDING), the callback will not
     *                      be called.
     *
     * @return              Should return PJ_SUCCESS only if data has been
     *                      succesfully queued to operating system for 
     *                      transmission. Otherwise it may return PJ_EPENDING
     *                      if the underlying transport can not send the
     *                      data immediately and will send it later, which in
     *                      this case caller doesn't have to do anything 
     *                      except wait the calback to be called, if it 
     *                      supplies one.
     *                      Other return values indicate the error code.
     */
    pj_status_t (*send_msg)(pjsip_transport *transport, 
                            pjsip_tx_data *tdata,
                            const pj_sockaddr_t *rem_addr,
                            int addr_len,
                            void *token,
                            pjsip_transport_callback callback);

    /**
     * Instruct the transport to initiate graceful shutdown procedure.
     * After all objects release their reference to this transport,
     * the transport will be deleted.
     *
     * Note that application MUST use #pjsip_transport_shutdown() instead.
     *
     * @param transport     The transport.
     *
     * @return              PJ_SUCCESS on success.
     */
    pj_status_t (*do_shutdown)(pjsip_transport *transport);

    /**
     * Forcefully destroy this transport regardless whether there are
     * objects that currently use this transport. This function should only
     * be called by transport manager or other internal objects (such as the
     * transport itself) who know what they're doing. Application should use
     * #pjsip_transport_shutdown() instead.
     *
     * @param transport     The transport.
     *
     * @return              PJ_SUCCESS on success.
     */
    pj_status_t (*destroy)(pjsip_transport *transport);

    /*
     * Application may extend this structure..
     */
};
```

