#Media Source Extensions
> W3C Candidate Recommendation
> 31 MARCH 2015

##Infomation
>最新发布版本：  
>http://www.w3.org/TR/media-source/  
>最新编辑者草案：  
>http://w3c.github.io/media-source/  
>以前的版本：  
>http://www.w3.org/TR/2014/CR-media-source-20140717/  
>编辑者：  
>Aaron Colwell, Google Inc.  
>Adrian Bateman, Microsoft Corporation  
>Mark Watson, Netflix Inc.  
>翻译者：  
>Hanzeil

##Abstract
Media Source API继承于HTMLMediaElement，它允许JavaScript生成可以播放的媒体流，使许多应用场景更加便利，比如自适应流及时移(adaptive streaming and time shifting live streams).  
如果你想对这篇W3C的文档提出建议或者提交bug，请点击[our public bug database](https://www.w3.org/Bugs/Public/enter_bug.cgi?product=HTML%20WG&component=Media%20Source%20Extensions&short_desc=%5BMSE%5D%20).

##Status of this Document
这一章节描述了当前发布的文档的状态，其他的文档可能会代替当前文档。获取当前的W3C发布列表和这篇技术文档的最新版本，请点击[W3C技术报告索引](http://www.w3.org/TR/)。  

工作组维护了[一个编辑者并未试图解决的bug列表](http://w3.org/brief/Mjcw)，该草案突出了一些工作组将要讨论的未解决问题。这些问题还没有决策，包括他们是否具有有效的结果。  

该规范没有[实现报告](http://www.w3.org/wiki/ImplementationReport)。  

实现者们应该注意，该规范并不稳定，若没有加入该讨论组，下个版本可能跟当前版本不兼容。感兴趣的实现者在实现之前，应加入到下面提到的邮件列表，参加讨论。  

由于缺乏实现，下面的特性有一定风险会被移除。[TotalFrameDelay](http://www.w3.org/TR/media-source/#widl-VideoPlaybackQuality-totalFrameDelay)  

该文档已经被[HTML工作组](http://www.w3.org/html/wg/)作为W3C候选标准发布了，该文档旨在成为W3C标准，如果您对本文提出意见，请发送给 public-html-media@w3.org，W3C发布了该候选标准，说明该文档是稳定的，并鼓励开发者社区实现。该候选推荐很可能会提前发布，最早可能在2015年4月28日。欢迎所有评论。  

作为W3C候选标准发布并不意味着W3C成员对其支持。这是一个草案，可能会随时更新、替换、丢弃。引用此文档做其他工作并不合适。  

工作组制定该文档，并在[5 February 2014 W3C Patent Policy](http://www.w3.org/Consortium/Patent-Policy-20040205/)下运行。W3C维护一个任何专利公开公示名单。该网页还包括用于公开的专利说明书。按照[W3C专利政策的第六章](http://www.w3.org/Consortium/Patent-Policy-20040205/#sec-Disclosure)，拥有专利知识的个人，必须公布包含[Essential Claim](http://www.w3.org/Consortium/Patent-Policy-20040205/#def-essential)的信息。  

该文档被[1 August 2014 W3C Process Document](http://www.w3.org/2014/Process-20140801/)所管理。  

##1. 简介
本规范允许JavaScript向`<video>`和`<audio>`标签动态构造媒体流，它定义对象允许JavaScript加载媒体分片转化为[HTMLMediaElement](http://www.w3.org/TR/html5/embedded-content-0.html#htmlmediaelement).同时，通过一种缓冲模型也可以实现用户代理（浏览器）播放无序被添加的媒体分片。在[MSE-REGISTRY](http://www.w3.org/TR/media-source/#bib-MSE-REGISTRY)中定义了媒体流的格式规范。  
![pieline_model](http://www.w3.org/TR/media-source/pipeline_model.png)
###1.1 目标
这篇规范设计了以下目标：

*   允许JavaScript独立构建媒体流；
*   定义了一种拼接模型和缓冲模型，使得一些应用场景更为便利，比如自适应流及时移，时移，视频编辑等；
*   最大限度减少JavaScript解析媒体的使用；
*   尽可能使用浏览器缓存；
*   不需要任何媒体格式和编解码器的支持。

###1.2 定义

####Active Track Buffers（轨道缓冲器）
active Track Buffers可以提供[enabled](http://www.w3.org/TR/html5/embedded-content-0.html#dom-audiotrack-enabled) [audioTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-audiotracks),  [selected](http://www.w3.org/TR/html5/embedded-content-0.html#dom-videotrack-selected) [videoTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-videotracks)和 "[showing](http://www.w3.org/TR/html5/embedded-content-0.html#dom-texttrack-showing)" 或者 "[hidden](http://www.w3.org/TR/html5/embedded-content-0.html#dom-texttrack-hidden)" [textTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-texttracks)的编码帧的缓冲，在activeSourceBuffers属性里，与SourceBuffer对象相关联。  

####append Window（添加窗口）
当添加媒体分片时，用显示时间戳来界定编码帧。append Window具有一个单一连续时间内的开始时间和结束时间，编码帧的显示时间戳如果属于此范围，则允许添加到SourceBuffer中，而此范围外的编码帧会被过滤。append Window的开始时间和结束时间由属性appendWindow-Start和appendWindowEnd属性控制。  

####Coded Frame（编码帧）
一个媒体数据单元，具有一个显示时间戳，一个解码时间戳，和一个编码帧长。  

####Coded Frame Duration（编码帧长）
编码帧的时间长度。对于视频和文字，时长表示视频帧的长度或者文字是否应该展示。对于音频，时长表示编码帧内所有音频采样的和。比如，如果一个音频帧包含441个采样，频率为44100Hz，那么编码帧长是10毫秒。  

####Coded Frame End Timestamp（编码帧结束时间戳）
编码帧显示时间戳和它的帧长之和。它表示显示时间戳紧随的编码帧。

####Coded Frame Group（编码帧组）
编码帧组是一组邻接的、单调递增的解码时间戳且没有任何间隙的编码帧。如果通过coded frame processing算法检测到不连续的帧，那么abort()方法将会触发，而且开始新的编码帧组。  

####Decode Timestamp（解码时间戳）
假设帧需要立即解码和展现（该帧和任何依赖它的帧），解码时间戳表示最新需要解码的帧的时间（等同于显示时间戳最早的帧，在显示顺序中，它是依赖此帧的）。如果帧可以在显示顺序之外被解码，则解码时间戳必须存在于或衍生于字节流，如果不是，用户代理必须运行append error算法并将decode error参数设置为true。如果解码时间戳不能在显示顺序之外解码，而且不存在于字节流，那么解码时间戳等同于显示时间戳。  

####Displayed Frame Delay（展示帧延迟）
帧的理论显示时间和实际时间（双精度类型，以秒为单位，四舍五入的离显示刷新间隔最接近的时间）之间的延迟，该延迟大于等于0，因为帧不会在它应该显示的时间之前显示。如果延迟大于0，说明可能存在播放抖动和A/V同步丢失。  

####Initialization Segment（初始化分片）
包含所有初始化信息的一个字节序列，可以解码剩下所有的媒体分片。它包含初始化解码信息，多轨道分片的轨道ID，和相应的时间偏移量。 

>在字节流格式登记文档[MSE-REGISTRY](http://www.w3.org/TR/media-source/#bib-MSE-REGISTRY)中的[byte stream format specifacations](http://www.w3.org/TR/media-source/#byte-stream-format-specs)文档包含特定格式的用例。 

####Media Segment（媒体分片）
一段封装的、具有时间戳的一部分媒体数据，媒体分片跟最先添加的初始化分片相关联。

>在字节流格式登记文档[MSE-REGISTRY](http://www.w3.org/TR/media-source/#bib-MSE-REGISTRY)中的[byte stream format specifacations](http://www.w3.org/TR/media-source/#byte-stream-format-specs)文档包含特定格式的用例。

####MediaSource object URL（MeDiaSource对象URL）
MediaSource对象的URL是一个Blob类型[[FILE API](http://www.w3.org/TR/FileAPI/#url)]的URI，用MediaSource的createObjectURL()方法创建，它用来将MediaSource对象绑定到一个HTMLMediaElement.    

这些URLs和Blob URI一样，所不同的是，它定义的不仅适用于File和Blob对象，还扩展到适用于MediaSource对象。

MediaSource对象的URL的来源是[effective script origin](http://www.w3.org/TR/html5/browsers.html#effective-script-origin)调用的[createObjectURL()](http://www.w3.org/TR/media-source/#widl-URL-createObjectURL-DOMString-MediaSource-mediaSource).  

>比如，MediaSource对象的URL的来源影响HTML5媒体标签[consumed by canvas](http://www.w3.org/TR/html5/scripting-1.html#security-with-canvas-elements)的方式。  

####Parent Media Source（父MediaSource对象）
创建SourceBuffer对象的那个MediaSource对象是该SourceBuffer对象的父MediaSource对象。

####Presentation Start Time（播放开始时间）  
开始时间是播放开始的最早时间点，并指定[initial playback position](http://www.w3.org/TR/html5/embedded-content-0.html#initial-playback-position)和[earliest possible position](http://www.w3.org/TR/html5/embedded-content-0.html#earliest-possible-position).而且用此规范的创建的媒体的开始时间为0。  

####Presentation Interval（显示间隔）
一个编码帧的显示间隔是从它的显示时间戳开始，到其加上编码帧长。比如，如果一个编码帧的显示时间戳是10s，编码帧长是100ms，那么显示间隔是[10s-10.1s]。注意这段区间的开始是包含在内的，而结尾是排除在外的。  

####Presentation Order（显示顺序）
编码帧在展示时的顺序。先后顺序的实现是由编码帧的显示时间戳的大小来实现的。  

####Presentation Timestamp（显示时间戳）
代表一个媒体展示时的一个具体时刻。一个编码帧显示时间戳表示何时帧应该呈现。  

####Random Access Point（随机访问点）
指任意媒体分片的任意一个位置，当指定该位置后，媒体可以从该位置继续解码和播放，不需要依赖之前分段的任何数据。对于视频，指第i帧的位置。对于音频，大多音频帧可以被当做随机访问点。因为视频轨道倾向于具有稀疏分布的随机接入点。这些点的位置通常被认为多轨道媒体流的随机访问点。  

####SourceBuffer byte stream format specification（SourceBuffer字节流格式规范）
[byte stream format specifacations](http://www.w3.org/TR/media-source/#byte-stream-format-specs)描述了SourceBuffer实例可以接受的流格式。通过addSourceBuffer()方法，选定满足[byte stream format specifacations](http://www.w3.org/TR/media-source/#byte-stream-format-specs)的格式，创建SourceBuffer对象。  

####SourceBuffer configuration（SourceBuffer配置）
创建MediaSource的实例SourceBuffer对象。而一组特定的轨道分布在一个或者多个SourceBuffer对象中。  
实现必须具有至少一个MediaSource对象，并需要以下配置。  

*   一个SourceBuffer对象加载一个音频或视频轨道；
*   两个SourceBuffer，一个加载一个音频轨道，另一个SourceBuffer加载一个视频轨道。

####Track Description（轨道描述）
一个字节流格式的特定的结构，包含轨道ID，编解码器的配置，和每个轨道的元配置。每个轨道的描述包括一个初始化分片，和唯一的轨道ID。如果在初始化分片时，轨道ID不唯一，用户代理必须运行append error算法并将decode error属性设置为true。  

####Track ID（轨道ID）
轨道ID是一个轨道的标示符。轨道ID在轨道描述里识别媒体分片属于哪个轨道。  

##2. MediaSource对象

MediaSource对象表示HTMLMediaElement的媒体数据源。它具有一个SourceBuffer对象列表，可以用来添加媒体数据用来播放。MediaSource对象被web应用所创建，然后连接到一个HTMLMediaElement上。引用通过SourceBuffer列表中的SourceBuffer对象添加媒体数据到源中。当需播放这些媒体数据时，HTMLMediaElement从MediaSource对象中获取这些媒体数据。

```
WebIDL
enum ReadyState {
    "closed",
    "open",
    "ended"
};
```

|枚举描述         |               |
|:-------------:|:-------------:|
| closed        | 表明MediaSource没有与媒体标签相关联。 |
| open          | MediaSource被媒体标签打开，可以创建SourceBuffer并向其中添加数据     |
| ended         | MediaSource依旧跟数据标签相关联，但是endOfStream()方法触发了      |

```
WebIDL
enum EndOfStreamError {
    "network",
    "decode"
};
```

|枚举描述         |               |   
|:-------------:|:-------------:|
| network       | 播放终止，是网络出现错误的标志。 |
| decode        | 播放终止，是解码出现错误的标志。     |

```
[Constructor]
interface MediaSource : EventTarget {
    readonly attribute SourceBufferList    sourceBuffers;
    readonly attribute SourceBufferList    activeSourceBuffers;
    readonly attribute ReadyState          readyState;
             attribute unrestricted double duration;
    SourceBuffer   addSourceBuffer (DOMString type);
    void     removeSourceBuffer (SourceBuffer sourceBuffer);
    void     endOfStream (optional EndOfStreamError error);
    static boolean isTypeSupported (DOMString type);
};
```

###2.1 属性

####activeSourceBuffers

>类型：SourceBufferList
>只读  

是sourceBuffers的子集，包含[enabled](http://www.w3.org/TR/html5/embedded-content-0.html#dom-audiotrack-enabled) [audioTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-audiotracks),  [selected](http://www.w3.org/TR/html5/embedded-content-0.html#dom-videotrack-selected) [videoTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-videotracks)和 "[showing](http://www.w3.org/TR/html5/embedded-content-0.html#dom-texttrack-showing)" 或者 "[hidden](http://www.w3.org/TR/html5/embedded-content-0.html#dom-texttrack-hidden)" [textTracks](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-texttracks).

在这个列表里的SourceBuffer对象必须跟sourceBuffers属性里的对象顺序相一致。（比如，假如只有sourceBuffers[0]和sourceBuffers[3]在activeSourceBuffers里，那么activeSourceBuffers[0]对应sourceBuffers[0],activeSourceBuffers[1]对应sourceBuffers[3]）。

>在这Changes to selected/enable track state章节描述了该属性是如何更新的。

####duration

>类型：[unrestricted double](http://dev.w3.org/2006/webapi/WebIDL/#idl-unrestricted-double)

允许web应用设置播放的持续时间，在MediaSource对象创建的时候，duration的初始值设置为NaN.
获取时，执行以下步骤：

*   如果readystate属性为”closed”,那么返回NaN。
*   否则返回duration的当前值。
设置时，执行以下步骤
*   如果设置的值无效或NaN，则会抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror)，并终止以下步骤。
*   如果readystate属性不是”open”，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果sourceBuffers属性中每个SourceBuffer对象的updating属性均为true，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   执行[duration change算法](http://www.w3.org/TR/media-source/#duration-change-algorithm)，为duration设定新分配的值。

>在某些情况下，appendBuffer(),appendStream(),endOfStream()可能会更新duration的值。

####readyState
>类型：ReadyState
>只读

指示MediaSource当前的状态，当MediaSource对象刚创建时，readyState的值为false.

####SourceBuffers
>类型：SourceBufferList
>只读

包括跟该MediaSource相关联的所有的SourceBuffer对象。当readyState的值为"false"时，这个列表为空。一旦readyState的为"open"状态，SourceBuffer对象可以添加到该列表通过addSourceBuffer().

###2.2 方法

####addSourceBuffer

添加一个SourceBuffer到sourceBuffers里

返回类型：SourceBuffer

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| type         | DOMString    |     ✘        |        ✘     |              |

当调用该方法时，用户代理必须执行以下步骤：

*   如果参数type是空字符串，则会抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror)，并终止以下步骤。
*   如果type包括一个MIME类型但是不被创建的SourceBuffer支持，则会抛出异常[NotSupportedError](http://www.w3.org/TR/html5/infrastructure.html#notsupportederror)，并终止以下步骤。
*   如果用户代理不能处理更多的SourceBuffer对象或者当创建type类型的SourceBuffer会导致一个不支持的SourceBuffer结构，会抛出异常[QuotaExceededError](http://www.w3.org/TR/html5/infrastructure.html#quotaexceedederror)，并终止以下步骤。
>比如，当readyState为[HAVE_MEDIADATA](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-have_metadata)时，此时媒体数据到达，但可能用户代理不支持在播放时添加更多的轨道，则会抛出异常[QuotaExceededError](http://www.w3.org/TR/html5/infrastructure.html#quotaexceedederror).
*   如果属性readyState不是"open"状态,则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   新建一个新的SourceBuffer对象和相关联的资源
*   将新建对象的generate timestamps flag的值设置成与参数type对应的[MSE-REGISTRY](http://www.w3.org/TR/media-source/#bib-MSE-REGISTRY)中的"Generate Timestamps Flag"值。
*   如果generate timestamps flag等于true,将属性mode设置为"sequence",否则，设置为"segments".
*   将新建的对象添加到属性sourceBuffers，并[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffers中的addSourceBuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   返回新建的对象

####endOfStream

发出终止流的信号

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| error         | EndOfStreamError    |     ✘      |   ✘  |              |

返回类型：void

当调用该方法时，用户代理必须执行以下步骤：
*   如果readyState不是"open"状态，则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果sourceBuffers中所有的SourceBuffer对象的updating属性等于"true",则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   执行end of stream算法并将error参数设置成error

####isTypeSupported,static

判断MediaSource是否可以为指定的MIME type创建SourceBuffer对象。

>如果这个方法返回true，它仅仅表明MediaSource可以为指定的MIME type创建SourceBuffer对象，但是若大量的媒体不支持创建额外的SourceBuffer对象，执行addSourceBuffer()时仍然可能出错。
>如果这个方法返回true，意味着HTMLMediaElement.canPlayType()可能返回 "maybe" or "probably"，因为MediaSource支持一个类型，但HTMLMediaElement却不能播放是没有意义的。

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| type         | DOMString    |     ✘        |        ✘     |              |

返回类型：boolean

当调用该方法时，用户代理必须执行以下步骤：

*   如果参数type是空字符串，则返回false.
*   如果type不包含一个合理的MIME type字符串，则返回false.
*   如果type包含MediaSource不支持的媒体类型或者子类型，则返回false.
*   如果type包含MediaSource不支持的编码格式，则返回false.
*   如果MediaSource不支持特定的媒体类型、子类型、编码组合，则返回false.
*   返回true.

####removeSourceBuffer
从MediaSource中移除一个SourceBuffer

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| sourceBuffer | SourceBuffer |     ✘        |        ✘     |              |

返回类型：void

当调用该方法时，用户代理必须执行以下步骤：

*   如果指定的sourceBuffer不在sourceBuffers中，则抛出异常[NotFoundError]，并终止以下步骤。
*   如果sourceBuffer的updating属性为true,执行以下步骤
    -   如果buffer append和stream append loop算法还在执行，将其终止。
    -   将sourceBuffer的updating属性置成false.
    -   在[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的abort[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
    -   在[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的updateend[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   让SourceBuffer audioTracks list等于sourceBuffer.audioTracks返回的AudioTrackList对象。
*   如果SourceBuffer audioTracks list不空，执行以下步骤
    -   让HTMLMediaElement audioTracks list等于HTMLMediaElement中属性audioTracks返回的[AudioTrackList](http://www.w3.org/TR/html5/embedded-content-0.html#audiotracklist)对象。
    -   将removed enabled audio track flag设置为false.
    -   对于SourceBuffer audioTracks list中的每个AudioTrack对象，执行以下步骤
        +   将[AudioTrack](http://www.w3.org/TR/html5/embedded-content-0.html#audiotrack)对象的sourceBuffer属性置空。
        +   如果AudioTrack对象中的enabled属性为true,那么将removed enabled audio track flag设置为true.
        +   移除HTMLMediaElement audioTracks list中的AudioTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement audioTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
        +   移除SourceBuffer audioTracks list中的AudioTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)SourceBuffer audioTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
    -   如果removed enabled audio track flag等于true,[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement audioTracks list中的[change](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onchange) [事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   让SourceBuffer videoTracks list等于sourceBuffer.videoTracks返回的VideoTrackList对象。
*   如果SourceBuffer videoTracks list不空，执行以下步骤
    -   让HTMLMediaElement videoTracks list等于HTMLMediaElement中属性videoTracks返回的[VideoTrackList](http://www.w3.org/TR/html5/embedded-content-0.html#videotracklist)对象。
    -   将removed enabled audio track flag设置为false.
    -   对于SourceBuffer audioTracks list中的每个AudioTrack对象，执行以下步骤
        +   将[VideoTrack](http://www.w3.org/TR/html5/embedded-content-0.html#videotrack)对象的sourceBuffer属性置空。
        +   如果VideoTrack对象中的selected属性为true,那么将removed enabled video track flag设置为true.
        +   移除HTMLMediaElement videoTracks list中的VideoTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement videoTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
        +   移除SourceBuffer videoTracks list中的VideoTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)SourceBuffer videoTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
    -   如果removed enabled video track flag等于true,[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement videoTracks list中的[change](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onchange) [事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   让SourceBuffer textTracks list等于sourceBuffer.textTracks返回的TextTrackList对象。
*   如果SourceBuffer textTracks list不空，执行以下步骤
    -   让HTMLMediaElement textTracks list等于HTMLMediaElement中属性textTracks返回的[TextTrackList](http://www.w3.org/TR/html5/embedded-content-0.html#texttracklist)对象。
    -   将removed enabled text track flag设置为false.
    -   对于SourceBuffer textTracks list中的每个TextTrack对象，执行以下步骤
        +   将[TextTrack](http://www.w3.org/TR/html5/embedded-content-0.html#texttrack)对象的sourceBuffer属性置空。
        +   如果TextTrack对象中的mode属性为true,那么将removed enabled text track flag设置为true.
        +   移除HTMLMediaElement textTracks list中的TextTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement textTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
        +   移除SourceBuffer textTracks list中的VideoTrack对象。
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)SourceBuffer textTracks list中的[removetrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onremovetrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。
    -   如果removed enabled text track flag等于true,[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)HTMLMediaElement textTracks list中的[change](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onchange) [事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   如果activeSourceBuffers中存在sourceBuffer,那么移除之，并[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffers返回的SourceBufferList中的removesourcebuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   移除sourceBuffers中的sourceBuffer,并[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffers返回的SourceBufferList中的removesourcebuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   释放sourceBuffer的所有资源。

###2.3 事件总结

|事件名称        |     接口     |   触发时间     |
|:------------:|:------------:|:------------:|
| sourceopen   | Event        |     readyState属性从"closed"变成"open"或从"ended"变成"open"|
| sourceended  | Event        |     readyState属性从"open"变成"ended"|
| sourceclose   | Event        |     readyState属性从"open"变成"closed"或从"ended"变成"closed"|

###2.4 算法

####2.4.1 连接到媒体标签(Attaching to a media element)

一个MediaSource对象可以连接到媒体标签，通过分配一个MediaSource object URL到媒体标签的src属性，一个MediaSource object URL通过MediaSource对象的createObjectURL()创建。

如果[resource fetch算法](http://www.w3.org/TR/html5/embedded-content-0.html#concept-media-load-resource)的绝对路径与MediaSource object URL相匹配，那么在执行resource fetch算法中的"Perform a potentially CORS-enabled fetch"步骤前，执行以下步骤。

*   如果readyState不是"closed"，执行[resource fetch算法](http://www.w3.org/TR/html5/embedded-content-0.html#concept-media-load-resource)中的 "If the media data cannot be fetched at all, due to network errors, causing the user agent to give up trying to fetch the resource"步骤。
*   否则
    -   将readyState置为"open".
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)MediaSource中的sourceopen[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
    -   继续执行[resource fetch算法](http://www.w3.org/TR/html5/embedded-content-0.html#concept-media-load-resource)中的"Perform a potentially CORS-enabled fetch"步骤，将[resource fetch算法](http://www.w3.org/TR/html5/embedded-content-0.html#concept-media-load-resource)中指向"the download"或者"bytes received"的文字委托到apendBuffer()和appendStream()获得的数据。

####2.4.2 与媒体标签分离(Detaching from a media element)

当媒体标签即将过渡为[NETWORK_EMPTY](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-network_empty),并[触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)媒体标签中的[emptied](http://www.w3.org/TR/html5/embedded-content-0.html#event-mediacontroller-emptied) [事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。时间以下步骤将会在任意情况下执行。

*   将属性readyState置为"closed".
*   将属性duration更新为NaN.
*   删除activeSourceBuffers里所有的SourceBuffer对象。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)activeSourceBuffers中的removesourcebuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   删除sourceBuffers里所有的SourceBuffer对象。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffers中的removesourcebuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)MediaSource中的sourceclose[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。

####2.4.3 随机定位(Seeking)

执行以下几个步骤，并作为[seek算法](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-seek)中"Wait until the user agent has established whether or not the media data for the new playback position is available, and, if it is, until it has decoded enough data to play back that position"步骤的一部分。

*   随机定位进度条时，需要的媒体分段包含新的播放位置。
    -   如果一个或者activeSourceBuffers中的对象缺少指定位置的媒体分段：
        +   如果HTMLMediaElement.readyState属性比HAVE_METADATA大，那么设置HTMLMediaElement.readyState为HAVE_METADATA。
        +   媒体标签等待appendBuffer()或者appendStream()执行，然后使用coded frame processing算法使得HTMLMediaElement.readyState比HAVE_METADATA大。
    -   否则继续。
*   媒体标签重设所有解码器和从initialization segment的合适位置初始化所有数据。
*   媒体标签从active track buffers中找到新的播放位置最近的随机访问点进行编码。
*   继续执行seek算法中的"Await a stable state"步骤。

####2.4.4 SourceBuffer Monitoring
####2.4.5 Changes to selected/enabled track state
####2.4.6  Duration change
####2.4.7 End of stream algorithm

##3. SourceBuffer对象

```
WebIDL
enum AppendMode {
    "segments",
    "sequence"
};
```

|枚举描述         |               |
|:-------------:|:-------------:|
| segments        | 媒体分片中的时间戳决定编码帧播放的位置，媒体分片可以无序添加 |
| sequence       | 媒体分片被当做是邻接的时间独立的片段，新媒体分片需要紧随先前媒体分配的编码帧。如果需要添加新的媒体分片，设定timestampOffset属性为"sequence"模式，允许媒体分片按照时间线的特定顺序添加，不需要知道每个媒体分片的时间戳信息 |

```
WebIDL
interface SourceBuffer : EventTarget {
                attribute AppendMode          mode;
    readonly    attribute boolean             updating;
    readonly    attribute TimeRanges          buffered;
                attribute double              timestampOffset;
    readonly    attribute AudioTrackList      audioTracks;
    readonly    attribute VideoTrackList      videoTracks;
    readonly    attribute TextTrackList       textTracks;
                attribute double              appendWindowStart;
                attribute unrestricted double appendWindowEnd;
    void appendBuffer (ArrayBuffer data);
    void appendBuffer (ArrayBufferView data);
    void appendStream (ReadableStream stream, [EnforceRange] optional unsigned long long maxSize);
    void abort ();
    void remove (double start, unrestricted double end);
                attribute TrackDefaultList    trackDefaults;
};
```

###3.1 属性

####appendWindowEnd

>类型：unrestricted double

添加窗口的终止时间戳，初始化为正无穷。

获取该属性时，返回正无穷或者最后设置的值。

设置时，执行以下几个步骤：

*   如果该属性已经从父MediaSource对象的sourceBuffers属性中移除，则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror),并终止以下步骤。
*   如果updating属性为true,则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror),并终止以下步骤。
*   如果new value为NaN，则抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror),并终止以下步骤。
*   如果new value小于等于appendWindowStart,则抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror),并终止以下步骤。
*   更新属性值为新值。

####appendWindowStart

>类型：double

添加窗口的开始时间戳，初始化为播放开始时间。

获取该属性时，返回初始值或者最后设置的值。

设置时，执行以下几个步骤：

*   如果该属性已经从父MediaSource对象的sourceBuffers属性中移除，则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror),并终止以下步骤。
*   如果updating属性为true,则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror),并终止以下步骤。
*   如果new value小于0或者大于appendWindowEnd,则抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror),并终止以下步骤。
*   更新属性值为新值。

####audioTracks

>类型：AudioTrackList
>只读

AudioTrack对象的列表通过此对象创建。

####buffered

>类型：TimeRanges
>只读

指示SourceBuffer已经缓冲的[时间范围](http://www.w3.org/TR/html5/embedded-content-0.html#timeranges)。对象刚建立时，该属性为一个空的TimeRanges对象。

当该对象被读的时候，需要执行以下步骤：

*   如果该属性已经从父MediaSource对象的sourceBuffers属性中移除，则抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror),并终止以下步骤。
*   让highest end time为所有SourceBuffer的track buffers范围中最大的终止时间。
*   让intersection ranges等于一个[TimeRange](http://www.w3.org/TR/html5/embedded-content-0.html#timeranges)对象，包含一个单一的从0到highest end time的范围。
*   对于SourceBuffer管理的每一个track buffer，执行以下步骤：
    -   让track ranges等于当前track buffer的track buffer ranges.
    -   如果readyState等于"ended",那么设置最后一个track ranges的终止时间为highest end time.
    -   让new intersection ranges等于intersection ranges和track ranges的交接范围。
    -   让new intersection ranges代替intersection ranges中的范围。
*   如果intersection ranges的值和当前属性表示的范围不一致，那么将该属性的值赋给intersection ranges.
*   返回当前属性的值。

####mode

>类型：AppendMode

控制处理媒体分片的方式，当对象初始化之后，通过addSourceBuffer()方法初始化该属性。

获取该属性时，返回初始值或者最后设置的值。

设置时，执行以下几个步骤：

*   如果该对象已经在父MediaSource对象的sourceBuffers中移除，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果updating属性等于true,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   让new mode等于分配给其的新值。
*   如果generate timestamps flag等于true,而且new mode等于"segments",则会抛出异常[InvalidAccessError](http://www.w3.org/TR/html5/infrastructure.html#invalidaccesserror)，并终止以下步骤。
*   如果父MediaSource对象的readyState属性为"ended",执行以下步骤：
    -   将该readyState设置为"open".
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)父MediaSource对象中的sourceopen[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   如果append state等于PARSING_MEDIA_SEGMENT,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果new mode等于"sequence",那么设置group start timestamp为group end timestamp.
*   更新属性为new mode值。

####textTracks

>类型：TextTrackList
>只读

TextTrack对象的列表被该对象创建。

####timestampOffset

>类型：double

控制接下来需要添加到SourceBuffer中的媒体分段的时间偏移量，初始化为0。

获取该属性时，返回初始值或者最后设置的值。

设置时，执行以下几个步骤：

*   让new timestamp值等于设置的新值。
*   如果该对象已经在父MediaSource对象的sourceBuffers中移除，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果updating属性等于true,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果父MediaSource对象的readyState属性为"ended",执行以下步骤：
    -   将该readyState设置为"open".
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)父MediaSource对象中的sourceopen[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   如果append state等于PARSING_MEDIA_SEGMENT,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果new mode等于"sequence",那么设置group start timestamp为group end timestamp.
*   更新属性为new timestamp值。

####trackDefaults

>类型：TrackDefaultList

当initialization segment received算法需要创建track对象时，如果initialization segment的种类、标签或者语言等信息不可用，然后可以用该属性的默认轨道。初始化为空的TrackDefaultList对象。

获取该属性时，返回初始值或者最后设置的值。

设置时，执行以下几个步骤：

*   如果该对象已经在父MediaSource对象的sourceBuffers中移除，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果updating属性等于true,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   更新属性为新的值。

####updating

>boolean
>只读

表明apendBuffer(),apendStream(),或者remove()操作是否仍在异步执行，SourceBuffer对象创建时，初始化false.

videoTracks

>VideoTrackList
>只读

videoTrack对象的列表被该对象创建。

###3.2 方法

####abort

终止当前分段，并重置分段分析器。

没有参数。

返回类型：void

当调用该方法时，用户代理必须执行以下步骤：

*   如果该对象已经在父MediaSource对象的sourceBuffers中移除，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果readystate属性不是”open”，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果updating属性等于true,执行以下步骤：
    -   如果stream append loop算法和buffer append正在执行，终止之。
    -   设置updating属性为true.
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)此SourceBuffer对象中的abort[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)此SourceBuffer对象中的updateend[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   执行reset parser state算法.
*   设置appendWindowStart为播放开始时间。
*   设置appendWindowEnd为正无穷。

####appendBuffer

添加ArrayBuffer格式的分段到sourceBuffer.

此方法的这个步骤与ArrayBufferView版本的appendBuffer()相同。

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| data         | ArrayBuffer    |     ✘        |        ✘     |              |

返回类型：void

####appendBuffer

添加ArrayBufferView格式的分段到sourceBuffer.

|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| data         | ArrayBufferView    |     ✘      |      ✘     |            |

返回类型：void

当调用该方法时，用户代理必须执行以下步骤：

*   执行prepare append算法.
*   添加数据到input buffer的结尾。
*   设置updating属性为true.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)此SourceBuffer对象中的updatestart[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   异步执行buffer append算法.

####appendStream

添加ReadableStream格式的分段到sourceBuffer。
|      参数     |     类型     |   可否为空     |  其他选项      |     描述      |
|:------------:|:------------:|:------------:|:------------:|:------------:|
| stream         | ReadableStream    |     ✘      |      ✘     |            |
|    maxSize       | unsigned long long    |     ✘      |      ✔     |            |

返回类型：void
当调用该方法时，用户代理必须执行以下步骤：

*   执行prepare append算法.
*   设置updating属性为true.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)此SourceBuffer对象中的updatestart[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   以参数stream和MaxSize异步执行stream append loop算法。

####remove

移除指定范围的媒体数据。
i.w3.org/TR/html5/webappapis.html#queue-a-task)父MediaSource对象中的sourceopen[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   执行range removal算法,start和end表示移除的范围。

###3.3 轨道缓冲器

一个轨道缓冲器存储单个轨道的轨道描述和编码帧。通过添加initialization segments和media segments到SourceBuffer里更新轨道缓冲器。

每一个轨道缓冲器具有一个last decode timestamp变量表示加入到当前编码帧组的最后编码帧的解码时间戳，该变量初始化没有设定，表示没有编码帧被加入。

每一个轨道缓冲器具有一个last frame duration变量表示加入到当前编码帧组的最后编码帧的编码帧长。该变量初始化没有设定，表示没有编码帧被加入。

每一个轨道缓冲器具有一个highest presentation timestamp变量表示加入到当前编码帧组的编码帧的最大显示时间戳。该变量初始化没有设定，表示没有编码帧被加入。

每一个轨道缓冲器具有一个need random access point flag变量表示该轨道是否等待一个随机播放点的编码帧，该属性初始化为true,表示在任意可以加到轨道缓冲器的编码帧前，需要随机访问点的编码帧。

每一个轨道缓冲器具有一个track buffer ranges变量表示加入到轨道缓冲器的展示时间范围。出于规范目的，这些信息都被看作是一个[规范化的TimeRanges对象](http://www.w3.org/TR/html5/embedded-content-0.html#normalized-timeranges-object)。

###3.4 事件总结

|事件名称        |     接口     |   触发时间     |
|:------------:|:------------:|:------------:|
| updatestart   | Event       |     updating从false变成true|
| update  | Event        |添加或者移除操作成功完成后，updating从true变成false|
| updateend   |Event| 添加或者移除操作终止后        |
| error   | Event       | 在append.updating从false变成true时，出现了错误|
| abort   | Event       |添加或者移除操作被abort()终止，updating从true变成了false|

###3.5 算法

####3.5.1 分段循环分析(segment parser loop)

每一个SourceBuffer对象都有一个internal append state状态变量，表示分段分析的状态。初始化为WAITING_FOR_SEGMENT,而且在添加数据时可以转化为以下几个状态。

|Append state值|      描述              |
|:-------------:|:--------------------:|
| WAITING_FOR_SEGMENT | 等待添加初始化分段或者媒体分段 |
| PARSING_INIT_SEGMENT | 正在添加初始化分段 |
| PARSING_MEDIA_SEGMENT | 正在添加媒体分段 |

input buffer是一个字符缓冲区，用来暂存尚未分析的通过appendBuffer()或者appendStream()获取的字节流。当SourceBuffer对象创建的时候，该缓冲区为空。

buffer full flag标志位表示是否允许appendBuffer()或者appendStream()添加更多字节流。当SourceBuffer对象创建的时候，该标志位为false,当数据添加或者移除时，该标志位会被修改。

group start timestamp变量表示在"sequence"模式中，一个新的编码帧组的开始时间戳，当SourceBuffer对象创建时，该变量没有设定，当"mode"设定为"sequence"而且"timestampOffset"属性设定时，或者coded frame processing算法执行时，该属性被更新。

group end timestamp变量表示在当前编码帧组中所有的编码帧中最大的结束时间戳，当SourceBuffer对象创建时，该变量初始化为0,而且在coded frame processing算法中更新。
>group end timestamp表示的是SourceBuffer中所有的轨道中最大的结束时间戳，因此，当加入混合的未设定时间戳的媒体分段时，应当注意"mode"的设定。

generate timestamps flag是一个boolean类型的变量，通过coded frame processing算法,它表示编码帧的时间戳是否应该更新，该变量由addSourceBuffer()创建SourceBuffer对象完毕后创建。

当分段循环分析算法触发时，用户代理必须执行以下步骤：

*   循环开始：当input buffer为空，转向下面的need more data步骤。
*   如果input buffer中的字节流违背了[SourceBuffer byte stream format specification](http://www.w3.org/TR/media-source/#sourcebuffer-byte-stream-format-spec)中的规定，那么执行append error算法,并将参数decode error设定为true,并终止此算法。
*   移除[byte stream format specifications]()规定的必须忽略的input buffer中的头部信息。
*   如果append state等于WAITING_FOR_SEGMENT,那么执行以下步骤。
    -   如果input buffer中的头部指示的是初始化分段的开始，那么设定append state为PARSING_INIT_SEGMENT.
    -   果input buffer中的头部指示的是媒体分段的开始，那么设定append state为PARSING_MEDIA_SEGMENT.
    -   转向上面的循环开始。
*   如果append state等于PARSING_INIT_SEGMENT,那么执行以下步骤。
    -   如果input buffer么有包含所有的初始化分段，那么跳转到下面的need more data步骤。
    -   执行initialization segment received算法.
    -   移除input buffer中的初始化分段字节流。
    -   设定append state为WAITING_FOR_SEGMENT。
    -   转向上面的循环开始。
*   如果append state等于PARSING_MEDIA_SEGMENT,那么执行以下步骤。
    -   如果first initialization segment received flag等于false,那么执行append error算法,并将参数decode error设定为true,并终止此算法。
    -   如果input buffer没有包含完整的媒体分段的头部，那么跳转到下面的need more data步骤。
    -   如果input buffer没有包含一个或者多个完整的编码帧，那么执行coded frame processing算法.
    -   如果该SourceBuffer已经满了，不能接收更多的媒体数据，那么设定buffer full flag为true.
    -   如果input buffer没有包含完整的媒体分段，那么跳转到下面的need more data步骤。
    -   移除input buffer开始的媒体分段。
    -   设定append state为WAITING_FOR_SEGMENT.
    -   跳转到上边的WAITING_FOR_SEGMENT.
*   Need more data:返回控制调用的算法。

####3.5.2 重置分析状态(Reset Parser State)

当分析状态需要重置的时候，执行以下步骤：

*   如果append state等于PARSING_MEDIA_SEGMENT，并且input buffer包含完整的编码帧，那么执行coded frame processing算法直到所有的编码帧被处理。
*   复原所有轨道缓冲器的last decode timestamp.
*   复原所有轨道缓冲器的last frame duration.
*   复原所有轨道缓冲器的highest presentation timestamp.
*   设定所有轨道缓冲器的need random access point flag为true.
*   移除input buffer中所有字节流。
*   设定append state为WAITING_FOR_SEGMENT.

####3.5.3 添加错误算法(Append Error Algorithm)

当添加媒体时出现错误时，该算法被调用。该算法具有一个decode error参数，表示endOfStream()是否应该被调用。

*   执行reset parser state算法.
*   将updating属性设定为false.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的error[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的error[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   如果decode error是true,然后设定error参数为"decode"并执行end of stream算法.

####3.5.4 添加准备算法(Prepare Append Algorithm)

当一个添加操作开始时，以下步骤将会执行，使SourceBuffer生效。

*   如果该对象已经在父MediaSource对象的sourceBuffers中移除，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果updating属性为true,则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果[HTMLMediaElement.error](http://www.w3.org/TR/html5/embedded-content-0.html#dom-media-error)属性不为空，则会抛出异常[InvalidStateError](http://www.w3.org/TR/html5/infrastructure.html#invalidstateerror)，并终止以下步骤。
*   如果父MediaSource对象的readyState属性为"ended",执行以下步骤：
    -   将该readyState设置为"open".
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)父MediaSource对象中的sourceopen[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   执行coded frame eviction算法.
*   如果buffer full flag等于true,抛出异常[QuotaExceededError](http://www.w3.org/TR/html5/infrastructure.html#quotaexceedederror).

>这是一个信号，表示添加数据太多已经不允许添加更多数据，web应用应该使用remove()去释放一些空间，减少添加窗口的大小。

####3.5.5 添加缓冲算法(Buffer Append Algorithm)

当appendBuffer()执行时，执行以下步骤处理添加数据：

*   执行segment parser loop算法.
*   如果前边的segment parser loop算法被终止了，那么终止此算法。
*   将updating属性置为false.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的update[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的updateend[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。

####3.5.6 循环添加流算法(Stream Append Loop)

该算法在调用appendStream()时触发，将ReadableStrem转化为SourceBuffer,该算法随appendStream的两个参数stream和maxSize初始化执行。

*   如果设定了maxSize,那么设定bytesLeft为maxSize.
*   循环开始：如果设定了maxSize,并且bytesLeft等于0,那么跳转到下边的loop done步骤。
*   如果stream.state等于"waiting",那么执行以下步骤：
    -   等待stream.ready或者stream.closed确定解决或者拒绝。
    -   继续以下步骤。
*   如果stream.state等于"closed",那么跳转到下边的循环结束步骤。
*   如果stream.state等于"errored",那么执行append error算法,参数decode error设定为false,并终止此算法。
*   如果stream.state不等于"readable",那么执行append error算法,参数decode error设定为false,并终止此算法。
*   当data等于stream.read()返回的值。
*   如果data不是一个ArrayBuffer或者一个ArrayBufferView,那么执行append error算法,参数decode error设定为false,并终止此算法。
*   如果设定了maxSize,那么执行以下步骤：
    -   如果data.byteLength比bytesLeft大，那么执行以下步骤：
        +   让new data等于data.slice(0, bytesLeft)返回的值。
        +   让remaining data等于data.slice(bytesLeft)返回的值。
        +   让remaining data推到stream的头部，因此它可以被stream.read()调用。
        +   将new data分配给data.
    -   从byteLeft中减去data.byteLength.
*   执行coded frame eviction算法.
*   如果buffer full flag等于true,那么执行append error算法,参数decode error设定为false,并终止此算法。
*   添加数据到input buffer的末尾。
*   执行segment parser loop算法。
*   如果前一步的segment parser loop算法终止，那么终止此算法。
*   跳转到上边的循环开始步骤。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的update[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的updateend[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。

####3.5.7 移除缓冲范围(Range Removal)

当调用一个显式范围移除操作并阻碍其它SourceBuffer更新时，执行以下步骤：

*   让start等于待移除范围的开始时间戳。
*   让end等于待移除范围的结束时间戳。
*   让updating属性等于true.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的updatestart[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   返回给调用者，并异步执行以下步骤。
*   使用start和end作为移除的起始和结束范围执行coded frame removal算法。
*   让updating属性等于false.
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的update[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)sourceBuffer中的updateend[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。

####3.5.8 初始化分段接收完毕(Initialization Segment Received)

当segment parser loop算法成功接收一段完整的初始化分段(initialization segment)时，执行以下步骤：

*   每一个SourceBuffer对象都有一个internal first initialization segment received标志，表示初始化分段是否被该算法添加或者接收。当SourceBuffer创建后，该标志设置为false,并在以下步骤中被更新。
*   如果duration等于NaN,那么更新该属性：
    -   如果初始化分段包含一个duration,那么设定new duration为初始化分段中的duration,并执行duration change算法。
    -   否则，将duration设置为infinity,并执行duration change算法。
*   如果初始化分段没有音频，视频或者文本轨道，那么执行append error算法,并将参数decode error设定为true,并终止此算法。
*   如果first initialization segment received flag为true,那么执行以下步骤：
    -   验证以下属性，如果有一项不符合，那么行append error算法,并将参数decode error设定为true,并终止此算法。
        +   音频，视频和文本轨道的数量与第一个初始化分段的个数相同。
        +   每个轨道的编解码器跟第一个初始化分段指定的相同。
        +   如果不止一个轨道类型存在(如两个音频轨道存在)，那么轨道ID需要跟第一个初始化分段中的的相对应。
    -   从初始化分段中提取轨道描述添加到每一个轨道缓冲器。
    -   设定所有轨道缓冲器的need random access point flag为true.
*   让active track flag等于true.
*   如果first initialization segment received flag为false,那么执行以下步骤：
    -   如果initialization segment中轨道的编码类型不被支持，那么执行append error算法,并将参数decode error设定为true,并终止此算法。
    >group end timestamp存储了一个SourceBuffer中所有轨道缓冲器中最大的coded frame end timestamp,因此，应该注意在添加多重媒体分段时mode属性的设定，以防轨道之间的时间戳不一致。
    -   对于初始化分段中的每个音频轨道，执行以下步骤：
        +   让audio byte stream track ID等于当前处理的轨道的轨道ID.
        +   让audio language为初始化分段中规定的BCP 47语言，或者如果没有语言需要呈现，置其为空字符串。
        +   如果audio language为空字符串或者为'und' BCP 47,那么执行default track language算法，并将byteStreamTrackID设置为audio byte stream track ID,将type设置为"audio",并让audio language等于该算法的返回值。
        +   让audio label为初始化分组中该轨道指定的label,或者如果没有label呈现，设置其为空字符串。
        +   如果audio label等于一个空字符串，那么执行default track label算法，并将byteStreamTrackID设置为audio byte stream track ID,将type设置为"audio",并让audio label等于该算法的返回值。
        +   让audio kinds为初始化分组中该轨道指定的kind字符串，或者如果没有提供kind信息，设置其为空字符串。
        +   如果audio kinds等于一个空数组，那么执行default track kinds算法，并将byteStreamTrackID设置为audio byte stream track ID,将type设置为"audio",并让audio kind等于该算法的返回值。
        +   对于audio kinds中的每一项，执行以下步骤：
            *   让current audio kind等于当前audio kinds迭代项。
            *   让new audio track为一个新的AudioTrack对象。
            *   生成一个唯一的ID,并将其赋给new audio track的id属性。
            *   将audio language赋给new audio track的language属性。
            *   将audio label赋给new audio track的label属性。
            *   将current audio kind赋给new audio track的kind属性
            *   如果audioTracks.length为0，那么执行以下步骤：
                -   让new audio track的enabled属性为true.
                -   让active track flag为true.
            *   添加new audio track到该SourceBuffer对象的audioTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了该SourceBuffer对象引用的AudioTrackList对象中的TrackEvent接口。
            *   添加new audio track到HTMLMediaElement的audioTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了HTMLMediaElement引用的AudioTrackList对象中的TrackEvent接口。
        +   创建一个新的轨道缓冲器储存该轨道的编码帧。
        +   为该轨道添加轨道描述，添加到轨道缓冲器。
    -   对于初始化分段中的每个视频轨道，执行以下步骤：
        +   让video byte stream track ID等于当前处理的轨道的轨道ID.
        +   让video language为初始化分段中规定的BCP 47语言，或者如果没有语言需要呈现，置其为空字符串。
        +   如果video language为空字符串或者为'und' BCP 47,那么执行default track language算法，并将byteStreamTrackID设置为video byte stream track ID,将type设置为"video",并让video language等于该算法的返回值。
        +   让video label为初始化分组中该轨道指定的label,或者如果没有label呈现，设置其为空字符串。
        +   如果video label等于一个空字符串，那么执行default track label算法，并将byteStreamTrackID设置为video byte stream track ID,将type设置为"video",并让video label等于该算法的返回值。
        +   让video kinds为初始化分组中该轨道指定的kind字符串，或者如果没有提供kind信息，设置其为空字符串。
        +   如果video kinds等于一个空数组，那么执行default track kinds算法，并将byteStreamTrackID设置为video byte stream track ID,将type设置为"video",并让video kind等于该算法的返回值。
        +   对于video kinds中的每一项，执行以下步骤：
            *   让current video kind等于当前video kinds迭代项。
            *   让new video track为一个新的VideoTrack对象。
            *   生成一个唯一的ID,并将其赋给new video track的id属性。
            *   将video language赋给new video track的language属性。
            *   将video label赋给new video track的label属性。
            *   将current video kind赋给new video track的kind属性
            *   如果videoTracks.length为0，那么执行以下步骤：
                -   让new video track的selected属性为true.
                -   让active track flag为true.
            *   添加new video track到该SourceBuffer对象的videoTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了该SourceBuffer对象引用的VideoTrackList对象中的TrackEvent接口。
            *   添加new video track到HTMLMediaElement的videoTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了HTMLMediaElement引用的VideoTrackList对象中的TrackEvent接口。
        +   创建一个新的轨道缓冲器储存该轨道的编码帧。
        +   为该轨道添加轨道描述，添加到轨道缓冲器。
    -   对于初始化分段中的每个文本轨道，执行以下步骤：
        +   让text byte stream track ID等于当前处理的轨道的轨道ID.
        +   让text language为初始化分段中规定的BCP 47语言，或者如果没有语言需要呈现，置其为空字符串。
        +   如果text language为空字符串或者为'und' BCP 47,那么执行default track language算法，并将byteStreamTrackID设置为text byte stream track ID,将type设置为"text",并让text language等于该算法的返回值。
        +   让text label为初始化分组中该轨道指定的label,或者如果没有label呈现，设置其为空字符串。
        +   如果text label等于一个空字符串，那么执行default track label算法，并将byteStreamTrackID设置为text byte stream track ID,将type设置为"text",并让text label等于该算法的返回值。
        +   让text kinds为初始化分组中该轨道指定的kind字符串，或者如果没有提供kind信息，设置其为空字符串。
        +   如果text kinds等于一个空数组，那么执行default track kinds算法，并将byteStreamTrackID设置为text byte stream track ID,将type设置为"text",并让text kind等于该算法的返回值。
        +   对于text kinds中的每一项，执行以下步骤：
            *   让current text kind等于当前text kinds迭代项。
            *   让new text track为一个新的TextTrack对象。
            *   生成一个唯一的ID,并将其赋给new text track的id属性。
            *   将text language赋给new text track的language属性。
            *   将text label赋给new text track的label属性。
            *   将current text kind赋给new text track的kind属性
            *   如果textTracks.length为0，那么执行以下步骤：
                -   让new text track的enabled属性为true.
                -   让active track flag为true.
            *   添加new text track到该SourceBuffer对象的textTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了该SourceBuffer对象引用的TextTrackList对象中的TrackEvent接口。
            *   添加new text track到HTMLMediaElement的textTracks属性。
            *   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)[addtrack](http://www.w3.org/TR/html5/embedded-content-0.html#handler-tracklist-onaddtrack)事件([trusted event](http://www.w3.org/TR/html5/infrastructure.html#concept-events-trusted) 不可撤销)。并且使用了HTMLMediaElement引用的TextTrackList对象中的TrackEvent接口。
        +   创建一个新的轨道缓冲器储存该轨道的编码帧。
        +   为该轨道添加轨道描述，添加到轨道缓冲器。
    -   如果active track flag等于true,那么执行以下步骤：
        +   添加该SourceBuffer到activeSourceBuffers.
        +   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)activeSourceBuffers中的addSourceBuffer[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
    -   设置first initialization segment received flag为true.
*   如果HTMLMediaElement.readyState属性为HAVE_NOTHING,那么执行以下步骤：
    -   如果sourceBuffers中有一个或者多个对象的 first initialization segment received flag设定为false,那么终止以下按步骤。
    -   设置HTMLMediaElement.readyState为HAVE_METADATA.
    -   [触发](http://www.w3.org/TR/html5/webappapis.html#queue-a-task)媒体标签中的loadedmetadata[事件](http://www.w3.org/TR/html5/webappapis.html#fire-a-simple-event)。
*   如果active track flag等于true并且HTMLMediaElement.readyState属性比HAVE_CURRENT_DATA大，那么设置HAVE_CURRENT_DATA为HAVE_METADATA.

#### 3.5.9 Default track language

#### 3.5.10 Default track label

#### 3.5.11 Default track kinds

#### 3.5.12 Coded Frame Processing

#### 3.5.13 Coded Frame Removal Algorithm

#### 3.5.14 Coded Frame Eviction Algorithm

#### 3.5.15 Audio Splice Frame Algorithm

#### 3.5.16 Audio Splice Rendering Algorithm

#### 3.5.17 Text Splice Frame Algorithm

