# SQL Injection (SQLi) - Unauthenticated Bypass (T1190)
Objective

## 💻 Deploying the Vulnerable Application for SQLi

We will create two files in the Apache web root directory (`/var/www/html/`):

1.  **`login.php`**: The vulnerable front-end login form.
2.  **`setup.php`**: A utility script to create the necessary MySQL database and table.

### 🛠️ Phase 1: Create the Setup Script and Database

We need a database and a table with a test user to make the login work.

**1. Create the Database Setup Script (`setup.php`)**

This script connects to your MySQL server and creates a database named `vulnerable_app`, a table named `users`, and inserts a test user.
First, you'll need to know your **MySQL root password** that you set during `sudo mysql_secure_installation`.

**1. Create the Database Setup Script (`setup.php`)**

```bash
sudo nano /var/www/html/setup.php
```

Paste the following content, making sure to replace **`YOUR_MYSQL_ROOT_PASSWORD`** with the actual root password you chose for MySQL:

```php
<?php
// Configuration (Replace with your details)
$servername = "localhost";
$username = "root";
$password = "YOUR_MYSQL_ROOT_PASSWORD"; // *** CHANGE THIS ***
$dbname = "vulnerable_app";

// Create connection
$conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// 1. Create Database
$sql = "CREATE DATABASE IF NOT EXISTS $dbname";
if ($conn->query($sql) === TRUE) {
    echo "Database '$dbname' created successfully.<br>";
} else {
    echo "Error creating database: " . $conn->error . "<br>";
}

// Select the database
$conn->select_db($dbname);

// 2. Create Users Table
$sql = "CREATE TABLE IF NOT EXISTS users (
    id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(30) NOT NULL,
    password VARCHAR(30) NOT NULL
)";

if ($conn->query($sql) === TRUE) {
    echo "Table 'users' created successfully.<br>";
} else {
    echo "Error creating table: " . $conn->error . "<br>";
}

// 3. Insert Test User (Password is 'password')
// In a real application, passwords would be securely hashed.
$sql = "INSERT INTO users (username, password) VALUES ('admin', 'password')";
// Use INSERT IGNORE to prevent errors if you run it multiple times
if ($conn->query("INSERT IGNORE INTO users (username, password) VALUES ('admin', 'password')") === TRUE) {
    echo "Test user 'admin'/'password' created successfully.<br>";
} else {
    echo "Error inserting test user (it may already exist): " . $conn->error . "<br>";
}

$conn->close();
?>
```

**2. Run the Setup Script**

Open a web browser on your **Ubuntu VM** and navigate to:

`http://localhost/setup.php`

You should see confirmation messages like:

  * `Database 'vulnerable_app' created successfully.`
  * `Table 'users' created successfully.`
  * `Test user 'admin'/'password' created successfully.`

**3. Clean Up**
Delete the setup file for security (as it contains the database credentials):

```bash
sudo rm /var/www/html/setup.php
```

-----

### 😈 Phase 2: Create the Vulnerable Login Page (`login.php`)

This is the key file for your SQLi demonstration. It is **deliberately insecure** because it directly inserts user input (`$_POST['username']` and `$_POST['password']`) into the SQL query **without any sanitization or using prepared statements**.

**1. Create the Login Script**

```bash
sudo nano /var/www/html/login.php
```

Paste the following PHP and HTML code, again replacing **`YOUR_MYSQL_ROOT_PASSWORD`**:

```php
<?php
$servername = "localhost";
$username = "root";
$password = "YOUR_MYSQL_ROOT_PASSWORD"; // *** CHANGE THIS ***
$dbname = "vulnerable_app";

$message = "";

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Connect to the database
    $conn = new mysqli($servername, $username, $password, $dbname);

    // Get user input (THIS IS THE VULNERABLE STEP)
    $user_input = $_POST['username'];
    $pass_input = $_POST['password'];

    // Insecure SQL Query: User input is directly concatenated into the query string
    $sql = "SELECT * FROM users WHERE username = '$user_input' AND password = '$pass_input'";

    $result = $conn->query($sql);

    if ($result === false) {
        // Display the SQL error for a classic Error-Based SQLi demo
        $message = "<div style='color: red;'><strong>Database Error!</strong> " . $conn->error . "</div>";
    } elseif ($result->num_rows > 0) {
        $message = "<div style='color: green;'><strong>SUCCESS!</strong> Login successful for user: " . htmlspecialchars($user_input) . "</div>";
    } else {
        $message = "<div style='color: red;'>Login failed: Invalid username or password.</div>";
    }

    $conn->close();
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Vulnerable Login Page</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f4f4; padding: 50px; }
        .login-box { background: white; padding: 20px; border-radius: 5px; box-shadow: 0 0 10px rgba(0,0,0,0.1); width: 300px; margin: auto; }
        input[type=text], input[type=password] { width: 90%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; border-radius: 4px; }
        input[type=submit] { background-color: #4CAF50; color: white; padding: 10px 15px; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <div class="login-box">
        <h2>Vulnerable Login</h2>
        <?php echo $message; ?>
        <form method="post" action="login.php">
            <label for="username">Username</label>
            <input type="text" id="username" name="username" required>

            <label for="password">Password</label>
            <input type="password" id="password" name="password" required>

            <input type="submit" value="Login">
        </form>
        <p style="font-size: 0.8em; color: #555;">
            *Use 'admin'/'password' to log in normally.
        </p>
        <p style="font-size: 0.8em; color: red;">
            *This page is intentionally vulnerable for educational purposes.
        </p>
    </div>
</body>
</html>
```

-----

### ✅ Phase 3: Verify and Attack Setup

**1. Restore Network Isolation**

As noted in your original plan, this is a **critical security step**. You must re-isolate your lab:

  * **Shut down** the Ubuntu VM.
  * In your Hypervisor (VirtualBox/VMware), change the network adapter back to **Host-Only Adapter**.
  * **Start** both your Kali and Ubuntu VMs.

**2. Verify Access (Kali to Ubuntu)**

From your **Kali VM**, open a browser and navigate to the IP address of your Ubuntu VM (e.g., `http://192.168.56.101/login.php`). This confirms your lab environment is ready.

**3. Test the SQL Injection Attack (Authentication Bypass)**

Now you can proceed with your **SQLi** exercise:

| Field | Injection Payload (Authentication Bypass) |
| :--- | :--- |
| **Username** | `admin' OR '1'='1` |
| **Password** | (Leave blank, or put anything) |

The final SQL query executed by the vulnerable PHP script will be:
`SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = ' '`

Since `'1'='1'` is always true, the database will return the first user (the **admin**) and grant you access, bypassing the password check.

You are now set up to practice the attack and, more importantly, to **capture and analyze the event in Splunk** and implement **defensive coding (using Prepared Statements)**, which is the ultimate goal of your project.

Would you like the instructions for **implementing the secure version** of the login page (the **mitigation strategy**) for comparison?
