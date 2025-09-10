# 程序可被调试

## 0x01 漏洞描述

当在 `AndroidManifest.xml `文件中设置 `android:debuggable="true"`时，应用程序可以以调试模式启动，并被任意调试器附加调试。

## 0x02 漏洞危害

客户端若设置了可被动态调试时，增加了apk被破解、分析的风险。

目前动态调试器的功能都很强大，如果debuggable属性为true，则可轻易被调试，通常用于重要代码逻辑分析、破解付费功能等。

## 0x03 修复意见

* 在应用发布时，不要将`android:debuggable`的值改为ture。