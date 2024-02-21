## pjmedia_format

This structure contains all the information needed to completely describe a media.

This structure is put in \a *detail* field of #pjmedia_format to describe detail information about an audio media.

```c
/**
 * This structure contains all the information needed to completely describe
 * a media.
 */
typedef struct pjmedia_format
{
    /**
     * The format id that specifies the audio sample or video pixel format.
     * Some well known formats ids are declared in pjmedia_format_id
     * enumeration.
     *
     * @see pjmedia_format_id
     */
    pj_uint32_t                  id;

    /**
     * The top-most type of the media, as an information.
     */
    pjmedia_type                 type;

    /**
     * The type of detail structure in the \a detail pointer.
     */
    pjmedia_format_detail_type   detail_type;

    /**
     * Detail section to describe the media.
     */
    union
    {
        /**
         * Detail section for audio format.
         */
        pjmedia_audio_format_detail     aud;

        /**
         * Detail section for video format.
         */
        pjmedia_video_format_detail     vid;

        /**
         * Reserved area for user-defined format detail.
         */
        char                            user[PJMEDIA_FORMAT_DETAIL_USER_SIZE];
    } det;

} pjmedia_format;
```



## pjmedia_audio_format_detail

```c
/**
 * This structure is put in \a detail field of #pjmedia_format to describe
 * detail information about an audio media.
 */
typedef struct pjmedia_audio_format_detail
{
    unsigned    clock_rate;     /**< Audio clock rate in samples or Hz. */
    unsigned    channel_count;  /**< Number of channels.                */
    unsigned    frame_time_usec;/**< Frame interval, in microseconds.   */
    unsigned    bits_per_sample;/**< Number of bits per sample.         */
    pj_uint32_t avg_bps;        /**< Average bitrate                    */
    pj_uint32_t max_bps;        /**< Maximum bitrate                    */
} pjmedia_audio_format_detail;

```

