## pj_ioqueue

### pj_ioqueue_callback

```c
/**
 * This structure describes the callbacks to be called when I/O operation
 * completes.
 */
typedef struct pj_ioqueue_callback
{
    /**
     * This callback is called when #pj_ioqueue_recv or #pj_ioqueue_recvfrom
     * completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param bytes_read    >= 0 to indicate the amount of data read,
     *                      otherwise negative value containing the error
     *                      code. To obtain the pj_status_t error code, use
     *                      (pj_status_t code = -bytes_read).
     */
    void (*on_read_complete)(pj_ioqueue_key_t *key,
                             pj_ioqueue_op_key_t *op_key,
                             pj_ssize_t bytes_read);

    /**
     * This callback is called when #pj_ioqueue_send or #pj_ioqueue_sendto
     * completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param bytes_sent    >= 0 to indicate the amount of data written,
     *                      otherwise negative value containing the error
     *                      code. To obtain the pj_status_t error code, use
     *                      (pj_status_t code = -bytes_sent).
     */
    void (*on_write_complete)(pj_ioqueue_key_t *key,
                              pj_ioqueue_op_key_t *op_key,
                              pj_ssize_t bytes_sent);

    /**
     * This callback is called when #pj_ioqueue_accept completes.
     *
     * @param key           The key.
     * @param op_key        Operation key.
     * @param sock          Newly connected socket.
     * @param status        Zero if the operation completes successfully.
     */
    void (*on_accept_complete)(pj_ioqueue_key_t *key,
                               pj_ioqueue_op_key_t *op_key,
                               pj_sock_t sock,
                               pj_status_t status);

    /**
     * This callback is called when #pj_ioqueue_connect completes.
     *
     * @param key           The key.
     * @param status        PJ_SUCCESS if the operation completes successfully.
     */
    void (*on_connect_complete)(pj_ioqueue_key_t *key,
                                pj_status_t status);
} pj_ioqueue_callback;
```

