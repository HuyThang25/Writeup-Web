Đây là challenge whitebox được viết bằng python. Ban đầu mình có xem qua các chức năng ở frontend và source code thì thấy một số chỗ dính ssti do sử dụng hàm `render_template_string`

![image](https://github.com/user-attachments/assets/b54b4c4a-b865-48eb-90c7-f88704f8341b)
![image](https://github.com/user-attachments/assets/21d44d2f-5471-414e-bf90-b87444522c7b)

Nhưng trong path `verify` yêu cầu login admin nên exploit path `/user/<link>` trước. Thử một số payload ssti rce và đọc file thì bị filter, mình cũng có tìm một số payload bypass nhưng đều bị filter hết. 
```
payloads = [
        "*",
        "script",
        "alert",
        "debug",
        "%",
        "include",
        "html",
        "if",
        "for",
        "config",
        "img",
        "src",
        ".py",
        "main",
        "herf",
        "pre",
        "class",
        "subclass",
        "base",
        "mro",
        "__",
        "[",
        "]",
        "def",
        "return",
        "self",
        "os",
        "popen",
        "init",
        "globals",
        "base",
        "class",
        "request",
        "attr",
        "args",
        "eval",
        "newInstance",
        "getEngineByName",
        "getClass",
        "join"
    ]
```
Blacklist chặn hết các ký tự cho phép truền tham số, không thể gọi đến các module khác. Nên mình chuyển sang hướng tìm cách login admin.
Phân tích lại source code mình tìm được chỗ khá lạ
![image](https://github.com/user-attachments/assets/c7549550-994e-473f-8083-3455c417d06c)

Chức năng `forgetpasswd` thực hiện hash sha256 của email và tạo một bản ghi trong bảng `Validlink`. Sau đó sử dụng giá trị hash này request tới `changepasswd/<Hash>` là có thể đổi được passwd. Điều này dẫn đến việc nếu biết được email của một user bất kỳ là có thể đổi được password user đó.

Và còn 1 chỗ khá lạ nữa là 
![image](https://github.com/user-attachments/assets/9a74d912-2a94-4513-a5d3-1e79b8fb9249)

Trong path `/user/<link>` thực hiện render `shareprofile.html + giá tị bio` và truyền vào cả class `User` và `email` trong khi xem template lại chỉ sử dụng username. Công thêm việc giá trị bio có thể control được thì ta có thể lợi dụng class `User` để lấy thông tin của các User trong database.

![image](https://github.com/user-attachments/assets/154ed4d2-e43b-4cd8-84b7-38d4654265de)

Sử dụng payload sau để lấy email của admin
```
{{User.query.filter_by(username="admin").first().email}}
```
![image](https://github.com/user-attachments/assets/9e9d1de6-a3ed-4680-8aef-87fa6f0023b2)
![image](https://github.com/user-attachments/assets/c448efc2-bc7e-43cb-8083-e7fa0435e9d3)
![image](https://github.com/user-attachments/assets/a9434c37-6399-4ae6-80d9-626cccf1b0bf)

Sau khi có email của admin thực hiện hash và đổi mật khẩu

![image](https://github.com/user-attachments/assets/407e7cfa-4104-4870-9ba2-24b616292ced)
![image](https://github.com/user-attachments/assets/196e0bf6-fd04-4dbf-90dc-7ba89d563eea)
![image](https://github.com/user-attachments/assets/b402a4c6-2aca-45bb-ba49-8a2b50fbe9b8)

Login 
![image](https://github.com/user-attachments/assets/8617d153-fd1e-4de6-8340-9a3342872ca7)

Phân tích tiếp trong chức năng `verify`, thực hiện lấy ra thông tin về artist trong metadata của các ảnh mà user upload lên và render.

![image](https://github.com/user-attachments/assets/58e84c0e-ed7e-481d-a755-56bbbf82afa7)

Sử dụng trang web [này](https://online-metadata.com/edit-metadata) để sửa thông tin artist thành payload ssti

![image](https://github.com/user-attachments/assets/06024a8b-9ae3-4388-9e27-4e78e5005047)

upload file ảnh

![image](https://github.com/user-attachments/assets/dcfed0b2-64b3-4d40-8b26-8596214548dd)
![image](https://github.com/user-attachments/assets/c4a870a0-5d17-43f2-97fd-266c91feec4a)

Thực hiện verify để lấy flag

![image](https://github.com/user-attachments/assets/cf5dadf5-a469-4391-9ed3-9b278c56489b)
![image](https://github.com/user-attachments/assets/d57a7493-a63f-4ccc-96f5-1f90f50a196f)
