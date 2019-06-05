### 视频讲解地址：https://edu.csdn.net/course/play/23750/268139
5行代码实现微信小程序模版消息推送 （含推送后台和小程序源码）

> 我们在做小程序开发时，消息推送是不可避免的。今天就来教大家如何实现小程序消息推送的后台和前台开发。源码会在文章末尾贴出来。

其实我之前有写过一篇：[《springboot实现微信消息推送，java实现小程序推送，含小程序端实现代码》](https://www.jianshu.com/p/ba54ea36a11d) 但是有同学反应这篇文章里的代码太繁琐，接入也比较麻烦。今天就来给大家写个精简版的，基本上只需要几行代码，就能实现小程序模版消息推送功能。

老规矩先看效果图
![](https://upload-images.jianshu.io/upload_images/6273713-37175ee03eda5956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是我们最终推送给用户的模版消息。这是用户手机微信上显示的推送消息截图。

### 本节知识点
1，java开发推送后台
2，springboot实现推送功能
3，小程序获取用户openid
4，小程序获取fromid用来推送

# 先来看后台推送功能的实现
只有下面一个简单的PushController类，就可以实现小程序消息的推送
![](https://upload-images.jianshu.io/upload_images/6273713-7188217b207ffe13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再来看下PushController类，你没看错，实现小程序消息推送，就需要下面这几行代码就可以实现了。
![](https://upload-images.jianshu.io/upload_images/6273713-6df7c66ded9837cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由于本推送代码是用springboot来实现的，下面就来简单的讲下。我我们需要注意的几点内容。
1，需要在pom.xml引入一个三方类库（推送的三方类库）
![](https://upload-images.jianshu.io/upload_images/6273713-a76ff40cae5c75da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
pom.xml的完整代码如下
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.qcl</groupId>
    <artifactId>wxapppush</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>wxapppush</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--微信小程序模版推送-->
        <dependency>
            <groupId>com.github.binarywang</groupId>
            <artifactId>weixin-java-miniapp</artifactId>
            <version>3.4.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
其实到这里我们java后台的推送功能，就已经实现了。我们只需要运行springboot项目，就可以实现推送了。
下面贴出完整的PushController.java类。里面注释很详细了。
```
package com.qcl.wxapppush;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

import cn.binarywang.wx.miniapp.api.WxMaService;
import cn.binarywang.wx.miniapp.api.impl.WxMaServiceImpl;
import cn.binarywang.wx.miniapp.bean.WxMaTemplateData;
import cn.binarywang.wx.miniapp.bean.WxMaTemplateMessage;
import cn.binarywang.wx.miniapp.config.WxMaInMemoryConfig;
import me.chanjar.weixin.common.error.WxErrorException;

/**
 * Created by qcl on 2019-05-20
 * 微信：2501902696
 * desc: 微信小程序模版推送实现
 */
@RestController
public class PushController {

    @GetMapping("/push")
    public String push(@RequestParam String openid, @RequestParam String formid) {
        //1,配置小程序信息
        WxMaInMemoryConfig wxConfig = new WxMaInMemoryConfig();
        wxConfig.setAppid("wx7c54942dfc87f4d8");//小程序appid
        wxConfig.setSecret("5873a729c365b65ab42bb5fc82d2ed49");//小程序AppSecret

        WxMaService wxMaService = new WxMaServiceImpl();
        wxMaService.setWxMaConfig(wxConfig);

        //2,设置模版信息（keyword1：类型，keyword2：内容）
        List<WxMaTemplateData> templateDataList = new ArrayList<>(2);
        WxMaTemplateData data1 = new WxMaTemplateData("keyword1", "获取老师微信");
        WxMaTemplateData data2 = new WxMaTemplateData("keyword2", "2501902696");
        templateDataList.add(data1);
        templateDataList.add(data2);

        //3，设置推送消息
        WxMaTemplateMessage templateMessage = WxMaTemplateMessage.builder()
                .toUser(openid)//要推送的用户openid
                .formId(formid)//收集到的formid
                .templateId("eDZCu__qIz64Xx19dAoKg0Taf5AAoDmhUHprF6CAd4A")//推送的模版id（在小程序后台设置）
                .data(templateDataList)//模版信息
                .page("pages/index/index")//要跳转到小程序那个页面
                .build();
        //4，发起推送
        try {            wxMaService.getMsgService().sendTemplateMsg(templateMessage);
        } catch (WxErrorException e) {
            System.out.println("推送失败：" + e.getMessage());
            return e.getMessage();
        }
        return "推送成功";
    }
}
```
看代码我们可以知道，我们需要做一些配置，需要下面信息
1，小程序appid
2，小程序AppSecret（密匙）
3，小程序推送模版id
4，用户的openid
5，用户的formid（一个formid只能用一次）

# 下面就是小程序部分，来教大家如何获取上面所需的5个信息。
1,appid和AppSecret的获取（登录小程序管理后台）
![](https://upload-images.jianshu.io/upload_images/6273713-a50191433b1f16ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2，推送模版id
![](https://upload-images.jianshu.io/upload_images/6273713-ba09fad3c72f66cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3，用户openid的获取，可以看下面的这篇文章，也可以看源码，这里不做具体讲解
 [小程序开发如何获取用户openid](https://www.jianshu.com/p/928932e74958)
4，获取formid
![](https://upload-images.jianshu.io/upload_images/6273713-8106f1a4d92f92e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看官方文档，可以知道我们的formid有效期是7天，并且一个form_id只能使用一次，所以我们小程序端所需要做的就是尽可能的多拿些formid，然后传个后台，让后台存到数据库中，这样7天有效期内，想怎么用就怎么用了。

### 所以接下来要讲的就是小程序开发怎么尽可能多的拿到formid了
![](https://upload-images.jianshu.io/upload_images/6273713-5985daad27d6e464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看下官方提供的，只有在表单提交时把report-submit设为true时才能拿到formid，比如这样
```
  <form report-submit='true' >
   <button  form-type='submit'>获取formid</button>
  </form>
```
所以我们就要在这里下功夫了，既然只能在form组件获取，我们能不能把我们小程序里用到最多的地方用form来伪装呢。

### 下面简单写个获取formid和openid的完整示例，方便大家学习
效果图
![](https://upload-images.jianshu.io/upload_images/6273713-83a337717c603e9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们要做的就是点击获取formid按钮，可以获取到用户的formid和openid，正常我们开发时，是需要把openid和formid传给后台的，这里简单起见，我们直接用获取到的formid和openid实现推送功能

##### 下面来看小程序端的实现代码
1,index.wxml
![](https://upload-images.jianshu.io/upload_images/6273713-4d1359587717263a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2,index.js
![](https://upload-images.jianshu.io/upload_images/6273713-acc8ec29c3344c66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里我们小程序端的代码也实现了，接下来测试下推送。

```
formid:  6ee9ce80c1ed4a2f887fccddf87686eb
openid o3DoL0Uusu1URBJK0NJ4jD1LrRe0
```
![](https://upload-images.jianshu.io/upload_images/6273713-661c5a9179a0482f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到我们用了上面获取到的openid和formid做了一次推送，显示推送成功
![](https://upload-images.jianshu.io/upload_images/6273713-39da1fd684b3ede4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6273713-28fe4f5b52c1fcf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里我们小程序消息推送的后台和小程序端都讲完了。
## 这里有两点需要大家注意
1，推送的openid和formid必须对应。
2，一个formid只能用一次，多次使用会报一下错误。
```
{"errcode":41029,"errmsg":"form id used count reach limit hint: [ssun8a09984113]"}
```
> 编程小石头，码农一枚，非著名全栈开发人员。分享自己的一些经验，学习心得，希望后来人少走弯路，少填坑。

这里就不单独贴出源码下载链接了，大家感兴趣的话，可以私信我，或者在底部留言,我会把源码下载链接贴在留言区。
单独找我要源码也行（微信2501902696）



源码链接：[https://github.com/qiushi123/wxapppush](https://github.com/qiushi123/wxapppush)
视频讲解地址：https://edu.csdn.net/course/play/23750/268139




