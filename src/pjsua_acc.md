## pjsua_acc

```c
/**
 * Account
 */
typedef struct pjsua_acc
{
    pj_pool_t       *pool;          /**< Pool for this account.         */
    pjsua_acc_config cfg;           /**< Account configuration.         */
    pj_bool_t        valid;         /**< Is this account valid?         */

    int              index;         /**< Index in accounts array.       */
    pj_str_t         display;       /**< Display name, if any.          */
    pj_str_t         user_part;     /**< User part of local URI.        */
    pj_bool_t        is_sips;       /**< Local URI uses "sips"?         */
    pj_str_t         contact;       /**< Our Contact header.            */
    pj_str_t         reg_contact;   /**< Contact header for REGISTER.
                                         It may be different than acc
                                         contact if outbound is used    */
    pj_bool_t        contact_rewritten;
                                    /**< Contact rewrite has been done? */
    pjsip_host_port  via_addr;      /**< Address for Via header         */
    pjsip_transport *via_tp;        /**< Transport associated with
                                         the Via address                */

    pj_str_t         srv_domain;    /**< Host part of reg server.       */
    int              srv_port;      /**< Port number of reg server.     */

    pjsip_regc      *regc;          /**< Client registration session.   */
    pj_status_t      reg_last_err;  /**< Last registration error.       */
    int              reg_last_code; /**< Last status last register.     */

    pj_str_t         reg_mapped_addr;/**< Our addr as seen by reg srv.
                                          Only if allow_sdp_nat_rewrite
                                          is set                        */

    struct {
        pj_bool_t        active;    /**< Flag of reregister status.     */
        pj_timer_entry   timer;     /**< Timer for reregistration.      */
        void            *reg_tp;    /**< Transport for registration.    */
        unsigned         attempt_cnt; /**< Attempt counter.             */
    } auto_rereg;                   /**< Reregister/reconnect data.     */

    pj_timer_entry   ka_timer;      /**< Keep-alive timer for UDP.      */
    pjsip_transport *ka_transport;  /**< Transport for keep-alive.      */
    pj_sockaddr      ka_target;     /**< Destination address for K-A    */
    unsigned         ka_target_len; /**< Length of ka_target.           */

    pjsip_route_hdr  route_set;     /**< Complete route set inc. outbnd.*/
    pj_uint32_t      global_route_crc; /** CRC of global route setting. */
    pj_uint32_t      local_route_crc;  /** CRC of account route setting.*/

    unsigned         rfc5626_status;/**< SIP outbound status:
                                           0: not used
                                           1: requested
                                           2: acknowledged by servers   */
    pj_str_t         rfc5626_instprm;/**< SIP outbound instance param.  */
    pj_str_t         rfc5626_regprm;/**< SIP outbound reg param.        */
    unsigned         rfc5626_flowtmr;/**< SIP outbound flow timer.      */

    unsigned         cred_cnt;      /**< Number of credentials.         */
    pjsip_cred_info  cred[PJSUA_ACC_MAX_PROXIES]; /**< Complete creds.  */

    pj_bool_t        online_status; /**< Our online status.             */
    pjrpid_element   rpid;          /**< RPID element information.      */
    pjsua_srv_pres   pres_srv_list; /**< Server subscription list.      */
    pjsip_publishc  *publish_sess;  /**< Client publication session.    */
    pj_bool_t        publish_state; /**< Last published online status   */

    pjsip_evsub     *mwi_sub;       /**< MWI client subscription        */
    pjsip_dialog    *mwi_dlg;       /**< Dialog for MWI sub.            */

    pj_uint16_t      next_rtp_port; /**< Next RTP port to be used.      */
    pjsip_transport_type_e tp_type; /**< Transport type (for local acc or
                                         transport binding)             */
    pjsua_ip_change_op ip_change_op;/**< IP change process progress.    */
} pjsua_acc;
```

