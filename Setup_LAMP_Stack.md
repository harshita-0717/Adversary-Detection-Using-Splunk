## 🛠️ Phase 1: Install the LAMP Stack (Ubuntu VM)

Since your Ubuntu VM is currently on a **Host-Only Network** and needs packages downloaded, you must temporarily switch your network adapter back to **NAT** to access the internet, just as you did before.

### 1\. **Prepare the System (Ensure Internet Access)**

1.  **Stop Ubuntu VM.**
2.  In your Hypervisor settings (VirtualBox/VMware), change the network adapter for the **Ubuntu VM** from **Host-Only** to **NAT** or **Bridged** (to allow internet access).
3.  **Start Ubuntu VM.**
4.  Run a quick update:
    ```bash
    sudo apt update
    ```

### 2\. **Install Apache Web Server**

Apache is the "A" in LAMP and serves the web pages.

```bash
sudo apt install apache2 -y
```

  * **Verification:** After installation, check the status:
    ```bash
    sudo systemctl status apache2
    ```
    *It should show **`active (running)`**.*

### 3\. **Install MySQL (Database)**

MySQL is the "M" in LAMP and will host the vulnerable application's data.

```bash
sudo apt install mysql-server -y
```

  * **Secure Installation:** It's best to run the secure installation script. Follow the prompts (setting a strong root password for MySQL).
    ```bash
    sudo mysql_secure_installation
    ```

### 4\. **Install PHP and Modules**

PHP is the "P" in LAMP and runs the web application code, connecting it to the database.

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

  * **Explanation:** `php-mysql` is critical as it allows PHP to talk to the MySQL database.

### 5\. **Restart Apache**

For PHP to be correctly integrated, restart the web server:

```bash
sudo systemctl restart apache2
```

-----

## 🧪 Phase 2: Verify the LAMP Stack

### 1\. **Verify Apache**

Open the web browser on your **Ubuntu VM** and navigate to:

```
http://localhost/
```

  * **Expected Result:** You should see the default **"Apache2 Ubuntu Default Page."**

### 2\. **Verification for PHP**

Create a simple PHP file in the web root directory (`/var/www/html/`) to confirm PHP processing works.

1.  Create the file:
    ```bash
    sudo nano /var/www/html/info.php
    ```
2.  Add this content to the file:
    ```php
    <?php
    phpinfo();
    ?>
    ```
3.  Save and close the file.
4.  Open the web browser on your **Ubuntu VM** and navigate to:
    ```
    http://localhost/info.php
    ```
      * **Expected Result:** You should see a large page displaying your PHP configuration details. This confirms PHP is working with Apache.

### 3\. **Clean Up**

Delete the test file for security:

```bash
sudo rm /var/www/html/info.php
```

-----

## ⚠️ Phase 3: Restore Isolation and Next Steps

Before moving to the attacks, you must isolate the environment again.

1.  **Shut down the Ubuntu VM.**
2.  Change the network adapter back to **Host-Only Adapter**.
3.  **Start both Kali and Ubuntu VMs.**

