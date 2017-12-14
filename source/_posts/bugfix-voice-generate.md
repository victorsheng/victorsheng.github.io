title: bugfix对接baidu语音合成
tags:
  - bugfix
categories: []
date: 2017-12-13 19:26:00
---
# 背景
项目中使用了baidu的语音合成,原本的设计是,批量获取设置到redis中,用户每次使用时从redis中获取

# 空指针异常
```
2017-12-13 16:34:30.076 ERROR [http-bio-6000-exec-45][DownloadVoiceFromBaiduYY.java:36] - DownloadVoiceFromBaiduYY error {}
java.lang.NullPointerException
    at com.baidu.aip.speech.AipSpeech.synthesis(AipSpeech.java:133) ~[java-sdk-3.2.0.jar:?]
    at com.ejlerp.web.wms.voice.baidu.DownloadVoiceFromBaiduYY.downloadVoice(DownloadVoiceFromBaiduYY.java:33) [classes/:?]
    at com.ejlerp.web.wms.voice.baidu.DownloadVoiceFromBaiduYY.downloadVoiceB64(DownloadVoiceFromBaiduYY.java:45) [classes/:?]
    at com.ejlerp.web.wms.voice.VoiceController.getB64(VoiceController.java:233) [classes/:?]
    at com.ejlerp.web.wms.voice.VoiceController.getVoiceData(VoiceController.java:149) [classes/:?]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:1.8.0_92]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[?:1.8.0_92]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_92]
    at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_92]
```
- 下载源码后debug返现返回的response Content-Type是大写,与sdk中get的string不一致,
于是查找github的源文件
<img src="http://pic.victor123.cn/17-12-13/86911891.jpg">

- 发现了,这个bug已经fix了,https://github.com/Baidu-AIP/java-sdk/commit/c338bc71690f84351def0d64b311986c586763d2

<img src="http://pic.victor123.cn/17-12-13/31618622.jpg">

# 其他遇到的问题
- 判断字符串写反
```
-            if (StringUtils.isEmpty(value)) {
+            if (!StringUtils.isEmpty(value)) {
                 redisMap.put(key, value);
             }

```
- APPkey修改错了配置文件

# 反思
- 在与外部api进行接口对接时,一定要判断各种异常情况的发生,不能只考虑正常情况
```
        try {
            TtsResponse res = client.synthesis(voice, "zh", 1, options);
            JSONObject result = res.getResult();
            if (result != null) {
                LOGGER.warn("downloadVoice:{}", result);
            }
            download = res.getData();

```
- 删除掉无用的代码,放置误导自己和别人
- 本地充分测试后,再提交测试部署,否则害人害己,耽误时间
- 应该充分利用各种调试工具或手段,如Btrace,(临时抱佛脚,来不及,自己有待提高)
- 不能依靠测试发现程序问题,即使是别人的代码,也要认真研读,滤清思路,不能头疼医头,脚痛医脚
- 要有自己的脚手架工程,方便自己后门程序的快速编写,和部署

# 待提高
- 应该充分利用各种调试工具或手段,如Btrace,(临时抱佛脚,来不及,自己有待提高)
- 测试用例编写
- 要有自己的脚手架工程,方便自己后门程序的快速编写,和部署
