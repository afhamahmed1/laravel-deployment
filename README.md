### **1. Prepare the VPS**

1. **Access the VPS:**

   - Use SSH to log in to your VPS:
     ```bash
     ssh user@your-vps-ip
     ```

2. **Install Required Software:**

   - Update your system:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Install PHP, Composer, MySQL, and a web server (e.g., Nginx or Apache):
     ```bash
     sudo apt install php php-cli php-mbstring php-xml php-bcmath php-curl php-zip unzip mysql-server nginx composer -y
     ```

3. **Set Up MySQL:**
   - Secure your MySQL installation:
     ```bash
     sudo mysql_secure_installation
     ```
   - Create a database and user for Laravel:
     ```sql
     CREATE DATABASE laravel_db;
     CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'your_password';
     GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
     FLUSH PRIVILEGES;
     ```

---

### **2. Prepare the Laravel Application**

1. **Upload Your Laravel Application:**

   - Use `scp` or a file manager to upload your project to the VPS:
     ```bash
     scp -r /path/to/your/laravel/project user@your-vps-ip:/var/www/laravel
     ```

2. **Set Permissions:**

   - Navigate to the project directory:
     ```bash
     cd /var/www/laravel
     ```
   - Set proper permissions for `storage` and `bootstrap/cache`:
     ```bash
     sudo chmod -R 775 storage bootstrap/cache
     sudo chown -R www-data:www-data /var/www/laravel
     ```

3. **Install Dependencies:**

   - Run Composer to install dependencies:
     ```bash
     composer install --optimize-autoloader --no-dev
     ```

4. **Set Up Environment Variables:**

   - Create or edit the `.env` file:
     ```bash
     cp .env.example .env
     nano .env
     ```
   - Update database credentials and other settings.

5. **Generate the Application Key:**

   ```bash
   php artisan key:generate
   ```

6. **Run Migrations:**

   ```bash
   php artisan migrate --force
   ```

7. **Set File Cache:**
   ```bash
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   ```

---

### **3. Configure the Web Server**

1. **Nginx Configuration:**

   - Create a new site configuration:
     ```bash
     sudo nano /etc/nginx/sites-available/laravel
     ```
   - Add the following:

     ```nginx
     server {
         listen 80;
         server_name your-domain.com;
         root /var/www/laravel/public;

         index index.php index.html;

         location / {
             try_files $uri $uri/ /index.php?$query_string;
         }

         location ~ \.php$ {
             include snippets/fastcgi-php.conf;
             fastcgi_pass unix:/var/run/php/php8.1-fpm.sock; # Adjust PHP version as needed
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
         }

         location ~ /\.ht {
             deny all;
         }
     }
     ```

   - Enable the site:

     ```bash
     sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
     ```

   - Test and reload Nginx:
     ```bash
     sudo nginx -t
     sudo systemctl reload nginx
     ```

2. **Apache Configuration (if using Apache):**

   - Enable mod_rewrite:
     ```bash
     sudo a2enmod rewrite
     ```
   - Configure your virtual host:
     ```bash
     sudo nano /etc/apache2/sites-available/laravel.conf
     ```
   - Add:

     ```apache
     <VirtualHost *:80>
         ServerName your-domain.com
         DocumentRoot /var/www/laravel/public

         <Directory /var/www/laravel>
             AllowOverride All
             Require all granted
         </Directory>

         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
     ```

   - Enable the site:
     ```bash
     sudo a2ensite laravel
     sudo systemctl reload apache2
     ```

---

### **4. Configure DNS**

- Point your domain's DNS records to your VPS's IP address.

---

### **5. Secure the Server**

1. **Install SSL:**

   - Use Let's Encrypt to secure your application:
     ```bash
     sudo apt install certbot python3-certbot-nginx
     sudo certbot --nginx -d your-domain.com
     ```

2. **Enable Firewall:**
   ```bash
   sudo ufw allow 'Nginx Full'
   sudo ufw enable
   ```

---

### **6. Test and Monitor**

- Access your app in the browser using your domain or IP.
- Monitor logs for issues:
  ```bash
  tail -f /var/log/nginx/error.log
  tail -f /var/www/laravel/storage/logs/laravel.log
  ```
