# Creating-Registration-Form-using-RDS
## About Project:

This project involves deploying a **user registration form** on an **Amazon EC2 instance**, with the backend connected to a **MySQL database hosted on Amazon RDS (Relational Database Service)**. The form allows users to submit their personal information such as **name, email, website, comment, and gender**, which is securely stored in a cloud-managed database. The application is built using **open-source technologies** and is hosted on a **Linux-based Nginx web server** with **PHP** for backend logic.

By separating the database layer onto **Amazon RDS**, the system benefits from **better scalability, easier maintenance, automated backups, high availability, and security features**, compared to managing a database directly on EC2.

## Technologies Used:

- **HTML**: For creating the frontend user registration form.
- **PHP**: Handles form submissions and communicates with the RDS-hosted MySQL database.
- **Amazon EC2**: Hosts the web server (Nginx) and the application code.
- **Amazon RDS (MySQL Engine)**: Provides a managed and scalable relational database service.
- **MySQL**: Used as the actual database engine on RDS to store user records.
- **Nginx**: Serves the PHP application and handles HTTP requests.
- **Amazon Linux**: Lightweight, secure OS running on EC2.
- **SSH**: Securely connects to the EC2 instance for deployment and configuration.

## Step 1: Launch an EC2 Instance

1. Go to AWS Console → EC2
2. Launch Instance.
    - Name → rds_register_form
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem_server_key
    - security group → launch-wizard-1

## Step 2: Connect and Install Packages

1. Connect to EC2 via SSH

```bash
ssh -i "pem-server-key.pem" ec2-user@ec2-44-203-87-121.compute-1.amazonaws.com
```

2. Update system

```bash
sudo yum update
```

3. Install nginx then start & enable

```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

4. Install mariadb105-server then start & enable

```bash
sudo yum install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

5. Install php-fpm then start & enable

```bash
sudo yum install php-fpm -y
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

6. Create file → signup.html

```bash
sudo vim /usr/share/nginx/html/signup.html
```

code of signup.html file:

```html
 <!DOCTYPE html>
 <html>
 <head>
 <title>Signup Form</title>
 </head>
 <body>
 <h2>Signup Form</h2>
 <form action="register.php" method="post">
 <label for="name">Name:</label><br>
 
 <input type="text" id="name" name="name" required><br><br>
 <label for="email">Email:</label><br>
 
 <input type="email" id="email" name="email" required><br><br>
 <label for="website">Website:</label><br>
 
 <input type="url" id="website" name="website"><br><br>
 <label for="comment">Comment:</label><br>
 
 <textarea id="comment" name="comment" rows="4" cols="50"></textarea><br><br>
 <label>Gender:</label><br>
 <input type="radio" id="female" name="gender" value="female" required>
 <label for="female">Female</label><br>
 <input type="radio" id="male" name="gender" value="male">
 <label for="male">Male</label><br>
 <input type="radio" id="other" name="gender" value="other">
 <label for="other">Other</label><br><br>
 <input type="submit" value="Submit">
 </form>
 </body>
 </html>
```

7. Launch an RDS MySQL Instance
- Go to **AWS Console** → **RDS**
- Click **Create Database**
    - Choose a database creation method **→ Standard create**
    - Engine options **→ MariaDB**
    - Engine version **→ MariaDB 11.4.5**
    - Templates **→ Free tier**
    - DB instance identifier **→** **myntra-db**
    - Master username **→** **admin**
    - Master password **→** **Auto generate password**
    - VPC → **Default**
    - Public access→ **Yes**
    - security group: **launch-wizard-1**
    - Click **Create database**
8. Now Connect your RDS database into your server

```bash
sudo mysql -u database-1.c8taowgcq72q.us-east-1.rds.amazonaws.com -u admin -p
```

9. Create the MySQL Database 

```bash
sudo mysql
CREATE DATABASE myntra;
USE myntra;
```

10. Inside the MySQL create Table:

```sql
CREATE TABLE users (
 id INT PRIMARY KEY AUTO_INCREMENT,
 name VARCHAR(20),
 email VARCHAR(100),
 website VARCHAR(255),
 gender VARCHAR(6),
 comment VARCHAR(100)
 );

EXIT;
```

11. Create php file → register.php

```bash
sudo vim /usr/share/nginx/html/register.php
```

Code of register.php file:

```php
 <?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

$name    = $_POST['name'];
$email   = $_POST['email'];
$website = $_POST['website'];
$comment = $_POST['comment'];
$gender  = $_POST['gender'];

// Database connection
$servername = "database-1.c8taowgcq72q.us-east-1.rds.amazonaws.com";
$username   = "admin";
$password   = "c410JrUFbr1DreBJ8wld";
$dbname     = "myntra";

// Create connection
$conn = mysqli_connect($servername, $username, $password, $dbname);

// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}

// Insert query
$sql = "INSERT INTO users (name, email, website, comment, gender)
        VALUES ('$name', '$email', '$website', '$comment', '$gender')";
?>
<!DOCTYPE html>
<html>
<head>
    <title>Form Submission Result</title>
</head>
<body>
<?php
if (mysqli_query($conn, $sql)) {
    echo "<h2>✅ New record created successfully!</h2>";
    echo "<h3>Submitted Information:</h3>";
    echo "<ul>";
    echo "<li><strong>Name:</strong> " . htmlspecialchars($name) . "</li>";
    echo "<li><strong>Email:</strong> " . htmlspecialchars($email) . "</li>";
    echo "<li><strong>Website:</strong> " . htmlspecialchars($website) . "</li>";
    echo "<li><strong>Comment:</strong> " . htmlspecialchars($comment) . "</li>";
    echo "<li><strong>Gender:</strong> " . htmlspecialchars($gender) . "</li>";
    echo "</ul>";
} else {
    echo "<h3>❌ Error: " . $sql . "<br>" . mysqli_error($conn) . "</h3>";
}

mysqli_close($conn);
?>
</body>
</html>
```

12. install connector for MySQL 

```bash
sudo yum install php8.4-mysqlnd.x86_64 -y
```

13. restart your nginx & php-fpm 

```bash
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```

14. now checking your database connected 

```bash
<enter-your-public-ip>/signup.html
```

15. Now Go to RDS Database check your data enter or not 

```bash
select*from users;
```

16. Terminate Your instance 
- go to AWS console → click on instance
- select your instance
- click on Instance state
- click on terminate (delete) instance
