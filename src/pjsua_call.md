## pjsua_call

```c
/** 
 * Structure to be attached to invite dialog. 
 * Given a dialog "dlg", application can retrieve this structure
 * by accessing dlg->mod_data[pjsua.mod.id].
 */
struct pjsua_call
{
    unsigned             index;     /**< Index in pjsua array.              */
    pjsua_call_setting   opt;       /**< Call setting.                      */
    pj_bool_t            opt_inited;/**< Initial call setting has been set,
                                         to avoid different opt in answer.  */
    pjsip_inv_session   *inv;       /**< The invite session.                */
    void                *user_data; /**< User/application data.             */
    pjsip_status_code    last_code; /**< Last status code seen.             */
    pj_str_t             last_text; /**< Last status text seen.             */
    pj_time_val          start_time;/**< First INVITE sent/received.        */
    pj_time_val          res_time;  /**< First response sent/received.      */
    pj_time_val          conn_time; /**< Connected/confirmed time.          */
    pj_time_val          dis_time;  /**< Disconnect time.                   */
    pjsua_acc_id         acc_id;    /**< Account index being used.          */
    int                  secure_level;/**< Signaling security level.        */
    pjsua_call_hold_type call_hold_type; /**< How to do call hold.          */
    pj_bool_t            local_hold;/**< Flag for call-hold by local.       */
    void                *hold_msg;  /**< Outgoing hold tx_data.             */
    pj_str_t             cname;     /**< RTCP CNAME.                        */
    char                 cname_buf[16];/**< cname buffer.                   */

    unsigned             med_cnt;   /**< Number of media in SDP.            */
    pjsua_call_media     media[PJSUA_MAX_CALL_MEDIA]; /**< Array of media   */
    unsigned             med_prov_cnt;/**< Number of provisional media.     */
    pjsua_call_media     media_prov[PJSUA_MAX_CALL_MEDIA];
                                    /**< Array of provisional media.        */
    pj_bool_t            med_update_success;
                                    /**< Is media update successful?        */
    pj_bool_t            hanging_up;/**< Is call in the process of hangup?  */

    int                  audio_idx; /**< First active audio media.          */
    pj_mutex_t          *med_ch_mutex;/**< Media channel callback's mutex.  */
    pjsua_med_tp_state_cb   med_ch_cb;/**< Media channel callback.          */
    pjsua_med_tp_state_info med_ch_info;/**< Media channel info.            */

    pjsip_evsub         *xfer_sub;  /**< Xfer server subscription, if this
                                         call was triggered by xfer.        */
    pj_stun_nat_type     rem_nat_type; /**< NAT type of remote endpoint.    */

    char    last_text_buf_[128];    /**< Buffer for last_text.              */

    struct {
        int              retry_cnt;  /**< Retry count.                      */
    } lock_codec;                    /**< Data for codec locking when answer
                                          contains multiple codecs.         */

    struct {
        pjsip_dialog        *dlg;    /**< Call dialog.                      */
        pjmedia_sdp_session *rem_sdp;/**< Remote SDP.                       */
        pj_pool_t           *pool_prov;/**< Provisional pool.               */
        pj_bool_t            med_ch_deinit;/**< Media channel de-init-ed?   */
        union {
            struct {
                pjsua_msg_data  *msg_data;/**< Headers for outgoing INVITE. */
                pj_bool_t        hangup;  /**< Call is hangup?              */
            } out_call;
            struct {            
                call_answer      answers;/**< A list of call answers.       */
                pj_bool_t        hangup;/**< Call is hangup?                */
                pjsip_dialog    *replaced_dlg; /**< Replaced dialog.        */
            } inc_call;
        } call_var;
    } async_call;                      /**< Temporary storage for async
                                            outgoing/incoming call.         */

    pj_bool_t            rem_offerer;  /**< Was remote SDP offerer?         */
    unsigned             rem_aud_cnt;  /**< No of active audio in last remote
                                            offer.                          */
    unsigned             rem_vid_cnt;  /**< No of active video in last remote
                                            offer.                          */
    
    pj_bool_t            rx_reinv_async;/**< on_call_rx_reinvite() async.   */
    pj_timer_entry       reinv_timer;  /**< Reinvite retry timer.           */
    pj_bool_t            reinv_pending;/**< Pending until CONFIRMED state.  */
    pj_bool_t            reinv_ice_sent;/**< Has reinvite for ICE upd sent? */
    pjsip_rx_data       *incoming_data;/**< Cloned incoming call rdata.
                                            On pjsua2, when handling incoming 
                                            call, onCreateMediaTransport() will
                                            not be called since the call isn't
                                            created yet. This temporary 
                                            variable is used to handle such 
                                            case, see ticket #1916.         */

    struct {
        pj_bool_t        enabled;
        pj_bool_t        remote_sup;
        pj_bool_t        remote_dlg_est;
        pjsua_op_state   trickling;
        int              retrans18x_count;
        pj_bool_t        pending_info;
        pj_timer_entry   timer;
    } trickle_ice;

    pj_timer_entry       hangup_timer;  /**< Hangup retry timer.            */
    unsigned             hangup_retry;  /**< Number of hangup retries.      */
    unsigned             hangup_code;   /**< Hangup code.                   */
    pj_str_t             hangup_reason; /**< Hangup reason.                 */
    pjsua_msg_data      *hangup_msg_data;/**< Hangup message data.          */
};
```

### pjsua_call_hold_type

```c
/**
 * This enumeration specifies how we should offer call hold request to
 * remote peer. The default value is set by compile time constant
 * PJSUA_CALL_HOLD_TYPE_DEFAULT, and application may control the setting
 * on per-account basis by manipulating \a call_hold_type field in
 * #pjsua_acc_config.
 */
typedef enum pjsua_call_hold_type
{
    /**
     * This will follow RFC 3264 recommendation to use a=sendonly,
     * a=recvonly, and a=inactive attribute as means to signal call
     * hold status. This is the correct value to use.
     */
    PJSUA_CALL_HOLD_TYPE_RFC3264,

    /**
     * This will use the old and deprecated method as specified in RFC 2543,
     * and will offer c=0.0.0.0 in the SDP instead. Using this has many
     * drawbacks such as inability to keep the media transport alive while
     * the call is being put on hold, and should only be used if remote
     * does not understand RFC 3264 style call hold offer.
     */
    PJSUA_CALL_HOLD_TYPE_RFC2543

} pjsua_call_hold_type;
```

