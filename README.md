
# ============================================================
# CSE-478 LAB 5 — QUICK COMMAND REFERENCE
# Securing Apache Web Server with TLS
# ============================================================

# 1. Install required packages
sudo apt update
sudo apt install -y apache2 openssl

# 2. Create lab directory
mkdir -p ~/apache-https-lab
cd ~/apache-https-lab

# 3. Create CA database directories
mkdir -p demoCA/newcerts
touch demoCA/index.txt
echo 1000 > demoCA/serial

# 4. Copy OpenSSL configuration
cp /usr/lib/ssl/openssl.cnf ./openssl.cnf

# 5. Generate Root CA certificate
openssl req -new -x509 \
  -newkey rsa:2048 \
  -sha256 \
  -keyout ca.key \
  -out ca.crt \
  -days 3650 \
  -config openssl.cnf \
  -extensions v3_ca

# 6. Generate example.com private key
openssl genrsa -out example.com.key 2048
chmod 600 example.com.key

# 7. Generate example.com CSR
openssl req -new \
  -key example.com.key \
  -out example.com.csr \
  -config openssl.cnf

# Common Name: example.com

# 8. Sign example.com CSR
openssl ca \
  -in example.com.csr \
  -out example.com.crt \
  -cert ca.crt \
  -keyfile ca.key \
  -config openssl.cnf \
  -days 825 \
  -batch

# 9. Generate webserverlab.com private key
openssl genrsa -out webserverlab.com.key 2048
chmod 600 webserverlab.com.key

# 10. Generate webserverlab.com CSR
openssl req -new \
  -key webserverlab.com.key \
  -out webserverlab.com.csr \
  -config openssl.cnf

# Common Name: webserverlab.com

# 11. Sign webserverlab.com CSR
openssl ca \
  -in webserverlab.com.csr \
  -out webserverlab.com.crt \
  -cert ca.crt \
  -keyfile ca.key \
  -config openssl.cnf \
  -days 825 \
  -batch

# 12. Verify certificates
openssl x509 -in ca.crt -text -noout
openssl x509 -in example.com.crt -text -noout
openssl x509 -in webserverlab.com.crt -text -noout

# 13. Temporary OpenSSL server for example.com
cat example.com.key example.com.crt > example.com.pem
chmod 600 example.com.pem

openssl s_server \
  -accept 4433 \
  -cert example.com.crt \
  -key example.com.key \
  -www

# Browser:
# https://example.com:4433/

# Stop temporary server:
# Ctrl + C

# 14. Temporary OpenSSL server for webserverlab.com
cat webserverlab.com.key webserverlab.com.crt > webserverlab.com.pem
chmod 600 webserverlab.com.pem

openssl s_server \
  -accept 4433 \
  -cert webserverlab.com.crt \
  -key webserverlab.com.key \
  -www

# Browser:
# https://webserverlab.com:4433/

# Stop temporary server:
# Ctrl + C

# 15. Copy certificates for Apache
sudo mkdir -p /etc/ssl/apache-lab

sudo cp ca.crt /etc/ssl/apache-lab/
sudo cp example.com.crt /etc/ssl/apache-lab/
sudo cp example.com.key /etc/ssl/apache-lab/
sudo cp webserverlab.com.crt /etc/ssl/apache-lab/
sudo cp webserverlab.com.key /etc/ssl/apache-lab/

sudo chmod 644 /etc/ssl/apache-lab/*.crt
sudo chmod 600 /etc/ssl/apache-lab/*.key

# 16. Enable Apache modules
sudo a2enmod ssl
sudo a2enmod headers

# 17. Edit example.com Apache configuration
sudo nano /etc/apache2/sites-available/example.com.conf

# Add the HTTPS VirtualHost:
#
# <IfModule mod_ssl.c>
#     <VirtualHost *:443>
#         ServerAdmin admin@example.com
#         ServerName example.com
#         ServerAlias www.example.com
#         DocumentRoot /var/www/example.com/html
#         ErrorLog ${APACHE_LOG_DIR}/error.log
#         CustomLog ${APACHE_LOG_DIR}/access.log combined
#         SSLEngine on
#         SSLCertificateFile /etc/ssl/apache-lab/example.com.crt
#         SSLCertificateKeyFile /etc/ssl/apache-lab/example.com.key
#     </VirtualHost>
# </IfModule>

# 18. Edit webserverlab.com Apache configuration
sudo nano /etc/apache2/sites-available/webserverlab.com.conf

# Add the HTTPS VirtualHost:
#
# <IfModule mod_ssl.c>
#     <VirtualHost *:443>
#         ServerAdmin admin@webserverlab.com
#         ServerName webserverlab.com
#         ServerAlias www.webserverlab.com
#         DocumentRoot /var/www/webserverlab.com/html
#         ErrorLog ${APACHE_LOG_DIR}/error.log
#         CustomLog ${APACHE_LOG_DIR}/access.log combined
#         SSLEngine on
#         SSLCertificateFile /etc/ssl/apache-lab/webserverlab.com.crt
#         SSLCertificateKeyFile /etc/ssl/apache-lab/webserverlab.com.key
#     </VirtualHost>
# </IfModule>

# 19. Enable virtual hosts
sudo a2ensite example.com.conf
sudo a2ensite webserverlab.com.conf

# 20. Test Apache configuration
sudo apache2ctl configtest

# Expected:
# Syntax OK

# 21. Restart Apache
sudo systemctl restart apache2

# 22. Check Apache status
sudo systemctl status apache2 --no-pager

# 23. Verify HTTPS using curl
curl --cacert ca.crt -I https://example.com
curl --cacert ca.crt -I https://webserverlab.com

# 24. Verify TLS certificates
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -CAfile ca.crt

openssl s_client \
  -connect webserverlab.com:443 \
  -servername webserverlab.com \
  -CAfile ca.crt

# Expected:
# Verify return code: 0 (ok)

# 25. Browser verification
# https://example.com
# https://webserverlab.com
#
# Import ca.crt into Firefox/Chrome if a trust warning appears.
# ============================================================







#!/bin/bash

# ============================================================
# CSE-446 Web Engineering Lab
# Apache Web Server Installation and Virtual Host Configuration
# ============================================================

set -e

echo "============================================================"
echo "Starting Apache Web Engineering Lab Setup"
echo "============================================================"

# ------------------------------------------------------------
# 1. Update package list
# ------------------------------------------------------------

echo "[1/15] Updating package list..."
sudo apt update

# ------------------------------------------------------------
# 2. Install Apache2
# ------------------------------------------------------------

echo "[2/15] Installing Apache2..."
sudo apt install -y apache2

# ------------------------------------------------------------
# 3. Start and enable Apache service
# ------------------------------------------------------------

echo "[3/15] Starting Apache service..."
sudo systemctl start apache2
sudo systemctl enable apache2

# ------------------------------------------------------------
# 4. Configure firewall
# ------------------------------------------------------------

echo "[4/15] Configuring firewall..."

# Allow Apache HTTP traffic
sudo ufw allow 'Apache' || true

# ------------------------------------------------------------
# 5. Create website directories
# ------------------------------------------------------------

echo "[5/15] Creating website directories..."

sudo mkdir -p /var/www/example.com/html
sudo mkdir -p /var/www/webserverlab.com/html
sudo mkdir -p /var/www/anothervhost.com/html

# ------------------------------------------------------------
# 6. Set ownership and permissions
# ------------------------------------------------------------

echo "[6/15] Setting permissions..."

sudo chown -R "$USER:$USER" /var/www/example.com/html
sudo chown -R "$USER:$USER" /var/www/webserverlab.com/html
sudo chown -R "$USER:$USER" /var/www/anothervhost.com/html

sudo chmod -R 755 /var/www

# ------------------------------------------------------------
# 7. Configure local domain mappings
# ------------------------------------------------------------

echo "[7/15] Configuring /etc/hosts..."

sudo sed -i '/# WEB_ENGINEERING_LAB_START/,/# WEB_ENGINEERING_LAB_END/d' /etc/hosts

sudo tee -a /etc/hosts > /dev/null <<EOF

# WEB_ENGINEERING_LAB_START
127.0.0.1 example.com
127.0.0.1 www.example.com
127.0.0.1 webserverlab.com
127.0.0.1 www.webserverlab.com
127.0.0.1 anothervhost.com
127.0.0.1 www.anothervhost.com
# WEB_ENGINEERING_LAB_END
EOF

# ------------------------------------------------------------
# 8. Create Student Grade Calculator website
# ------------------------------------------------------------

echo "[8/15] Creating Student Grade Calculator..."

cat > /var/www/example.com/html/index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Student Grade Calculator</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: #eef2ff;
            padding: 30px;
        }

        .container {
            max-width: 500px;
            margin: auto;
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 5px 20px rgba(0, 0, 0, 0.12);
        }

        h1 {
            text-align: center;
            color: #1e3a8a;
        }

        label {
            display: block;
            margin-top: 15px;
            font-weight: bold;
        }

        input {
            width: 100%;
            padding: 10px;
            margin-top: 6px;
            border: 1px solid #aaa;
            border-radius: 6px;
        }

        button {
            width: 100%;
            margin-top: 20px;
            padding: 12px;
            border: none;
            border-radius: 6px;
            background: #2563eb;
            color: white;
            font-size: 16px;
            cursor: pointer;
        }

        button:hover {
            background: #1d4ed8;
        }

        #result {
            margin-top: 20px;
            padding: 15px;
            border-radius: 6px;
            background: #eff6ff;
            line-height: 1.8;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Student Grade Calculator</h1>

        <label for="name">Student Name</label>
        <input type="text" id="name" placeholder="Enter student name">

        <label for="marks">Marks</label>
        <input type="number" id="marks" placeholder="Enter marks" min="0" max="100">

        <button onclick="calculateGrade()">Calculate Grade</button>

        <div id="result"></div>
    </div>

    <script>
        function calculateGrade() {
            const name = document.getElementById("name").value.trim();
            const marks = Number(document.getElementById("marks").value);
            const result = document.getElementById("result");

            if (name === "") {
                result.innerHTML = "Please enter the student's name.";
                return;
            }

            if (isNaN(marks) || marks < 0 || marks > 100) {
                result.innerHTML = "Please enter valid marks between 0 and 100.";
                return;
            }

            let grade;
            let status;

            if (marks >= 80) {
                grade = "A+";
                status = "Excellent";
            } else if (marks >= 70) {
                grade = "A";
                status = "Very Good";
            } else if (marks >= 60) {
                grade = "A-";
                status = "Good";
            } else if (marks >= 50) {
                grade = "B";
                status = "Satisfactory";
            } else if (marks >= 40) {
                grade = "C";
                status = "Pass";
            } else {
                grade = "F";
                status = "Fail";
            }

            result.innerHTML = `
                <strong>Student Name:</strong> ${name}<br>
                <strong>Marks:</strong> ${marks}<br>
                <strong>Grade:</strong> ${grade}<br>
                <strong>Status:</strong> ${status}
            `;
        }
    </script>

</body>
</html>
EOF

# ------------------------------------------------------------
# 9. Create Electricity Bill Calculator website
# ------------------------------------------------------------

echo "[9/15] Creating Electricity Bill Calculator..."

cat > /var/www/webserverlab.com/html/index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Electricity Bill Calculator</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: #ecfdf5;
            padding: 30px;
        }

        .container {
            max-width: 500px;
            margin: auto;
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 5px 20px rgba(0, 0, 0, 0.12);
        }

        h1 {
            text-align: center;
            color: #047857;
        }

        label {
            display: block;
            margin-top: 15px;
            font-weight: bold;
        }

        input {
            width: 100%;
            padding: 10px;
            margin-top: 6px;
            border: 1px solid #aaa;
            border-radius: 6px;
        }

        button {
            width: 100%;
            margin-top: 20px;
            padding: 12px;
            border: none;
            border-radius: 6px;
            background: #059669;
            color: white;
            font-size: 16px;
            cursor: pointer;
        }

        button:hover {
            background: #047857;
        }

        #result {
            margin-top: 20px;
            padding: 15px;
            border-radius: 6px;
            background: #ecfdf5;
            line-height: 1.8;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Electricity Bill Calculator</h1>

        <label for="units">Electricity Units Consumed</label>
        <input
            type="number"
            id="units"
            placeholder="Enter consumed units"
            min="0"
        >

        <button onclick="calculateBill()">Calculate Bill</button>

        <div id="result"></div>
    </div>

    <script>
        function calculateBill() {
            const units = Number(document.getElementById("units").value);
            const result = document.getElementById("result");

            if (isNaN(units) || units < 0) {
                result.innerHTML = "Please enter a valid number of units.";
                return;
            }

            let bill = 0;

            if (units <= 100) {
                bill = units * 5;
            } else if (units <= 200) {
                bill = (100 * 5) + ((units - 100) * 7);
            } else if (units <= 300) {
                bill = (100 * 5) + (100 * 7) + ((units - 200) * 10);
            } else {
                bill = (100 * 5) + (100 * 7) + (100 * 10) + ((units - 300) * 15);
            }

            const serviceCharge = 50;
            const totalBill = bill + serviceCharge;

            result.innerHTML = `
                <strong>Units Consumed:</strong> ${units}<br>
                <strong>Energy Charge:</strong> ৳${bill.toFixed(2)}<br>
                <strong>Service Charge:</strong> ৳${serviceCharge.toFixed(2)}<br>
                <strong>Total Bill:</strong> ৳${totalBill.toFixed(2)}
            `;
        }
    </script>

</body>
</html>
EOF

# ------------------------------------------------------------
# 10. Create simple page for anothervhost.com
# ------------------------------------------------------------

echo "[10/15] Creating third virtual host page..."

cat > /var/www/anothervhost.com/html/index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Another Virtual Host</title>
</head>
<body>
    <h1>Welcome to anothervhost.com</h1>
    <p>This website is hosted by Apache Virtual Host.</p>
</body>
</html>
EOF

# ------------------------------------------------------------
# 11. Create Apache virtual host configuration files
# ------------------------------------------------------------

echo "[11/15] Creating Apache virtual host configurations..."

sudo tee /etc/apache2/sites-available/example.com.conf > /dev/null <<'EOF'
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/example.com/html

    <Directory /var/www/example.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
</VirtualHost>
EOF

sudo tee /etc/apache2/sites-available/webserverlab.com.conf > /dev/null <<'EOF'
<VirtualHost *:80>
    ServerName webserverlab.com
    ServerAlias www.webserverlab.com

    ServerAdmin webmaster@webserverlab.com
    DocumentRoot /var/www/webserverlab.com/html

    <Directory /var/www/webserverlab.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/webserverlab.com-error.log
    CustomLog ${APACHE_LOG_DIR}/webserverlab.com-access.log combined
</VirtualHost>
EOF

sudo tee /etc/apache2/sites-available/anothervhost.com.conf > /dev/null <<'EOF'
<VirtualHost *:80>
    ServerName anothervhost.com
    ServerAlias www.anothervhost.com

    ServerAdmin webmaster@anothervhost.com
    DocumentRoot /var/www/anothervhost.com/html

    <Directory /var/www/anothervhost.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/anothervhost.com-error.log
    CustomLog ${APACHE_LOG_DIR}/anothervhost.com-access.log combined
</VirtualHost>
EOF

# ------------------------------------------------------------
# 12. Enable websites
# ------------------------------------------------------------

echo "[12/15] Enabling virtual hosts..."

sudo a2ensite example.com.conf
sudo a2ensite webserverlab.com.conf
sudo a2ensite anothervhost.com.conf

# Disable the default Apache website
sudo a2dissite 000-default.conf || true

# ------------------------------------------------------------
# 13. Check Apache configuration
# ------------------------------------------------------------

echo "[13/15] Checking Apache configuration..."

sudo apache2ctl configtest

# ------------------------------------------------------------
# 14. Restart Apache
# ------------------------------------------------------------

echo "[14/15] Restarting Apache..."

sudo systemctl restart apache2

# ------------------------------------------------------------
# 15. Display results and test websites
# ------------------------------------------------------------

echo "[15/15] Displaying Apache virtual hosts..."

sudo apache2ctl -S

echo
echo "============================================================"
echo "Testing websites..."
echo "============================================================"

curl --noproxy '*' -I http://example.com
curl --noproxy '*' -I http://webserverlab.com
curl --noproxy '*' -I http://anothervhost.com

echo
echo "============================================================"
echo "Setup completed successfully!"
echo "============================================================"

echo "Open these URLs in your browser:"
echo "http://example.com"
echo "http://webserverlab.com"
echo "http://anothervhost.com"

echo
echo "Apache service status:"
sudo systemctl --no-pager status apache2
