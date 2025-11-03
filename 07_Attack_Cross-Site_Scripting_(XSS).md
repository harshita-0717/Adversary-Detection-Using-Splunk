# 🗺️Cross-Site Scripting (XSS) - Reflected or Stored (T0866)



### ✅ Confirmation: MITRE Mapping

| Attack Name | MITRE ID | Description |
| :--- | :--- | :--- |
| **Cross-Site Scripting** | **T0866** | A client-side injection attack where an adversary injects malicious scripts into content that is then delivered to other users. |



## 🎯 Objective: Cross-Site Scripting (XSS)

The objective of performing the XSS attack is to demonstrate and detect **Client-Side Code Injection**.

| Key Question | Answer |
| :--- | :--- |
| **What is the Objective?** | To execute a malicious script (usually **JavaScript**) inside the web browser of an unsuspecting website user by injecting it into a vulnerable part of the target website (your Ubuntu VM's guestbook page). |
| **Why Are We Doing This?** | XSS is a common vulnerability that allows for **Session Hijacking** (stealing cookies to take over a user's account) and **Phishing** (modifying the page to trick the user). Demonstrating and detecting this shows expertise in Application Security. |
| **What is Happening?** | The attacker sends a malicious script to the server, and the server, failing to recognize it as code, stores it (Stored XSS) or immediately reflects it (Reflected XSS) back to the victim's browser, which then **runs the script** because it believes the script came from the trusted website. |

---

## 🕵️ Breakdown of XSS Concepts

### 1. Attacker's Goal: Session Hijacking (MITRE TA0006)

The primary goal of XSS is often to steal a user's **Session Cookie**.

* **How it Works:** When you log into a site, the server gives your browser a cookie (your "session token") to remember you. The XSS payload uses JavaScript to read that cookie and send it to a server controlled by the attacker (e.g., `document.cookie` is sent to Kali).
* **The Impact:** Once the attacker has the victim's session cookie, they can load the website, manually input that cookie, and instantly **take over the victim's session** without needing a password.

### 2. The Payload: `<script>alert('1')</script>`

* **Why we use it:** The payload `<script>alert('1')</script>` is used during testing because it's harmless. If you see the pop-up box, you know the browser executed the script, proving the vulnerability exists.
* **The Real Payload:** The actual malicious payload would replace the `alert('1')` with code that sends the victim's cookie to the attacker.

### 3. The Two Types in Your Project

| XSS Type | How it Works | Your Project Scenario |
| :--- | :--- | :--- |
| **Stored XSS** | The malicious script is permanently **saved in the application's database** (e.g., in a comment field or profile bio). It runs every time any user loads that page. | **This is what you set up** with the `guestbook.php` page. The script is stored in the MySQL table. |
| **Reflected XSS** | The malicious script is sent via the **URL** (e.g., in a search parameter) and is immediately displayed ("reflected") back to the user without being saved anywhere. | This can be simulated by injecting a script into a vulnerable search bar on your website. |

---

## 🛡️ Detection Focus in Splunk (Blue Team Perspective)

For XSS, the Blue Team needs to analyze the web server logs, as that is where the attacker's script payload first lands.

* **Log Source:** **Apache Access Logs** (`access.log`).
* **Why Access Logs?** When the attacker submits the comment, the web server logs the full HTTP request, including the malicious payload, in the raw log file.
* **Detection Strategy:** We don't look for the script *running* (that happens in the victim's browser, which the server can't see). We look for the script **being submitted**.
* **Specific Signatures to Look For:**
    * **Common Tags:** `<script`, `onerror=`, `onload=`
    * **URL Encoding:** The characters `<` and `>` often appear as **`%3c`** and **`%3e`** in the URL or payload when submitted via a browser. Searching for these encoded patterns is very effective for detection.

This attack showcases your ability to secure the application layer and detect attacks using high-level log analysis.

### 🚀 Next Steps: Building the Environment for Stored XSS

Since you want to proceed with building the environment to demonstrate the full cycle (Attack, Defense, Detection), creating a **simple blog or guestbook page with a comment box** is the ideal setup for **Stored XSS**.

Here is the step-by-step guide to create a simple, vulnerable Stored XSS Guestbook page on **Ubuntu VM** using the existing LAMP stack.



## 💻 Phase 1: Deploying the Vulnerable Stored XSS Guestbook

This requires two files and the use of your existing `vulnerable_app` database.

### Step 1: Create the Database Table for Comments

We need a table to store the comments. Access your MySQL server (on your Ubuntu VM) to create the table:

```bash
sudo mysql -u root -p
```

*(Enter your MySQL root password)*

Then, execute the following SQL commands:

```sql
USE vulnerable_app;

CREATE TABLE IF NOT EXISTS comments (
    id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    comment TEXT NOT NULL,
    post_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

EXIT;
```

### Step 2: Create the Vulnerable Guestbook Page (`guestbook.php`)

This page will handle displaying existing comments and accepting new ones **without sanitization**.

On your **Ubuntu VM**, create the file:

```bash
sudo nano /var/www/html/guestbook.php
```

Paste the following PHP and HTML code (remember to replace **`YOUR_MYSQL_ROOT_PASSWORD`**):

```php
<?php
// Configuration (Replace with your details)
$servername = "localhost";
$db_user = "root";
$db_pass = "YOUR_MYSQL_ROOT_PASSWORD"; // *** CHANGE THIS ***
$dbname = "vulnerable_app";

$conn = new mysqli($servername, $db_user, $db_pass, $dbname);

// Handle new comment submission
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $username = $_POST['username'];
    $comment = $_POST['comment'];
    
    // WARNING: Storing unsanitized data directly into the database (the Stored XSS vector)
    $sql = "INSERT INTO comments (username, comment) VALUES ('$username', '$comment')";
    
    if ($conn->query($sql) === TRUE) {
        // Redirect to prevent form resubmission
        header("Location: guestbook.php"); 
        exit();
    }
}

// Fetch all comments for display
$comments = [];
$result = $conn->query("SELECT username, comment, post_time FROM comments ORDER BY post_time DESC");

if ($result && $result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        $comments[] = $row;
    }
}

$conn->close();
?>

<!DOCTYPE html>
<html>
<head>
    <title>Vulnerable Guestbook (Stored XSS)</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #fff8f8; padding: 20px; }
        .comment-box { border: 1px solid #ccc; padding: 10px; margin-bottom: 15px; background-color: white; border-radius: 5px; }
        .comment-text { margin-top: 5px; font-size: 1.1em; }
        textarea, input[type=text] { width: 100%; padding: 8px; margin: 5px 0 15px 0; border: 1px solid #ddd; box-sizing: border-box; }
        input[type=submit] { background-color: red; color: white; padding: 10px 15px; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Vulnerable Guestbook</h1>
    <p style="color: red;">*This page does not sanitize output, enabling Stored XSS.</p>

    <h2>Leave a Comment</h2>
    <form method="post" action="guestbook.php">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>

        <label for="comment">Comment:</label>
        <textarea id="comment" name="comment" rows="4" required></textarea>

        <input type="submit" value="Post Comment">
    </form>
    
    <hr>

    <h2>Recent Comments</h2>
    <?php foreach ($comments as $c): ?>
        <div class="comment-box">
            <strong><?php echo htmlspecialchars($c['username']); ?></strong> 
            <small>on <?php echo $c['post_time']; ?></small>
            <div class="comment-text"><?php echo $c['comment']; ?></div>
        </div>
    <?php endforeach; ?>
</body>
</html>
```

### Step 3: Execute the Stored XSS Attack (Kali VM)

1.  Open your browser on **Kali VM** and navigate to:
    `http://[Ubuntu-VM-IP-Address]/guestbook.php`
2.  **Post the Malicious Payload:**
      * **Username:** `XSS-Attacker`
      * **Comment:** `<script>alert('Stored XSS Success!')</script>`
3.  Click **Post Comment**.

### Expected Result

  * The page will reload.
  * The malicious comment should be displayed, and a **JavaScript pop-up box** with the message **"Stored XSS Success\!"** will immediately appear.
  * This confirms the script is **stored** in the database and **runs** every time the page is viewed by any user (including you).

-----

## 🛡️ Phase 2: Mitigation Strategy (Secure Stored XSS)

The fix for Stored XSS is to ensure the malicious script cannot be executed when it is **read from the database**. The defense involves using the `htmlspecialchars()` function when **displaying** the user-supplied data.

### Step 1: Create the Secure Guestbook Page (`secure_guestbook.php`)

On your **Ubuntu VM**, create the secure file:

```bash
sudo nano /var/www/html/secure_guestbook.php
```

Paste the following code. The crucial change is in the display loop:

```php
<?php
// Configuration
$servername = "localhost";
$db_user = "root";
$db_pass = "YOUR_MYSQL_ROOT_PASSWORD"; // *** CHANGE THIS ***
$dbname = "vulnerable_app";

$conn = new mysqli($servername, $db_user, $db_pass, $dbname);

// Handle new comment submission (SECURE SQL INSERT)
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $username = $_POST['username'];
    $comment = $_POST['comment'];
    
    // SECURE: Use prepared statements to prevent SQL Injection
    $stmt = $conn->prepare("INSERT INTO comments (username, comment) VALUES (?, ?)");
    $stmt->bind_param("ss", $username, $comment);
    
    if ($stmt->execute()) {
        $stmt->close();
        header("Location: secure_guestbook.php"); 
        exit();
    } else {
        // Display a general error instead of a database error
        echo "Error: Could not save comment.";
        $stmt->close();
    }
}

// Fetch all comments for display
$comments = [];
$result = $conn->query("SELECT username, comment, post_time FROM comments ORDER BY post_time DESC");

if ($result && $result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        $comments[] = $row;
    }
}

$conn->close();
?>

<!DOCTYPE html>
<html>
<head>
    <title>Secure Guestbook (Mitigated XSS)</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f0fff0; padding: 20px; }
        .comment-box { border: 1px solid #ccc; padding: 10px; margin-bottom: 15px; background-color: white; border-radius: 5px; }
        .comment-text { margin-top: 5px; font-size: 1.1em; }
        textarea, input[type=text] { width: 100%; padding: 8px; margin: 5px 0 15px 0; border: 1px solid #ddd; box-sizing: border-box; }
        input[type=submit] { background-color: green; color: white; padding: 10px 15px; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Secure Guestbook</h1>
    <p style="color: green;">*This page uses htmlspecialchars() to prevent Stored XSS.</p>

    <h2>Leave a Comment</h2>
    <form method="post" action="secure_guestbook.php">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>

        <label for="comment">Comment:</label>
        <textarea id="comment" name="comment" rows="4" required></textarea>

        <input type="submit" value="Post Comment">
    </form>
    
    <hr>

    <h2>Recent Comments</h2>
    <?php foreach ($comments as $c): ?>
        <div class="comment-box">
            <strong><?php echo htmlspecialchars($c['username']); ?></strong> 
            <small>on <?php echo $c['post_time']; ?></small>
            <div class="comment-text"><?php echo htmlspecialchars($c['comment']); ?></div>
        </div>
    <?php endforeach; ?>
</body>
</html>
```

### Step 2: Restart Apache

To ensure the PHP changes take effect:

```bash
sudo systemctl restart apache2
```

### Step 3: Verify the Defense (Kali VM)

1.  Open your browser on **Kali VM** and navigate to: `http://[Ubuntu-VM-IP-Address]/secure_guestbook.php`
2.  Post the malicious payload again (e.g., `<script>alert('XSS_FAIL')</script>`).
3.  **Expected Result:** The page should display the full script tag as **plain text** on the screen, and the JavaScript alert box should **not** appear. This confirms the defense is effective.

-----

## 🚨 Phase 3: Detection with Splunk (XSS)

Since the XSS payload is sent via a `POST` request (like the SQLi), it will be in the body of the Apache log, and we must search for the URL-encoded script components.

### Step 3: Search for XSS Attack Patterns

1.  In Splunk, set the time range to the last hour.

2.  Search for the XSS event using common signatures:

    ```spl
    index=main sourcetype="access_combined" "POST /guestbook.php"
    ```

      * **`"POST /guestbook.php"`**: Filters for submission to your vulnerable guestbook.


Here's an insightful observation\! 

Check the status code in logs and see the `302` log entry is the one we want to use for the **Stored XSS Attack Detection**, Why

| Event Time | Status Code | Bytes | Interpretation |
| :--- | :--- | :--- | :--- |
| **4:05:13 PM** | **`302`** | **`231`** | This request was likely a **successful submission** of the malicious comment to the vulnerable page. The server processed the `POST` request and then returned a **`302 Found`** status code, which is a common **redirect** instructing the browser to go back to the same page (`guestbook.php`) to display the new comment. |
| 4:02:02 PM | `200` | `547` | This was a successful `POST` request, but the server returned the final page content directly (Status `200 OK`) instead of redirecting. This might be a difference in how the **vulnerable vs. secure** scripts handle the submission, or simply a page view. |

Since the vulnerable script used a redirect after submission, the log entry with **`POST /guestbook.php 302`** is the one most reliably associated with the **injection of the payload** into the database.

-----

## 🚨 Phase 3: Creating a Splunk Alert for Stored XSS

We'll use a combination of the specific status code and the XSS payload keywords to create a highly accurate alert.

### Step 1: Refine the Search Query

We will target the vulnerable submission (the `302` redirect).

1.  In the Splunk Search & Reporting app, enter the following query:

    ```spl
    index=main sourcetype="access_combined" "POST /guestbook.php" 302
    ```

      * **`302`**: Narrows the search down to only redirect events (the successful injection).
      

### Step 2: Save the Search as an Alert

1.  Set the time range to **Last 5 minutes**.
2.  Click the **Save As** dropdown menu $\rightarrow$ Select **Alert**.

### Step 3: Configure and Save the Alert

Configure the following fields in the alert creation window:

| Field | Value | Notes |
| :--- | :--- | :--- |
| **Title** | `High Priority: Stored XSS Payload Detected (302)` | Clear title referencing the method. |
| **Alert Type** | **Scheduled** | |
| **Run Alert** | **Every 5 minutes** | |
| **Time Range** | **Last 5 minutes** | |
| **Trigger Condition** | **Number of Results** is greater than **0** | Trigger on any single detection. |
| **Trigger Actions** | **Add Actions** $\rightarrow$ **Log Event** | Logs the security incident. |
| **Log to Index** | `_internal` | Standard index for security operation logs. |
| **Event Text** | `Stored XSS payload detected (via 302 redirect) from $clientip$ in guestbook.php.` | Provides clear context. |
3\.  Click **Save**.

-----

## 🚀 Verification & Next Attack

The XSS attack cycle is now complete\!

1.  **Vulnerability:** Demonstrated the attack on `guestbook.php`.
2.  **Mitigation:** Secured the code in `secure_guestbook.php`.
3.  **Detection:** Created a Splunk alert to detect the payload based on the `POST` and `302` status code.


### Step 4: Create the Splunk XSS Alert

1.  Use the successful search query from Step 3.
2.  Click **Save As** $\rightarrow$ **Alert**.
3.  Configure the alert:
      * **Title:** `High Priority: Stored XSS Payload Detected`
      * **Alert Type:** `Scheduled` (e.g., Every 5 minutes)
      * **Trigger Condition:** `Number of Results` is greater than `0`.
      * **Trigger Action (Log Event):** Log to `_internal` with Event Text: `Stored XSS payload detected from $clientip$ in guestbook submission.`
4.  Click **Save**.

We have now completed the full security cycle for **Stored XSS**\!
