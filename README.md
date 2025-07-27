🚦 FULL REQUEST FLOW: From Browser to Database and Back

🔁 Step 1: User Makes a Request

User enters in browser:

http://erms.anantgadaili.com.np:8081

🧭 This hits port 8081 on your host machine, mapped to the nginx container.
🔁 Step 2: Nginx Container Receives the Request

Nginx checks its config:

    location ~ \.php$ {
        fastcgi_pass php:9000;
        ...
    }

  .php file detected (index.php)

   Nginx knows PHP is handled by the php container (thanks to fastcgi_pass php:9000)

   Nginx forwards the request to php:9000 over Docker's internal DNS

🔁 Step 3: PHP-FPM Container Handles the PHP Script

PHP-FPM in the php container:

  Receives the .php file path and parameters from Nginx

  Finds the file at /var/www/html/admin/index.php

  Starts processing it with the PHP engine

🔁 Step 4: PHP Needs Data → Connects to MySQL

The PHP script might contain something like:

    <?php
    $con=mysqli_connect("db", "erms_user", "erms_pass", "erms");
    if(mysqli_connect_errno()){
    echo "Connection Fail".mysqli_connect_error();
    }


  PHP tries to connect to a host named mysql, which is the MySQL container service name defined in docker-compose.yml

  Uses port 3306 (default MySQL port)

  Authenticates using credentials set in the docker-compose.yml

🔁 Step 5: MySQL Container Handles the Query

    MySQL receives query from PHP

    Processes it

    Returns data (e.g., a list of employees)

🔁 Step 6: PHP-FPM Renders the Page

    PHP builds an HTML page dynamically using the MySQL data

    Sends it back to Nginx through FastCGI protocol

🔁 Step 7: Nginx Sends Response to Browser

    Nginx receives the generated HTML output

    Sends it back to the browser via HTTP (port 8081)

🔚 Final Result

User sees a fully rendered web page (e.g., employee dashboard) in their browser — all thanks to this internal container-to-container workflow.

# Docker Compose Handles Internal Networking
Docker Compose allows all containers to talk to each other by their service names (like php, mysql) — no IPs or host changes needed.


Container	Role
1.Nginx	Receives web requests, serves static files, and forwards PHP scripts to PHP-FPM
2.PHP-FPM	Executes .php scripts, connects to MySQL, generates HTML
3.MySQL	Stores and provides data to PHP via queries
