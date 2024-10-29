![image](https://github.com/user-attachments/assets/d9dcebbe-a5f6-4dc2-8d9a-e270919008d3)
Đọc source code thì thấy mục tiêu của bài là phải bypass vào admin và vào được `/render` để attack ssti
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
Lúc đầu mình định làm theo hướng bypass jwt nhưng sau một hồi tìm kiếm thì không thấy vuln gì. Chuyển sang tìm chỗ khác thì thấy `ujson.loads` có (CVE-2022-31116)[https://security.snyk.io/vuln/SNYK-PYTHON-UJSON-2942122] . Về cơ bản của CVE này là khi chè nhưng giá trị 

![image](https://github.com/user-attachments/assets/86e6ed7a-d9f0-4227-bb08-4c53d9f8cb84)


![image](https://github.com/user-attachments/assets/b0a23c89-4523-4b73-82c6-be71f52cd361)



