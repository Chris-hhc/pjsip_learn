## transaction

### Invite session data to be attached to transaction.

```c
/* Invite session data to be attached to transaction. */
struct tsx_inv_data
{
    pjsip_inv_session   *inv;       /* The invite session                   */
    pj_bool_t            sdp_done;  /* SDP negotiation done for this tsx?   */
    pj_bool_t            retrying;  /* Resend (e.g. due to 401/407)         */
    pj_str_t             done_tag;  /* To tag in RX response with answer    */
    pj_bool_t            done_early;/* Negotiation was done for early med?  */
    pj_bool_t            done_early_rel;/* Early med was realiable?         */
    pj_bool_t            has_sdp;   /* Message with SDP?                    */
};

```

### pjsip_transaction

```c
/**
 * This structure describes SIP transaction object. The transaction object
 * is used to handle both UAS and UAC transaction.
 */
struct pjsip_transaction
{
    /*
     * Administrivia
     */
    pj_pool_t                  *pool;           /**< Pool owned by the tsx. */
    pjsip_module               *tsx_user;       /**< Transaction user.      */
    pjsip_endpoint             *endpt;          /**< Endpoint instance.     */
    pj_bool_t                   terminating;    /**< terminate() was called */
    pj_grp_lock_t              *grp_lock;       /**< Transaction grp lock.  */
    pj_mutex_t                 *mutex_b;        /**< Second mutex to avoid
                                                     deadlock. It is used to
                                                     protect timer.         */

    /*
     * Transaction identification.
     */
    char                        obj_name[PJ_MAX_OBJ_NAME];  /**< Log info.  */
    pjsip_role_e                role;           /**< Role (UAS or UAC)      */
    pjsip_method                method;         /**< The method.            */
    pj_int32_t                  cseq;           /**< The CSeq               */
    pj_str_t                    transaction_key;/**< Hash table key.        */
    pj_str_t                    transaction_key2;/**< Hash table key (2)   
                                                     for merged requests
                                                     tsx lookup.            */
    pj_uint32_t                 hashed_key;     /**< Key's hashed value.    */
    pj_uint32_t                 hashed_key2;    /**< Key's hashed value (2).*/
    pj_str_t                    branch;         /**< The branch Id.         */

    /*
     * State and status.
     */
    int                         status_code;    /**< Last status code seen. */
    pj_str_t                    status_text;    /**< Last reason phrase.    */
    pjsip_tsx_state_e           state;          /**< State.                 */
    int                         handle_200resp; /**< UAS 200/INVITE  retrsm.*/
    int                         tracing;        /**< Tracing enabled?       */

    /** Handler according to current state. */
    pj_status_t (*state_handler)(struct pjsip_transaction *, pjsip_event *);

    /*
     * Transport.
     */
    pjsip_transport            *transport;      /**< Transport to use.      */
    pj_bool_t                   is_reliable;    /**< Transport is reliable. */
    pj_sockaddr                 addr;           /**< Destination address.   */
    int                         addr_len;       /**< Address length.        */
    pjsip_response_addr         res_addr;       /**< Response address.      */
    unsigned                    transport_flag; /**< Miscelaneous flag.     */
    pj_status_t                 transport_err;  /**< Internal error code.   */
    pjsip_tpselector            tp_sel;         /**< Transport selector.    */
    pjsip_tx_data              *pending_tx;     /**< Tdata which caused
                                                     pending transport flag
                                                     to be set on tsx.      */
    pjsip_tp_state_listener_key *tp_st_key;     /**< Transport state listener
                                                     key.                   */

    /*
     * Messages and timer.
     */
    pjsip_tx_data              *last_tx;        /**< Msg kept for retrans.  */
    int                         retransmit_count;/**< Retransmission count. */
    pj_timer_entry              retransmit_timer;/**< Retransmit timer.     */
    pj_timer_entry              timeout_timer;  /**< Timeout timer.         */

    /** Module specific data. */
    void                       *mod_data[PJSIP_MAX_MODULE];
};
```



```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

