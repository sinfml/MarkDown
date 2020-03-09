---
layout:     post
title:      Android对多国语言字符串进行排序
subtitle:   举例对日语排序
date:       2020-3-5
author:     sinfml
catalog: true
tags:
    - Android
---

### 问题

对于日语列表的排序是比较复杂的,一般分为数字0~9,字母A~Z,假名,汉字
对于看不懂日语,对于假名以及汉字怎样才是正确的排序并不能确定下来;
因此借用Android中的一些接口,结合数据库或者Collections对列表进行快速的排序

### 处理

主要用到的是Collator这个类

"Collator 类执行区分语言环境的 String 比较。使用此类可为自然语言文本构建搜索和排序例程。 
Collator 是一个抽象基类。其子类实现具体的整理策略。Java 平台目前提供了 RuleBasedCollator 子类，它适用于很多种语言。还可以创建其他子类，以处理更多的专门需要"

它针对的是某个特定语言环境下,字符串的一些排序细节,就如Collections.sort 中的权值.
例如我们要对字符串在日语环境下排序,就通过 Collator collator = Collator.getInstance(Locale.JAPANESE); 构建对象

然后我们需要一个字符串 String abc = "xxxxxx";

通过collator.getCollationKey(abc); 获得CollatorKey对象;

关于CollatorKey对象: 看下它的注释

```Java
/**
     * Transforms the String into a series of bits that can be compared bitwise
     * to other CollationKeys. CollationKeys provide better performance than
     * Collator.compare when Strings are involved in multiple comparisons.
     * See the Collator class description for an example using CollationKeys.
     * @param source the string to be transformed into a collation key.
     * @return the CollationKey for the given String based on this Collator's collation
     * rules. If the source String is null, a null CollationKey is returned.
     * @see java.text.CollationKey
     * @see java.text.Collator#compare
     */
    public abstract CollationKey getCollationKey(String source);
```

简单的来说,就是根据语言环境,将字符串转化为一串字节数组,用于和其它的字符串作为对比,也就是获取到了这个字符串的权值对象
但要获取对应真正可以观察到的数据,需要通过 byte[] byteArray = collator.getCollationKey(abc).toByteArray(); 放进数组中;

但为了方便观看,我们要对数组做一些处理

```Java

        int move = 5;
        int index = 0;

        for (byte b : byteArray) {
            value += ((b & 0xff) * Math.pow(1000, move));
            move--;
            index++;
            if (index == 6) {
                break;
            }
        }

```
相当于取前6个byte,然后每处理一个byte就*1000放在1个3位里面,在把他们加起来
ex:
 byte[] b :      b[0] b[1] b[2] ...
 long   x :      aaa  bbb  ccc ...


然后看下处理完成后的例子:
/*字符串*/                                            /*权值*/       

Everybody (Logic Mix)                               177211177203217152
Attention                                           169207207177195200
いちご娘はひとりっ子                                  38008040026072032 
鬼日_1117KIBI                                         38016050060003008
電気グルーヴ30周年の唄                                38044104020022088
いちご娘（ひとりっ子でない）                          38008040026072032
電気グルーヴ10周年の歌 2019                           38044104020022088
WIRE WIRED, WIRELESS                                213185203177004224
Slow Motion（30th Mix）                             205191197213004192

很明显,根据大小排序的话,日语或者汉字是排在英文前面的,这就是一个字符串的权值,至于怎么利用,可以通过db中的sort排序,亦或者Collections.sort排序了

