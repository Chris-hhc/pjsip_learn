## Codec

### pjmedia_codec_factory

```c
struct pjmedia_codec_factory
{
    /** Entries to put this structure in the codec manager list. */
    PJ_DECL_LIST_MEMBER(struct pjmedia_codec_factory);

    /** The factory's private data. */
    void                     *factory_data;

    /** Operations to the factory. */
    pjmedia_codec_factory_op *op;

};

```

### pjmedia_codec_factory_op

```c
/**
 * This structure describes operations that must be supported by codec 
 * factories.
 */
typedef struct pjmedia_codec_factory_op
{
    /** 
     * Check whether the factory can create codec with the specified 
     * codec info.
     *
     * @param factory   The codec factory.
     * @param info      The codec info.
     *
     * @return          PJ_SUCCESS if this factory is able to create an
     *                  instance of codec with the specified info.
     */
    pj_status_t (*test_alloc)(pjmedia_codec_factory *factory, 
                              const pjmedia_codec_info *info );

    /** 
     * Create default attributes for the specified codec ID. This function
     * can be called by application to get the capability of the codec.
     *
     * @param factory   The codec factory.
     * @param info      The codec info.
     * @param attr      The attribute to be initialized.
     *
     * @return          PJ_SUCCESS if success.
     */
    pj_status_t (*default_attr)(pjmedia_codec_factory *factory, 
                                const pjmedia_codec_info *info,
                                pjmedia_codec_param *attr );

    /** 
     * Enumerate supported codecs that can be created using this factory.
     * 
     *  @param factory  The codec factory.
     *  @param count    On input, specifies the number of elements in
     *                  the array. On output, the value will be set to
     *                  the number of elements that have been initialized
     *                  by this function.
     *  @param info     The codec info array, which contents will be 
     *                  initialized upon return.
     *
     *  @return         PJ_SUCCESS on success.
     */
    pj_status_t (*enum_info)(pjmedia_codec_factory *factory, 
                             unsigned *count, 
                             pjmedia_codec_info codecs[]);

    /** 
     * Create one instance of the codec with the specified codec info.
     *
     * @param factory   The codec factory.
     * @param info      The codec info.
     * @param p_codec   Pointer to receive the codec instance.
     *
     * @return          PJ_SUCCESS on success.
     */
    pj_status_t (*alloc_codec)(pjmedia_codec_factory *factory, 
                               const pjmedia_codec_info *info,
                               pjmedia_codec **p_codec);

    /** 
     * This function is called by codec manager to return a particular 
     * instance of codec back to the codec factory.
     *
     * @param factory   The codec factory.
     * @param codec     The codec instance to be returned.
     *
     * @return          PJ_SUCCESS on success.
     */
    pj_status_t (*dealloc_codec)(pjmedia_codec_factory *factory, 
                                 pjmedia_codec *codec );

    /**
     * This callback will be called to deinitialize and destroy this factory.
     */
    pj_status_t (*destroy)(void);

} pjmedia_codec_factory_op;
```

