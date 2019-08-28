# android protobuff使用
说实话，想来总结这篇文章很久了，在还没有和```小团子芳芳```分手的时候，an系列```sailf```中，就提供了数据持久化的能力，序列化和反序列化用sailf很容易实现(具体在[an框架](https://zhuanlan.zhihu.com/sunst?author=qydq)专栏中)，当时还是2.0之前的一个版本，而

上篇介绍了[android跨进程通信-Messenger，AIDL使用](https://zhuanlan.zhihu.com/p/63333559)，不同进程间数据传递，更加深了想来总结本文的动力，为此查了网上很多资料，说实话，真没找到一个人，把protobuff介绍得"那个"，，ok，当然看完本篇，你就可以使用本篇的```protobuf```作为结构化数据的工具来完成不同进程间传递结构化数据了，本篇介绍的protobuf版本是基于最新```3.0```的，
>本篇的标题是```android protobuff的使用```，所以在人工智能专栏中还会有一篇文章是介绍```python的protobuff使用```，你们可以点赞和加关注

  文章归档：[ word 20180106] - ```ina系列Android树```3章节.markdown
## 版权声明CopyRight：
> **本内容作者：sunst，转载或引用请**[标明出处](https://www.zhihu.com/people/qydq)**，违者追究法律责任！！！**
## 一：protobuf简介 
>* [Google Protobuf官网click](https://developers.google.cn/protocol-buffers/)
>* [GitHub源代码托管click](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)

Protobuf全称：```Google Protocol Buffer```，是Google开源的一种轻便高效的**结构化数据存储格式**，可以用于结构化数据的串行化，也称作序列化，主要用于数据存储或是```RPC```数据交换，比较类似于XML，JSON；具有更快、更简单、更轻量级等特性。支持多种语言，只需定义好数据结构，利用Protobuf框架生成源代码，就可很轻松地实现数据结构的序列化和反序列化。

### 官方解释如下：
>Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages. You can even update your data structure without breaking deployed programs that are compiled against the "old" format.
 
**其内部数据是纯二进制格式，不依赖于语言和平台，目前用于序列化与反序列化官方支持的语言有C++，C#， GO， JAVA， PYTHON。适用于大小在1M以内的数据，在移动设备中，内存是很弥足珍贵的。**
  
### 1.为什么使用ProtoBuff？Why not just use XML or JSON?  
>Protocol buffers have many advantages over XML and JSON for serializing structured data.

**Protobuf具有以下优点**：（缺点是什么可读性啊）

因为p是二进制传输，位运算规则可以自己定义，所以不易被破解，安全是它的第一点，诸如此外，高效性、兼容性，多语言性，有代码生成机制等。

### （1）高效性
直接拿千里眼项目中跑出来的数据进行对比

**序列化时间效率对比**
![](https://github.com/qydq/anopen/blob/master/screen/protbuff_time.png?raw=true)
**序列化空间效率对比：**
![](https://github.com/qydq/anopen/blob/master/screen/protbuff_space.png?raw=true)

从上面的数据可以看出来，Protobuf序列化时，和JSON对比，不管在时间和空间上都是更加高效，对于XML来说
>are simpler ，are 3 to 10 times smaller ，are 20 to 100 times faster  ，are less ambiguous(含糊)  ，generate data access classes that are easier to use programmatically  

可以看到它的优点同样都是非常明显的，xml、json也是结构化数据，但使用protobuf表示的数据却能更加高效，并且将数据压缩得更小，后面我会继续探讨
 
### （2）支持向后兼容和向前兼容

当客户端和服务器同事使用一块协议的时候， 当客户端在协议中增加一个字节，并不会影响客户端的使用

### （3）支持多种编程语言

Google官方发布的源代码中包含了c++、java、Python三种语言

### 2.支持的数据类型  
https://developers.google.cn/protocol-buffers/docs/proto3  
  ![图1-支持的数据类型](https://github.com/qydq/anopen/blob/master/screen/protobuff1.png?raw=true)

protobuff支持 repeated map 复杂结构 ，根据数据常用范围，及最大范围确定使用类型，关于数据类型，后面会补充一篇-比较详细的关于Java中数据类型的理解
>可能本文后面不会在更新，你们可以点赞和加关注留意数据类型的详解
  
### 3.protobuf反序列化原理  
https://developers.google.cn/protocol-buffers/docs/encoding  
![protobuf反序列化原理](https://github.com/qydq/anopen/blob/master/screen/protobuff2.png?raw=true)
采用可变长int  ，循环读取tag value  
 >tag = field_id << 3| type  
  ```
selfInfo.deviceName = "device";  
0A06646576696365  
tag = 0A  
0000 1010  
field_id = 0000 1 = 1  
type = 0000 0 010 = 2 变长 Length-delimited  
value = byte[]{“646576696365”}  
```

## 二：配置
如果是在android studio应用层开发，需要gradle配置，对于系统应用级别的开发，配置相对应的mk，如下介绍两种参考

### 1.gradle配置
在项目的根目录下
```
dependencies {
	classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.10'  
}
```
修改需要支持protobuf的工程gradle (建议使用lite版本)  
```  
apply plugin: 'java-library'  
apply plugin: 'com.google.protobuf'  
buildscript {  
	repositories {  
		mavenCentral()  
	}  
	dependencies {  
		classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.10'  
	}
}  
  
protobuf {  
	protoc {  
		artifact = "com.google.protobuf:protoc:3.4.0"  
	}  
	plugins {  
		javalite {  
			artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'  
		}  
	}  
	generateProtoTasks {  
		all()*.builtins {  
		remove java  
		}  
		all()*.plugins {  
		javalite {}  
		//javanano {}  
		}  
	}  
}

jar {  
	manifest {  
		attributes 'Main-Class': 'com.sunst.lib.MyClass'  
	}  
	exclude '**.proto'  
}  
  
sourceSets {  
	main {  
		proto {  
		srcDir(["${protobuf.generatedFilesBaseDir}/main/javalite"])
		include  '**/*.proto'
		}  
}  
}  
  
repositories {  
	mavenCentral()  
}  
  
dependencies {  
implementation fileTree(dir: 'libs', include: ['*.jar'])
// 定义protobuf依赖，使用精简版
implementation 'com.google.protobuf:protobuf-lite:3.0.1'  
//https://mvnrepository.com/artifact/com.google.protobuf.nano/protobuf-javanano  
//implementation 'com.google.protobuf.nano:protobuf-javanano:3.1.0'  
implementation 'junit:junit:4.12'  
}  
clean {  
	delete "$projectDir/out"  
	delete protobuf.generatedFilesBaseDir  
}  
  
tasks.withType(JavaCompile) {  
	options.encoding = "UTF-8"  
}  
sourceCompatibility = "7"  
targetCompatibility = "7"  
```  
**PPS说明：**

* 如果工程的build.grade配置了，根目录下的build.gradle也可以不配置，
* `apply plugin: 'com.google.protobuf'`是Protobuf的Gradle插件，帮助我们在编译时通过语义分析自动生成源码，提供数据结构的初始化、序列化以及反序列等接口。
* `compile "com.google.protobuf:protobuf-lite:3.0.0"`是**Protobuf支持库的精简版本**，在原有的基础上，用public替换set、get方法，**减少Protobuf生成代码的方法数目**
>这也解释了提高p高效性的一个方法就是使用lite版本，除此之外更加重要的是nano，官网对这个库不再维护了。

  
### 2.参考我的mk配置  
```
LOCAL_PATH := $(call my-dir)  
include $(CLEAR_VARS)  
  
#LOCAL_AAPT_FLAGS := --auto-add-overlay  
  
LOCAL_STATIC_JAVA_LIBRARIES := \  
zxing \  
wearableservicesdk \  
protobufnano \  
protobuflite \  
  
LOCAL_RESOURCE_DIR := \  
$(LOCAL_PATH)/res \  
frameworks/support/v7/recyclerview/res \  
frameworks/support/v7/appcompat/res  
  
#LOCAL_SRC_FILES := $(call all-java-files-under, src)  
LOCAL_SRC_FILES := \  
$(call all-java-files-under, src) \  
$(call all-proto-files-under, proto)  
LOCAL_PROTOC_OPTIMIZE_TYPE := lite  
  
#LOCAL_JACK_ENABLED:=disabled  
  
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res  
  
include packages/apps/Car/libs/car-stream-ui-lib/car-stream-ui-lib.mk  
include frameworks/base/packages/SettingsLib/common.mk  
  
LOCAL_PACKAGE_NAME := protol_sunstlovefang  
LOCAL_CERTIFICATE := platform  
LOCAL_PROGUARD_ENABLED := disabled  
LOCAL_PRIVILEGED_MODULE := true  
  
include $(BUILD_PACKAGE)  
include $(CLEAR_VARS)  
#include $(BUILD_STATIC_JAVA_LIBRARY)  
  
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES += protobufnano:libs/protobuf-javanano-3.1.0.jar  
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES += protobuflite:libs/protobuf-lite-3.0.1.jar  
  
include $(BUILD_MULTI_PREBUILT)  
#include $(BUILD_STATIC_JAVA_LIBRARY)  
include $(call all-makefiles-under,$(LOCAL_PATH))  
``` 
mk编译的解析，可以参考我的另一篇文章介绍
## 三：编写ProtoBuff  
  
配置完成后，我们要开始使用ProtoBuff了，使用方法也比较简单，大概有如下3个步骤
-   定义用于消息文件.proto
-   使用protobuf的编译器编译消息文件
-   使用编译好对应语言的类文件进行消息的序列化与反序列化
  
### 1.TestProto.proto编写参考 

在src子目录main中创建proto文件夹，放.proto文件  
```
syntax = "proto3";  //sunst2019-07-25 注意文中版权声明，谢谢
  
option java_package = "com.example.protos";  
option java_outer_classname = "TestProto";  
option java_multiple_files = false;  
  
  
message SimpleMessage {  
int32 lucky_number = 1;  
int32 unlucky_number = 2;  
}  
  
enum SimpleEnum {  
SimpleEnum_UNDEFINE = 0;  
SimpleEnum_FANGFANG = 1;  
SimpleEnum_SUNST = 2;  
GOAL_TYPE_MONTHLY = 3;  
}  
  
message SimpleEnumStruct {  
SimpleEnum simle_enum = 0x01;  
SimpleMessage simleMessage = 0x04;  
int32 simple_data = 0x05;  
}  
message SelfInfo {  
string deviceName = 1;  
int32 timeOut = 2;  
bool b1 = 3;  
string string2 = 4;  
repeated SimpleMessage repeat = 5;  
map<string, SimpleMessage> mapfild = 6;  
}  
  
message SetSimpleEnumStruct {  
repeated SimpleEnumStruct goal_list = 1;  
}

message Person { 
int32 id = 1;//24
string name = 2; //sunst
string email = 3; //qyddai@gmail.com
}
```  
**pps注意：**  
>1. 必须有syntax = "proto3";  
>2. 可以使用import包名\文件名.proto 引入其他proto中message  
>3. message中不能有同样名字和tag  
>4. enum 第一个必须是0  
>5. 多个enum中字段名不能重复  
>6. 对象默认值：  
>数字类默认0，bool 默认false，string = ""  
>复杂对象默认不为空  
>Map字段默认为空  
>enum默认为0  

### 【重要点get】这里拿Person这个简单的Message来验证以下，为什么前面说的对比XML，GSON更加高效

实际的Person二进制消息为:

>08 18 12 0a 77 75 6a 69 6e 67 63 68 61 6f 1a 16 77 75 6a 69 6e 67 63 68 61 6f 39 32 40 67 6d 61 69 6c 2e 63 6f 6d

这段二进制流数据是怎么组成的

#### （1）.Varints

一般情况下int类型都是固定4个字节，protobuf定义了一种变长的int，每个字节最高位表示后面还有没有字节，低7位就为实际的值，并且使用小端的表示方法。例如1,varint的表示方法就为:
>0000 0001

是不是这样就省了三个字节。

再例如300,4字节表示为:10 0101100,varint表示为:
>10101100 00000010

所以前面消息为Person的id的值为00011000，即0x18。

负数的最高位为1，如果负数也使用这种方式表示就会出现一个问题,int32总是需要5个字节，int64总是需要10个字节。

所以定义了另外一种类型：sint32，sint64。采用ZigZag编码，所有的负数都使用正数表示,计算方式:
-   sint32  
    (n << 1) ^ (n >> 31)
-   sint64  
    (n << 1) ^ (n >> 63)
![](https://github.com/qydq/anopen/blob/master/screen/protobuff_varints.png?raw=true)
使用Varint编码的类型有int32, int64, uint32, uint64, sint32, sint64, bool, enum。Java里面没有对应的无符号类型，int32与uint32一样。

#### （2）.Wire Type

每个消息项前面都会有对应的tag，才能解析对应的数据类型，表示tag的数据类型也是Varint。

tag的计算方式: (field_number << 3) | wire_type

每种数据类型都有对应的wire_type:
![](https://github.com/qydq/anopen/blob/master/screen/protobuff_writetype.png?raw=true)
所以wire_type最多只能支持8种，目前有6种。

所以前面Person的id,field_number为1,wire_type为0，所以对应的tag为

```
1 <<< 3 | 0  = 0x08
```
Person的name,field_number为2,wire_type为2,所以对应的tag为
```
2 <<< 3 | 2 = 0x12
```
对应Length-delimited的wire type,后面紧跟着的Varint类型表示数据的字节数。

所以name的tag后面紧跟的0x0a表示后面的数据长度为10个字节，即"wujingchao"的UTF-8 编码或者ASCII值:
>08 18 12 0a 77 75 6a 69 6e 67 63 68 61 6f 1a 16

嵌套的消息类型embedded messages与packed repeated fields也是使用这种方式表示，对应默认值的数据，是不会写进protobuf消息里面的。

packed repeated与repeated的区别在于编码方式不一样，repeated将多个属性类型与值分开存储。而packed repeated采用Length-delimited方式。下面这个是官方文档的例子:
```
message Test4 {  
	repeated int32 d = 4 [packed=true];  
}
22 // tag (field number 4, wire type 2)  
06 // payload size (6 bytes)  
03 // first element (varint 3)   
8E 02 // second element (varint 270)   
9E A7 05 // third element (varint 86942)
```

如果没有packed的属性是这样存储的:
```
20 //tag(field number 4,wire type 0)  
03 //first element (varint 3)  
20 //tag(field number 4,wire type 0)  
8E 02//second element (varint 270)  
20 //tag(field number 4,wire type 0)  
9E A7 05 // third element (varint 86942)
```

这种方式比较节省内存，所以proto3的repeated默认就是使用packed这种方式来存储。(proto2与proto3区别在于.proto的语法)。

有了以上的相关概念，我们在读protobuf的源码就比较容易了。

[参考](https://developers.google.com/protocol-buffers/docs/encoding)

### 2. 编译工程，

生成java中间文件在目录
>build\generated\source\proto\main\javanano  
 
**注意：  **
* 反序列化的对象，如果数据中不存在会使用默认值  
* 如果数据比对象字段多，默认会忽略  
  
### 3.ProtoBuf序列化与反序列化  
```
public void encode() {  
TestProto.SelfInfo selfInfo = new TestProto.SelfInfo();  
  
TestProto.SimpleMessage message = new TestProto.SimpleMessage();  
message.luckyNumber = 110;  
message.unluckyNumber = 119;  
  
selfInfo.mapfild = new HashMap<>();  
selfInfo.mapfild.put("tessMsg", message);  
  
selfInfo.b1 = true;  
selfInfo.deviceName = "sunst_ina_lafa";//inafal  
  
TestProto.SimpleMessage message2 = new TestProto.SimpleMessage();  
message2.luckyNumber = 1110;  
message2.unluckyNumber = 1119;  
  
selfInfo.repeat = new TestProto.SimpleMessage[]{message, message2};  
  
byte[] bytes = TestProto.SelfInfo.toByteArray(selfInfo);  
System.out.println(HexBin.encode(bytes));  
  
System.out.println(selfInfo.deviceName);  
}  
  
@Test  
public void decode() {  
byte[] data = HexBin.decode("057669636518012A04073734D73671204086E1077");  
try {  
TestProto.SelfInfo selfInfo = TestProto.SelfInfo.parseFrom(data);  
System.out.println(selfInfo);  
} catch (InvalidProtocolBufferNanoException e) {  
e.printStackTrace();  
}  
}
```
---
以上便是android protobuff使用的全部内容，protobuff使用的场景非常的广泛，如跨进程，蓝牙，就像我在介绍socket那章节，利用protobuff完成python和android之间的通信也是未尝不可

请尊重劳动成果，注意文中**版权声明**，Android专栏不定时更新，可以[点击关注我知乎](https://www.zhihu.com/people/qydq/activities)。也可以同时关注人工智能专栏，本内容作者`sunst`，有问题请沟通`qyddai@gmail.com`

> 作者：sunst 开始创建日期：2016-08-31 最后更新时间：2019-07-25 00:55