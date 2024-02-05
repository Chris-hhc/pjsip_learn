## Session

### invite session

```c
/**
 * This structure describes the invite session.
 *
 * Note regarding the invite session's pools. The inv_sess used to have
 * only one pool, which is just a pointer to the dialog's pool. Ticket
 * https://github.com/pjsip/pjproject/issues/877 has found that the memory
 * usage will grow considerably everytime re-INVITE or UPDATE is
 * performed.
 *
 * Ticket #877 then created two more memory pools for the inv_sess, so
 * now we have three memory pools:
 *  - pool: to be used to allocate long term data for the session
 *  - pool_prov and pool_active: this is a flip-flop pools to be used
 *     interchangably during re-INVITE and UPDATE. pool_prov is
 *     "provisional" pool, used to allocate SDP offer or answer for
 *     the re-INVITE and UPDATE. Once SDP negotiation is done, the
 *     provisional pool will be made as the active pool, then the
 *     existing active pool will be reset, to release the memory
 *     back to the OS. So these pool's lifetime is synchronized to
 *     the SDP offer-answer negotiation.
 *
 * Higher level application such as PJSUA-LIB has been modified to
 * make use of these flip-flop pools, i.e. by creating media objects
 * from the provisional pool rather than from the long term pool.
 *
 * Other applications that want to use these pools must understand
 * that the flip-flop pool's lifetimes are synchronized to the
 * SDP offer-answer negotiation.
 *
 * The lifetime of this session is controlled by the reference counter in this
 * structure, which is manipulated by calling #pjsip_inv_add_ref and
 * #pjsip_inv_dec_ref. When the reference counter has reached zero, then
 * this session will be destroyed.
 */
struct pjsip_inv_session
{
    char                 obj_name[PJ_MAX_OBJ_NAME]; /**< Log identification */
    pj_pool_t           *pool;                      /**< Long term pool.    */
    pj_pool_t           *pool_prov;                 /**< Provisional pool   */
    pj_pool_t           *pool_active;               /**< Active/current pool*/
    pjsip_inv_state      state;                     /**< Invite sess state. */
    pj_bool_t            cancelling;                /**< CANCEL requested   */
    pj_bool_t            pending_cancel;            /**< Wait to send CANCEL*/
    pjsip_tx_data       *pending_bye;               /**< BYE to send later  */
    pjsip_status_code    cause;                     /**< Disconnect cause.  */
    pj_str_t             cause_text;                /**< Cause text.        */
    pj_bool_t            notify;                    /**< Internal.          */
    pj_bool_t            sdp_done_early_rel;        /**< Nego done in early
                                                         med was reliable?  */
    unsigned             cb_called;                 /**< Cb has been called */
    pjsip_dialog        *dlg;                       /**< Underlying dialog. */
    pjsip_role_e         role;                      /**< Invite role.       */
    unsigned             options;                   /**< Options in use.    */
    pjmedia_sdp_neg     *neg;                       /**< Negotiator.        */
    unsigned             sdp_neg_flags;             /**< SDP neg flags.     */
    pjsip_transaction   *invite_tsx;                /**< 1st invite tsx.    */
    pjsip_tx_data       *invite_req;                /**< Saved invite req   */
    pjsip_tx_data       *last_answer;               /**< Last INVITE resp.  */
    pjsip_tx_data       *last_ack;                  /**< Last ACK request   */
    pj_int32_t           last_ack_cseq;             /**< CSeq of last ACK   */
    void                *mod_data[PJSIP_MAX_MODULE];/**< Modules data.      */
    struct pjsip_timer  *timer;                     /**< Session Timers.    */
    pj_bool_t            following_fork;            /**< Internal, following
                                                         forked media?      */
    pj_atomic_t         *ref_cnt;                   /**< Reference counter. */
    pj_bool_t            updated_sdp_answer;        /**< SDP answer just been
                                                         updated?           */
};
```

