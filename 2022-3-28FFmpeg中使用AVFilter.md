---
layout:     post
title:      2022-3-28FFmpeg中使用AVFilter
subtitle:   FFmpeg中使用Filter对信号作简单处理
date:       2022-3-24
author:     sinfml
catalog: true
tags:
    - FFmpeg
---

## 前提

使用AVFilter对音频信号作处理

## 配置流程

#### 1、打开文件获取AVFormatContext以及AVCodecContext

```C
int ret, i;
int stream_index = -1;
AVFormatContext *fmt_ctx;
AVCodec *codec;
AVCodecContext *cdc_ctx;

const char *file_name = "...";

if (ret = avformat_open_input(&fmt_ctx, file_name, NULL, NULL)) {
    return failure;
}

avformat_find_stream_info(fmt_ctx, NULL);

for (i = 0; i < fmt_ctx->nb_streams, ++i) {
    if (fmt_ctx->streams[i]->codec->codec_type == AVMEDIA_TYPE_AUDIO) {
        //找到音频codec
        stream_index = i;
        break;
    }
}

if (stream_index < 0) {
    avformat_close_input(&fmt_ctx);
    return failure;
}

cdc_ctx = fmt_ctx->streams[i]->codec;
codec = avcodec_find_decoder(cdc_ctx->codec_id);
avcodec_copy_context(codec, cdc_ctx);

if ((avcodec_open2(codec, cdc_ctx, NULL)) < 0) {
    return failure;
}
```

#### 2、根据AVCodecContext中的采样率等配置，初始化AVFilter

AVFilter主要由三部分组成，AVFilterGraph、AVFilterContext、AVFilter。其中AVFilter代表一个类型的filter，而AVFilterContext就是这个AVFilter的实例化。AVFilter包含了输入以及输出。

AVFilterGraph表示一个网状结构，包含了诸多AVFilterContext，以一个filter的输出作为另外一个filter的输入，连接起来，用以实现复杂的filter功能。

```
 _________
|         |
| input 0 |\                    __________
|_________| \                  |          |
             \   _________    /| output 0 |
              \ |         |  / |__________|
 _________     \| complex | /
|         |     |         |/
| input 1 |---->| filter  |\
|_________|     |         | \   __________
               /| graph   |  \ |          |
              / |         |   \| output 1 |
 _________   /  |_________|    |__________|
|         | /
| input 2 |/
|_________|

example :
 * The filter chain it uses is:
 * (input) -> abuffer -> volume -> aformat -> abuffersink -> (output)

```


代码，以FFmpeg中的filter_audio.c为例
```C
    AVFilterGraph *filter_graph;
    AVFilterContext *abuffer_ctx;
    AVFilter        *abuffer;
    AVFilterContext *volume_ctx;
    AVFilter        *volume;
    AVFilterContext *aformat_ctx;
    AVFilter        *aformat;
    AVFilterContext *abuffersink_ctx;
    AVFilter        *abuffersink;

    AVDictionary *options_dict = NULL;
    uint8_t options_str[1024];
    uint8_t ch_layout[64];

    int err;


    //注册filter
    avfilter_register_all();

    //创建一个filter_graph
    filter_graph = avfilter_graph_alloc();

    //获取abuffer的filter类型，这个是作为输入chain
    abuffer = avfilter_get_by_name("abuffer");
    if (!abuffer) {
        fprintf(stderr, "Could not find the abuffer filter.\n");
        return AVERROR_FILTER_NOT_FOUND;
    }

    //创建abuffer的filter实例， "src"代表该chain的名字
    abuffer_ctx = avfilter_graph_alloc_filter(filter_graph, abuffer, "src");
    if (!abuffer_ctx) {
        fprintf(stderr, "Could not allocate the abuffer instance.\n");
        return AVERROR(ENOMEM);
    }


    //设置filter中的AVOptions，src配置输入参数，可以从AVFormatContext以及AVCodecContext中获取
    av_get_channel_layout_string(ch_layout, sizeof(ch_layout), 0, INPUT_CHANNEL_LAYOUT);
    av_opt_set    (abuffer_ctx, "channel_layout", ch_layout,                            AV_OPT_SEARCH_CHILDREN);
    av_opt_set    (abuffer_ctx, "sample_fmt",     av_get_sample_fmt_name(INPUT_FORMAT), AV_OPT_SEARCH_CHILDREN);
    av_opt_set_q  (abuffer_ctx, "time_base",      (AVRational){ 1, INPUT_SAMPLERATE },  AV_OPT_SEARCH_CHILDREN);
    av_opt_set_int(abuffer_ctx, "sample_rate",    INPUT_SAMPLERATE,                     AV_OPT_SEARCH_CHILDREN);

    /* 创建abuffer对应的abuffer_ctx实例，这里options传NULL即可，上面已经配置进去. */
    err = avfilter_init_str(abuffer_ctx, NULL);
    if (err < 0) {
        fprintf(stderr, "Could not initialize the abuffer filter.\n");
        return err;
    }


    /** 从AVFormatContext以及AVCodecContext中获取, 跟上面代码效果一致
        if (!cdc_ctx->channel_layout)
                cdc_ctx->channel_layout = av_get_default_channel_layout(cdc_ctx->channels);
        snprintf(args, sizeof(args),
            "time_base=%d/%d:sample_rate=%d:sample_fmt=%s:channel_layout=0x%"PRIx64,
             time_base.num, time_base.den, cdc_ctx->sample_rate,
             av_get_sample_fmt_name(cdc_ctx->sample_fmt), cdc_ctx->channel_layout);

        err = avfilter_graph_create_filter(&abuffer_ctx, abuffer, "src",
                                       args, NULL, filter_graph);
    **/

    /* 创建volume类型的filter. */
    volume = avfilter_get_by_name("volume");
    if (!volume) {
        fprintf(stderr, "Could not find the volume filter.\n");
        return AVERROR_FILTER_NOT_FOUND;
    }

    volume_ctx = avfilter_graph_alloc_filter(filter_graph, volume, "volume");
    if (!volume_ctx) {
        fprintf(stderr, "Could not allocate the volume instance.\n");
        return AVERROR(ENOMEM);
    }

    /* A different way of passing the options is as key/value pairs in a
     * dictionary. */
    av_dict_set(&options_dict, "volume", AV_STRINGIFY(VOLUME_VAL), 0);
    err = avfilter_init_dict(volume_ctx, &options_dict);
    av_dict_free(&options_dict);
    if (err < 0) {
        fprintf(stderr, "Could not initialize the volume filter.\n");
        return err;
    }

    /* Create the aformat filter;
     * it ensures that the output is of the format we want. */
    aformat = avfilter_get_by_name("aformat");
    if (!aformat) {
        fprintf(stderr, "Could not find the aformat filter.\n");
        return AVERROR_FILTER_NOT_FOUND;
    }

    aformat_ctx = avfilter_graph_alloc_filter(filter_graph, aformat, "aformat");
    if (!aformat_ctx) {
        fprintf(stderr, "Could not allocate the aformat instance.\n");
        return AVERROR(ENOMEM);
    }

    /* A third way of passing the options is in a string of the form
     * key1=value1:key2=value2.... */
    snprintf(options_str, sizeof(options_str),
             "sample_fmts=%s:sample_rates=%d:channel_layouts=0x%"PRIx64,
             av_get_sample_fmt_name(AV_SAMPLE_FMT_S16), 44100,
             (uint64_t)AV_CH_LAYOUT_STEREO);
    err = avfilter_init_str(aformat_ctx, options_str);
    if (err < 0) {
        av_log(NULL, AV_LOG_ERROR, "Could not initialize the aformat filter.\n");
        return err;
    }

    /* 最后创建abuffersink类型的filter，这个是作为输出chain. */
    abuffersink = avfilter_get_by_name("abuffersink");
    if (!abuffersink) {
        fprintf(stderr, "Could not find the abuffersink filter.\n");
        return AVERROR_FILTER_NOT_FOUND;
    }

    abuffersink_ctx = avfilter_graph_alloc_filter(filter_graph, abuffersink, "sink");
    if (!abuffersink_ctx) {
        fprintf(stderr, "Could not allocate the abuffersink instance.\n");
        return AVERROR(ENOMEM);
    }

    /* This filter takes no options. */
    err = avfilter_init_str(abuffersink_ctx, NULL);
    if (err < 0) {
        fprintf(stderr, "Could not initialize the abuffersink instance.\n");
        return err;
    }

    /* 连接其所有的filter，(in)abuffer_cxt-->volume_ctx->>aformat_ctx-->abuffersink_ctx (out). */
    err = avfilter_link(abuffer_ctx, 0, volume_ctx, 0);
    if (err >= 0)
        err = avfilter_link(volume_ctx, 0, aformat_ctx, 0);
    if (err >= 0)
        err = avfilter_link(aformat_ctx, 0, abuffersink_ctx, 0);
    if (err < 0) {
        fprintf(stderr, "Error connecting filters\n");
        return err;
    }

    /* 最终配置到filter_graph. */
    err = avfilter_graph_config(filter_graph, NULL);
    if (err < 0) {
        av_log(NULL, AV_LOG_ERROR, "Error configuring the filter graph\n");
        return err;
    }

```

#### 3、解码时，调用AVFilter对其中的数据作处理

数据处理主要由2个接口组成，av_buffersrc_add_frame(abuffer_ctx, frame)，以及av_buffersink_get_frame_flags(abuffersink_ctx, frame, 0)。


以filter_audio.c为例

```C

    /* the main filtering loop */
    for (i = 0; i < nb_frames; i++) {
        /* get an input frame to be filtered */
        err = get_input(frame, i);
        if (err < 0) {
            fprintf(stderr, "Error generating input frame:");
            goto fail;
        }

        /* 把数据放入buffersrc中，即上文提到的src chain里面. */
        err = av_buffersrc_add_frame(src, frame);
        if (err < 0) {
            av_frame_unref(frame);
            fprintf(stderr, "Error submitting the frame to the filtergraph:");
            goto fail;
        }

        /* 再从out chain中把处理好的数据重新取出. */
        while ((err = av_buffersink_get_frame(sink, frame)) >= 0) {
            /* now do something with our filtered frame */
            err = process_output(md5, frame);
            if (err < 0) {
                fprintf(stderr, "Error processing the filtered frame:");
                goto fail;
            }
            av_frame_unref(frame);
        }

        if (err == AVERROR(EAGAIN)) {
            /* Need to feed more frames in. */
            continue;
        } else if (err == AVERROR_EOF) {
            /* Nothing more to do, finish. */
            break;
        } else if (err < 0) {
            /* An error occurred. */
            fprintf(stderr, "Error filtering the data:");
            goto fail;
        }
    }

    avfilter_graph_free(&graph);

```