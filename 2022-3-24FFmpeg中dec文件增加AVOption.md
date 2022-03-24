---
layout:     post
title:      2022-3-24FFmpeg中dec文件增加AVOption
subtitle:   DST解码增加Avoption,使得支持传送Native数据
date:       2022-3-24
author:     sinfml
catalog: true
tags:
    - FFmpeg
---

## 前提

需要在打开dst decoder时,告诉decoder需要使用Native还是使用D2P,从而获取不同的数据,但dec本身只支持D2P方式,因此需要通过AVOption在打开的时候告诉decoder需要什么解码模式.

## 代码

#### 1、libavcodec/dstdec.c增加define定义

```C

#define DICT_DSD_NATIVE 0 // 支持NATIVE
#define DICT_DSD_D2P    1 // 支持D2P

```

#### 2、libavcodec/dstdec.c DSTContext 增加需要的变量dsd_mode

```C

typedef struct DSTContext {
    AVClass *class;

    GetBitContext gb;
    ArithCoder ac;
    Table fsets, probs;
    DECLARE_ALIGNED(16, uint8_t, status)[DST_MAX_CHANNELS][16];
    DECLARE_ALIGNED(16, int16_t, filter)[DST_MAX_ELEMENTS][16][256];
    DSDContext dsdctx[DST_MAX_CHANNELS];

    int dsd_mode;  //增加dsd_mode用于区分Native以及D2P
} DSTContext;


```

#### 3、libavcodec/dstdec.c 增加对应的AVOption以及AVClass用于接收AVDictionary传递的参数

```C

static const AVOption options[] = {
{ "set_dsd_mode", "set native/d2p as dsd mode", offsetof(DSTContext, dsd_mode), AV_OPT_TYPE_INT, {.i64 = 0 }, 0, 1, AV_OPT_FLAG_DECODING_PARAM | AV_OPT_FLAG_AUDIO_PARAM },
{ NULL },
};

static const AVClass dst_decoder_class = {
    "DST decoder",
    av_default_item_name,
    options,
    LIBAVUTIL_VERSION_INT,
};

AVCodec ff_dst_decoder = {
    .name           = "dst",
    .long_name      = NULL_IF_CONFIG_SMALL("DST (Digital Stream Transfer)"),
    .type           = AVMEDIA_TYPE_AUDIO,
    .id             = AV_CODEC_ID_DST,
    .priv_data_size = sizeof(DSTContext),
    .init           = decode_init,
    .decode         = decode_frame,
    .capabilities   = CODEC_CAP_DR1,
    .sample_fmts    = (const enum AVSampleFormat[]) { AV_SAMPLE_FMT_FLT, AV_SAMPLE_FMT_NONE },
    .priv_class     = &dst_decoder_class,
};


```

#### 4、在打开文件时，传递AVDictionary用于配置对应的dsd_mode

```C
//创建AVDictionary
AVDictionary *dic = NULL; 
uint8_t ret;

av_register_all();

ret = avformat_open_input(&ctx, fileName, NULL, NULL);

#define DSD_NATIVE 0
#define DSD_D2P    1
av_dict_set_int(&dic, "set_dsd_mode", DSD_D2P, 0);

//将dsd_mode的参数通过avcodec_open传递到对应decoder中
ret = avcodec_open2(avctx, avcodec, &dic);

//最后释放掉avdic
av_dict_free(&dic);

return ret;
```

## 关于AVDictionary、AVOption以及AVClass

1、AVDictionary相当于Java中的map，结构为key-value，用于FFmpeg中的值传递。
创建需要将其赋值为NULL。通过av_dict_set传递指针变量的地址，以及对应的key-value设置值。
```C
AVDictionary *dic = NULL; 


// int av_dict_set(AVDictionary **pm, const char *key, const char *value, int flags)
av_dict_set(&dic, ..., ..., ...);

```

2、其中flags代表不同的选项组合，包括
```C

#define AV_DICT_MATCH_CASE      1   /**< Only get an entry with exact-case key match. Only relevant in av_dict_get(). */
#define AV_DICT_IGNORE_SUFFIX   2   /**< Return first entry in a dictionary whose first part corresponds to the search key,
                                         ignoring the suffix of the found key string. Only relevant in av_dict_get(). */
#define AV_DICT_DONT_STRDUP_KEY 4   /**< Take ownership of a key that's been
                                         allocated with av_malloc() or another memory allocation function. */
#define AV_DICT_DONT_STRDUP_VAL 8   /**< Take ownership of a value that's been
                                         allocated with av_malloc() or another memory allocation function. */
#define AV_DICT_DONT_OVERWRITE 16   ///< Don't overwrite existing entries.
#define AV_DICT_APPEND         32   /**< If the entry already exists, append to it.  Note that no
                                      delimiter is added, the strings are simply concatenated. */

```
- AV_DICT_MATCH_CASE ：检索key区分大小写，相同key将会覆盖
- AV_DICT_IGNORE_SUFFIX ：相当于like，忽略后缀，例如version，传递ver也能够匹配。
- AV_DICT_DONT_STRDUP_KEY/AV_DICT_DONT_STRDUP_VAL ：接收方去处理内存管理
- AV_DICT_DONT_OVERWRITE ：不覆盖现有的key，若之前已配置key，再设置同一个key不会覆盖上一个内容
- AC_DICT_APPEND ：在现有key中的value追加信息，相当于字符串拼接。

3、AVDictionary几个主要函数

```C
av_dict_get
av_dict_set
av_dict_set_int
av_dict_copy
av_dict_free

```

4、AVClass最主要的作用是给结构体AVFormatContext等增加AVOption功能，例如在decoder中一般通过priv_class，传递AVOption中的变量，赋值给AVFormatContext中的变量。

例如在上文提到的AVOption

```C
static const AVOption options[] = {

{ "set_dsd_mode", "set native/d2p as dsd mode", offsetof(DSTContext, dsd_mode), AV_OPT_TYPE_INT, {.i64 = 0 }, 0, 1, AV_OPT_FLAG_DECODING_PARAM | AV_OPT_FLAG_AUDIO_PARAM },

{ NULL },

};

```

```C
AVOption

typedef struct AVOption {
    const char *name;

    /**
     * short English help text
     * @todo What about other languages?
     */
    const char *help;

    /**
     * The offset relative to the context structure where the option
     * value is stored. It should be 0 for named constants.
     */
    int offset;
    enum AVOptionType type;

    /**
     * the default value for scalar options
     */
    union {
        int64_t i64;
        double dbl;
        const char *str;
        /* TODO those are unused now */
        AVRational q;
    } default_val;
    double min;                 ///< minimum valid value for the option
    double max;                 ///< maximum valid value for the option

    int flags;
#define AV_OPT_FLAG_ENCODING_PARAM  1   ///< a generic parameter which can be set by the user for muxing or encoding
#define AV_OPT_FLAG_DECODING_PARAM  2   ///< a generic parameter which can be set by the user for demuxing or decoding
#if FF_API_OPT_TYPE_METADATA
#define AV_OPT_FLAG_METADATA        4   ///< some data extracted or inserted into the file like title, comment, ...
#endif
#define AV_OPT_FLAG_AUDIO_PARAM     8
#define AV_OPT_FLAG_VIDEO_PARAM     16
#define AV_OPT_FLAG_SUBTITLE_PARAM  32
/**
 * The option is inteded for exporting values to the caller.
 */
#define AV_OPT_FLAG_EXPORT          64
/**
 * The option may not be set through the AVOptions API, only read.
 * This flag only makes sense when AV_OPT_FLAG_EXPORT is also set.
 */
#define AV_OPT_FLAG_READONLY        128
#define AV_OPT_FLAG_FILTERING_PARAM (1<<16) ///< a generic parameter which can be set by the user for filtering
//FIXME think about enc-audio, ... style flags

    /**
     * The logical unit to which the option belongs. Non-constant
     * options and corresponding named constants share the same
     * unit. May be NULL.
     */
    const char *unit;
} AVOption;

```

- name 代表配置的名称，在AVDctionary配置中的key。  
- help 代表描述。  
- offset 代表在对应AVFormatContext结构体中需要配置的成员变量的偏移值。  
- type 代表value数据类型  
- default_val 代表默认选项
- min/max 选项的最小/最大值
- flags 标记
- unit 逻辑单元