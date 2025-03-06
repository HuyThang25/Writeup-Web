Đây là challeng blackbox được viết bằng PHP. Sau một hồi fuzzing thì mình tìm được bug `LFI`
![image](https://github.com/user-attachments/assets/ecf86fe8-3aab-4476-a483-25851aa1b22d)

- Theo như tên bài mình đoán ngay là LFI to RCE bằng phpinfo. Đọc được thông tin cấu hình qua file `phpinfo.php`
![image](https://github.com/user-attachments/assets/7db0edd8-8c9b-472e-a2fa-5fff03f4718c)

- [Bài viết này](https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo) có hướng dẫn khai thác. Điều kiện để khai thác thì ta phải có một web dính `lfi`, xem được hàm `phpinfo()`, cho phép upload file,  và cuối cùng là có quyền ghi vào thư mục `/tmp` (thường thì mặc định user nào cũng có quyền ghi vào thư mục này). 
Đọc trong file phpinfo ta tìm thấy một giá trị `file_uploads` được set là On
![image](https://github.com/user-attachments/assets/d36e5233-fedd-40ae-99e9-dfc8e870bb09)

- Ý tưởng là ta sẽ thực hiện upload file trên file `phpinfo.php` để lấy được tên file. Các file được upload mặc định được lưu tạm thời ở thư mục /tmp với trên được random 6 ký tự. Nhưng có một vấn đề là khi kết thúc request thì file sẽ tự động xóa nên ta sẽ không thể lfi để đọc file. 

- Trong php sử dụng output buffer để tăng hiệu quả tranfer dữ liệu và giá trị này luôn được set mặc định là 4069byte . Khi output từ code php của bạn nhỏ hơn 4069byte thì nó vẫn tranfer bình thường nhưng khi output lớn hơn 4069byte thì lúc này php không thể gửi hết phần output được do đó nó sẽ gửi từng phần output bằng phương pháp chunked transfer encoding. 

- Lợi dụng điểm này ta sẽ chèn vào request lượng lớn ký tự để việc xử lý lâu hơn và các đủ thời gian thực thi file upload trước khi bị xóa. Sử dụng [script](https://www.insomniasec.com/downloads/publications/phpinfolfi.py) được viết sẵn để exploit. Trước khi chạy ra phân tích qua script một chút.

- Sử dụng 5000 ký tự A ở chèn vào các Header và param khiến cho viêc xử lý chậm lại
![image](https://github.com/user-attachments/assets/a9e92307-c0ae-4957-99b2-c7fffc8f3d28)

- Sau đó upload lên script ghi webshell vào file `/tmp/g`
![image](https://github.com/user-attachments/assets/7c7b28f7-1cbd-415f-97a4-bf6d90def595)

- Tiếp theo là gửi request liên tục để lấy tên file được upload và lfi file đó để thực thi cho đến khi thành công. Sửa lại param dính `lfi` 

![image](https://github.com/user-attachments/assets/e0b84f4f-cc44-4c8b-bd74-ed80ce93e5ee)

- Chạy tool

![image](https://github.com/user-attachments/assets/5c1acc76-e1e7-457f-b055-824750165340)

- Ở đây mình nhận được thông báo lỗi không tìm được file upload lên

![image](https://github.com/user-attachments/assets/58e48774-4337-436e-9059-77bf71067366)

- Sửa code để in ra request upload file

![image](https://github.com/user-attachments/assets/fbf8e406-7e86-45ee-8e53-759fcc8258b8)

![image](https://github.com/user-attachments/assets/1c1da8d1-b80c-4a02-ae12-9b5ff63dfa6e)

- Copy request sang burp suite để xem request có vấn đề gì

![image](https://github.com/user-attachments/assets/b4377b43-d752-4447-a432-120637c75278)

- Nhận được thông báo là do header hoặc cookie quá dài. Mình xóa phần `cookie` và header `HTTP_PRAGMA` thì request bình thường trở lại

![image](https://github.com/user-attachments/assets/c4e0c975-48d0-4ad8-b92c-f6f16d7f68c4)

- Vào tool và xóa 2 phần đó đi
![image](https://github.com/user-attachments/assets/27b43e51-c819-4f81-84ef-bb3f6fdc65b2)

- Vẫn nhận được thông báo không tìm thấy file upload

![image](https://github.com/user-attachments/assets/e5ea09c1-46cd-4701-8476-271eddc84290)

- Kiểm tra lại code thì thấy phần tìm kiếm tên file tìm đến vị trí của chuỗi sau
![image](https://github.com/user-attachments/assets/f182a6d0-e152-430e-adf2-a26304a844cc)

- Do response về đã encode ký tự `=>` nên không thể tìm đến chuỗi và nhận được thông báo không tim thấy file
![image](https://github.com/user-attachments/assets/61d00262-fc00-4498-9323-cccffb748582)

- Sửa lại thành như sau
![image](https://github.com/user-attachments/assets/e942f538-357d-4410-acc5-e0f0e53a488a)

- Chạy lai và nhận được thông báo upload lên thành công
![image](https://github.com/user-attachments/assets/ccf1d1e3-a2e8-45cf-b1c7-1935890a854f)

- Nhưng chạy webshell thì không nhận đc gì
![image](https://github.com/user-attachments/assets/36c6273e-8f64-45f0-b43f-f7a4066ffcfe)

- Sau một hồi loay hoay thì tìm thấy trong phpinfo đã cấm các hàm sau =((
![image](https://github.com/user-attachments/assets/d9276f8e-b85b-435d-903e-089dc18cd189)

- Lúc này mình chuyển hướng sang viết script php để đọc các file trong thư mục 
```php
<?php

$directory = '/'; 

if (is_dir($directory)) {
    if ($handle = opendir($directory)) {
        echo "Files in directory '$directory':\n";


        while (($file = readdir($handle)) !== false) {
            if ($file !== "." && $file !== "..") {
                echo $file . "\n";
            }
        }
        closedir($handle);
    } else {
        echo "Cannot open directory: $directory";
    }
} else {
    echo "Directory does not exist: $directory";
}
?>
```
![image](https://github.com/user-attachments/assets/0464def3-3a6d-49d1-bfab-b651c8d099fd)

- Sửa lại tool in ra response khi run code thành công

![image](https://github.com/user-attachments/assets/51fe0a6c-43c2-4fbc-9805-fdad0a8a8248)

- Thành công tìm được file chưa flag
![image](https://github.com/user-attachments/assets/7141ad3b-1298-4e56-ad82-d46c7d60fde5)

![image](https://github.com/user-attachments/assets/d5daa94b-3b27-4b60-99d7-c44a7f8deead)






