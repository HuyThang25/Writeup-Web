Đề cho một trang login

![image](https://github.com/user-attachments/assets/d692518c-886b-483d-94e1-6292a02ee4af)

Sau khi thử một số paload SQLi thì thấy web không dính và nhập sai thì không thấy thông báo hay thay đổi gì. Mình thử xem source thì thấy có comment code ở dưới

![image](https://github.com/user-attachments/assets/9cb549dc-3914-4d86-a612-af53c8af552f)

Đoạn code thực hiện kiểm tra hash sha256 của password nếu bằng `"0"` thì cho qua. Mình search `magic hash` của sha256 thì ra một số chuỗi
```
34250003024812:0e46289032038065916139621039085883773413820991920706299695051332
TyNOQHUS:0e66298694359207596086558843543959518835691168370379069085300385
CGq'v]`1:0e24075800390395003020016330244669256332225005475416462877606139
\}Fr@!-a:0e72388986848908063143227157175161069826054332235509517153370253
|+ydg uahashcat:0e47232208479423947711758529407170319802038822455916807443812134
```
Cơ chế là khi hash một chuỗi ra định dạng `0e[0-9]`, sẽ được hiểu là biểu thức toán học `0x10^x` = 0 nên vượt qua được điều kiện if. Mình thực hiện đăng nhập theo chuỗi trên thì được dẫn tới trang upload file.

![image](https://github.com/user-attachments/assets/58252a49-6587-4566-9ab1-f687b63dfc5a)


Thực hiện upload webshell lên để lấy flag thôi.
