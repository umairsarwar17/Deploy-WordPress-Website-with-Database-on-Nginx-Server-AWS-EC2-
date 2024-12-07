# Deploy WordPress Website with Database on Nginx Server (AWS EC2)

## Prerequisites
[[Deploy Hospital Management System Frontend Website on Apache2 Server (AWS EC2)]]
1. **AWS Account:** Ensure you have an active AWS account.
2. **WordPress Package:** Download WordPress or use the official repository.
3. **Database:** The website should support a database such as MySQL or MariaDB.
4. **AWS CLI (Optional):** Install the AWS Command Line Interface for convenience.

---

## Step 1: Launch an EC2 Instance

1. **Log in to the AWS Management Console:**
   Navigate to the **EC2 Dashboard**.

2. **Launch an Instance:**
   - Click on **Launch Instance**.
   - **Choose an AMI:** Select an Amazon Machine Image (AMI), such as **Amazon Linux 2** or **Ubuntu Server**.
   - **Instance Type:** Select **t2.micro** (free tier eligible).
   - **Configure Security Group:** Allow the following inbound rules:
     - **HTTP (Port 80):** Open to `0.0.0.0/0`.
     - **SSH (Port 22):** Restrict to your IP or open to `0.0.0.0/0` (not recommended for production).
     - **MySQL/Aurora (Port 3306):** Open to `0.0.0.0/0` for database access (adjust for security).
   - **Key Pair:** Create or select an existing key pair for SSH access.
   - **Launch the Instance.**

---

## Step 2: Connect to Your EC2 Instance

1. **SSH into the Instance:**
   Use the `.pem` key file to connect:
   ```bash
   ssh -i "your-key-file.pem" ec2-user@<INSTANCE_PUBLIC_IP>
   ```

2. **Update the Package Manager:**
   Run the following commands to update the system:
   ```bash
   sudo yum update -y   # For Amazon Linux
   sudo apt update -y   # For Ubuntu
   ```

---

## Step 3: Install Nginx, PHP, and MySQL

1. **Install Nginx:**
   ```bash
   sudo yum install nginx -y   # For Amazon Linux
   sudo apt install nginx -y   # For Ubuntu
   ```

2. **Start and Enable Nginx:**
   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

3. **Install PHP and Required Extensions:**
   ```bash
   sudo yum install php php-mysqlnd php-fpm -y   # For Amazon Linux
   sudo apt install php-fpm php-mysql -y         # For Ubuntu
   ```

4. **Install MariaDB or MySQL:**
   ```bash
   sudo yum install mariadb-server -y   # For Amazon Linux
   sudo apt install mysql-server -y     # For Ubuntu
   ```

5. **Start and Enable the Database Service:**
   ```bash
   sudo systemctl start mariadb   # For MariaDB
   sudo systemctl enable mariadb
   sudo systemctl start mysql     # For MySQL
   sudo systemctl enable mysql
   ```

6. **Secure the Database Installation:**
   Run the following command and follow the prompts:
   ```bash
   sudo mysql_secure_installation
   ```

---

## Step 4: Create a Database for WordPress

1. **Log into MySQL:**
   ```bash
   sudo mysql -u root -p
   ```

2. **Create Database and User:**
   ```sql
   CREATE DATABASE wordpress;
   CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

---

## Step 5: Download and Configure WordPress

1. **Download WordPress:**
   ```bash
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

2. **Extract WordPress:**
   ```bash
   tar -xvzf latest.tar.gz
   ```

3. **Move WordPress Files:**
   ```bash
   sudo mv wordpress/* /usr/share/nginx/html/
   ```

4. **Set Permissions:**
   ```bash
   sudo chown -R nginx:nginx /usr/share/nginx/html
   sudo chmod -R 755 /usr/share/nginx/html
   ```

5. **Configure WordPress:**
   ```bash
   cd /usr/share/nginx/html
   sudo cp wp-config-sample.php wp-config.php
   sudo nano wp-config.php
   ```

   Update the following lines with your database details:
   ```php
   define( 'DB_NAME', 'wordpress' );
   define( 'DB_USER', 'wp_user' );
   define( 'DB_PASSWORD', 'your_password' );
   define( 'DB_HOST', 'localhost' );
   ```

---

## Step 6: Configure Nginx for WordPress

1. **Edit Nginx Configuration File:**
   ```bash
   sudo nano /etc/nginx/conf.d/wordpress.conf
   ```

2. **Add the Following Configuration:**
   ```nginx
   server {
       listen 80;
       server_name <INSTANCE_PUBLIC_IP>;
       root /usr/share/nginx/html;

       index index.php index.html index.htm;

       location / {
           try_files $uri $uri/ /index.php;
       }

       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass 127.0.0.1:9000;
           fastcgi_index index.php;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

3. **Test and Restart Nginx:**
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

---

## Step 7: Verify Deployment

1. **Access the Website:**
   Open the EC2 instance's public IP in your browser:
   ```
   http://<INSTANCE_PUBLIC_IP>
   ```

2. **Complete WordPress Setup:**
   Follow the on-screen instructions to set up your WordPress site.

---

## Step 8: Secure and Clean Up

1. **Enhance Security:**
   - Restrict SSH access to specific IPs in the security group.
   - Regularly update and patch the system.

2. **Backup Data:**
   Regularly back up your website files and database.

3. **Remove Unused Files:**
   Delete unnecessary files from `/usr/share/nginx/html` to keep the server clean.
