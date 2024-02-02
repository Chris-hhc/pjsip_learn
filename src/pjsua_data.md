## pjsua_data  pjsua_var

```c
/**
 * Global pjsua application data.
 */
struct pjsua_data
{

    /* Control: */
    pj_caching_pool      cp;        /**< Global pool factory.           */
    pj_pool_t           *pool;      /**< pjsua's private pool.          */
    pj_pool_t           *timer_pool;/**< pjsua's timer pool.            */
    pj_mutex_t          *mutex;     /**< Mutex protection for this data */
    unsigned             mutex_nesting_level; /**< Mutex nesting level. */
    pj_thread_t         *mutex_owner; /**< Mutex owner.                 */
    pjsua_state          state;     /**< Library state.                 */

    /* Logging: */
    pjsua_logging_config log_cfg;   /**< Current logging config.        */
    pj_oshandle_t        log_file;  /**<Output log file handle          */

    /* SIP: */
    pjsip_endpoint      *endpt;     /**< Global endpoint.               */
    pjsip_module         mod;       /**< pjsua's PJSIP module.          */
    pjsua_transport_data tpdata[8]; /**< Array of transports.           */
    pjsip_tp_state_callback old_tp_cb; /**< Old transport callback.     */

    /* Threading: */
    pj_bool_t            thread_quit_flag;  /**< Thread quit flag.      */
    pj_thread_t         *thread[4];         /**< Array of threads.      */

    /* STUN and resolver */
    pj_stun_config       stun_cfg;  /**< Global STUN settings.          */
    pj_sockaddr          stun_srv;  /**< Resolved STUN server address   */
    pj_status_t          stun_status; /**< STUN server status.          */
    pjsua_stun_resolve   stun_res;  /**< List of pending STUN resolution*/
    unsigned             stun_srv_idx; /**< Resolved STUN server index  */
    unsigned             stun_opt;  /**< STUN resolution option.        */
    pj_dns_resolver     *resolver;  /**< DNS resolver.                  */   

    /* UPnP */
    pj_status_t          upnp_status; /**< UPnP status.                 */

    /* Detected NAT type */
    pj_stun_nat_type     nat_type;      /**< NAT type.                  */
    pj_status_t          nat_status;    /**< Detection status.          */
    pj_bool_t            nat_in_progress; /**< Detection in progress    */

    /* List of outbound proxies: */
    pjsip_route_hdr      outbound_proxy;

    /* Account: */
    unsigned             acc_cnt;            /**< Number of accounts.   */
    pjsua_acc_id         default_acc;        /**< Default account ID    */
    pjsua_acc            acc[PJSUA_MAX_ACC]; /**< Account array.        */
    pjsua_acc_id         acc_ids[PJSUA_MAX_ACC]; /**< Acc sorted by prio*/

    /* Calls: */
    pjsua_config         ua_cfg;                /**< UA config.         */
    unsigned             call_cnt;              /**< Call counter.      */
    pjsua_call           calls[PJSUA_MAX_CALLS];/**< Calls array.       */
    pjsua_call_id        next_call_id;          /**< Next call id to use*/

    /* Buddy; */
    unsigned             buddy_cnt;                 /**< Buddy count.   */
    pjsua_buddy          buddy[PJSUA_MAX_BUDDIES];  /**< Buddy array.   */

    /* Presence: */
    pj_timer_entry       pres_timer;/**< Presence refresh timer.        */

    /* Media: */
    pjsua_media_config   media_cfg; /**< Media config.                  */
    pjmedia_endpt       *med_endpt; /**< Media endpoint.                */
    pjsua_conf_setting   mconf_cfg; /**< Additionan conf. bridge. param */
    pjmedia_conf        *mconf;     /**< Conference bridge.             */
    pj_bool_t            is_mswitch;/**< Are we using audio switchboard
                                         (a.k.a APS-Direct)             */

    /* Sound device */
    pjmedia_aud_dev_index cap_dev;  /**< Capture device ID.             */
    pjmedia_aud_dev_index play_dev; /**< Playback device ID.            */
    pj_uint32_t          aud_svmask;/**< Which settings to save         */
    pjmedia_aud_param    aud_param; /**< User settings to sound dev     */
    pj_bool_t            aud_open_cnt;/**< How many # device is opened  */
    pj_bool_t            no_snd;    /**< No sound (app will manage it)  */
    pj_pool_t           *snd_pool;  /**< Sound's private pool.          */
    pjmedia_snd_port    *snd_port;  /**< Sound port.                    */
    pj_timer_entry       snd_idle_timer;/**< Sound device idle timer.   */
    pjmedia_master_port *null_snd;  /**< Master port for null sound.    */
    pjmedia_port        *null_port; /**< Null port.                     */
    pj_bool_t            snd_is_on; /**< Media flow is currently active */
    unsigned             snd_mode;  /**< Sound device mode.             */

    /* Video device */
    pjmedia_vid_dev_index vcap_dev;  /**< Capture device ID.            */
    pjmedia_vid_dev_index vrdr_dev;  /**< Playback device ID.           */

    /* For keeping video device settings */
#if PJSUA_HAS_VIDEO
    pjmedia_vid_conf     *vid_conf;
    pj_uint32_t           vid_caps[PJMEDIA_VID_DEV_MAX_DEVS];
    pjmedia_vid_dev_param vid_param[PJMEDIA_VID_DEV_MAX_DEVS];
#endif

    /* File players: */
    unsigned             player_cnt;/**< Number of file players.        */
    pjsua_file_data      player[PJSUA_MAX_PLAYERS];/**< Array of players.*/

    /* File recorders: */
    unsigned             rec_cnt;   /**< Number of file recorders.      */
    pjsua_file_data      recorder[PJSUA_MAX_RECORDERS];/**< Array of recs.*/

    /* Video windows */
#if PJSUA_HAS_VIDEO
    pjsua_vid_win        win[PJSUA_MAX_VID_WINS]; /**< Array of windows */
#endif

    /* Timer entry and event list */
    pjsua_timer_list     active_timer_list;
    pjsua_timer_list     timer_list;
    pjsua_event_list     event_list;
    pj_mutex_t          *timer_mutex;
};
```

