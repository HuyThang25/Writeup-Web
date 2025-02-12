Đọc source ở `/api/guestbook` thì mình thấy có vẻ là bug stored xss, và thử payload thì thấy dính bug. Nhưng bài lại không cho bot và web cung không có phân quyền người dùng nên cũng không khai thác được.

![image](https://github.com/user-attachments/assets/dc3f4407-30fc-452b-ad75-d362e5451c9e)

![image](https://github.com/user-attachments/assets/1a381eb0-e698-4e02-951f-62af79f00695)

Sau một hồi tìm kiếm thì thấy được trong file `Dockerfile` build app dưới mode dev khá lạ. Thường thì sẽ build mode production.

![image](https://github.com/user-attachments/assets/d6af12e6-a005-40b9-bcd8-d0176fc9e536)

Trong file `package.json` build bằng câu lệnh `next dev --turbopack`

![image](https://github.com/user-attachments/assets/50511cf1-4182-4d26-accc-3379c63857e3)

Giải thích qua một chút thì Turbopack là một bundler thế hệ mới được phát triển bởi Vercel, một công cụ build nhanh hơn Webpack, giúp tăng tốc độ khởi động và hot reload.
Mình đã thử tìm các bug liên quan đến version nextjs và turbopack nhưng không thấy bug nào khả thi. Sau khi end giải, atuhor có đưa ra solution:

```js
const REMOTE = "http://localhost:3000"

await (await fetch(`${REMOTE}/api/guestbook`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    "UR MESSAGE": `//#sourceMappingURL=${encodeURIComponent("/flag.txt")}`
  })
}))

let res2 = await fetch(`${REMOTE}/__nextjs_source-map?filename=file:///app/guestbook.txt`)
console.log(res2.status + ": " + await res2.text())
```

Đọc solution thì mình có thắc mắc làm sao có thể biết được path `__nextjs_source-map` này mà sử dụng? Trong khi trước đó mình có search trên các công cụ tìm kiếm nhưng không thấy nhắc đến các api này. Và mình đã tìm đến source của [nextjs](https://github.com/vercel/next.js/blob/canary/packages/next/src/client/components/react-dev-overlay/server/middleware-turbopack.ts) để đọc. Kết quả là tìm thấy có 1 `middleware turbopack` có xử lý path này 
![image](https://github.com/user-attachments/assets/cb1f61f3-40f9-4151-aae9-8d6074222703)

Path này truyền vào tham số `filename` nếu có prefix là `file:` thì sẽ gọi tới hàm `getSourceMapFromFile` trong `packages\next\src\client\components\react-dev-overlay\internal\helpers\get-source-map-from-file.ts` để xử lý 
![image](https://github.com/user-attachments/assets/f10bfac9-525a-45a5-b9c2-9639d20bbf31)

Hàm `getSourceMapFromFile` sẽ thực hiện đọc nội dung file và gọi tới hàm `getSourceMapUrl` trong `packages\next\src\client\components\react-dev-overlay\internal\helpers\get-source-map-url.ts`

![image](https://github.com/user-attachments/assets/5a4d18f6-d895-4ed3-86cd-e0b448e44869)

Hàm này tìm tới vị trí regex `/\/\/[#@] ?sourceMappingURL=([^\s'"]+)\s*$/gm` để lấy ra uri sau nó
![image](https://github.com/user-attachments/assets/83c5f958-9291-4e28-92a2-99e18eead263)

Và thực hiện lấy tài nguyên từ uri

![image](https://github.com/user-attachments/assets/f96f397f-7812-4d9a-9ffb-2be63c830853)

Nếu truyền vào uri với scheme `file://` thì có thể đọc được file local, từ đó lấy được flag. Sâu chuỗi lại thì ta có thể control được nội dung của file `guestbook.txt` thông qua path `/api/guestbook`. Và lợi dụng file này để đọc flag thông qua path `__nextjs_source-map`.

Ngoài ra thì còn một middleware xử lý 2 path là `/__nextjs_original-stack-frames` và `/__nextjs_launch-editor`. Thực hiện tạo stack frames và mở file bằng trình editor.
![image](https://github.com/user-attachments/assets/ee633f1a-72a7-4471-83f7-d66e269b7d41)

![image](https://github.com/user-attachments/assets/8cc19d36-a087-4f6a-92b3-085833e998ee)









