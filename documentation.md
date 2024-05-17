# LEMP STACK - Web Solution Implementation

## Preface 

The LEMP is an acronym that describes a Linux operating system, with an Nginx (pronounced like “Engine-X”) web server. The backend data is stored in the MySQL database and the web dynamic process is handled by PHP.

**This procedure demonstrates how to install a LEMP stack on an Ubuntu 24.04 server using EC2 instance on AWS**

## Step 1 

To Implement the Lemp Stack web solution, you will need an AWS account and a virtual server with Ubuntu Server OS. 

1. Create a new EC2 instance of t2.micro family with Ubuntu Server 24.04 LTS (HVM) Image in the AWS Console. 

![EC2 Instance](<Images/SC - 01 EC2 instance creation.png>)

2. Create Key pair (Pem File format) that will be downloaded from the AWS Console while provisioning the server.  

![Key pair Creation](<Images/SC - 02 Key Pair Creation.png>)

 We are going to use the pem Key downloaded to connect our EC2 instance via ssh.

3. Create Security group configuration with the following inbound rules: 

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet. 
- Allow traffic on Port 22 (SSH) with source from any IP address. 
- Allow traffic on Port 443 (HTTPS) with source from anywhere on the internet. 

![Secrity Group](<Images/SC - 03 Security Group.png>)

4. Click Launch instance and Your instance will be created. 

![Instance Launch result](<Images/SC - 04 Instance Launched.png>)

5. To connect your instance via SSH, Go to your EC2 instance in your AWS console. Select the instance, and click connect.

![Connect instance via SSH](<Images/SC - 05 Connect instance via SSH.png>)

6. Go to the Windows Terminal and `cd` into the folder where the `Pem file` was downloaded. Type the commands below to ssh into the instance via your windows Terminal. 

```bash
cd downloads

chmod 400 "Lemp-Keys.pem"

ssh -i "Lemp-Keys.pem" ubuntu@ec2-54-167-68-55.compute-1.amazonaws.com
```

![Connect instance via terminal](<Images/SC - 05 Connect instance via Terminal.png>)


## Step 2 - Installing the NginX Web Server 

In order to display web pages to our site visitors, we are going to employ Nginx, a high performance web server. we will use the `apt` package manager to install this package. 
Since this is our first time using `apt` for this session, we will update the server's package index. 
Following that you can use `apt install` to get NginX installed. 

1. Update Package Manager 

``` bash
sudo apt update

```
![update package manager](<Images/SC - 06 Update Package Manager.png>)

2. Install Nginx Web server 

```bash 
sudo apt install nginx
```

![install nginx](<Images/SC - 07 Instal nginx.png>)

When prompted, enter _Y_ to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your ubuntu web server. 

3. Verify NginX is running 

To verify that Nginx was successfully installed and is running as a service. Tyoe the command: 

```bash
sudo systemctl status nginx 
```

![nginx status](<Images/SC - 08 nginx status.png>)

If it is green and running, then you did everything correctly - you have launched you web server in the cloud. 

4. Access the server locally in our Ubuntu shell

Before we can receive any traffic by our web server, we need to open `TCP port 80` which is the default port that web browsers use to access web pages in the internet. 

Our server is running and we can access it locally from the internet (Source 0.0.0.0/0)

To check if we can access it locally in our Ubuntu shell, run: 

```bash 
curl http://localhost:80
or 
curl http://127.0.0.1:80
```

![nginx running](<Images/SC - 09 running nginx.png>)

5. Test if Nginx can respond to request from the Internet 

Copy the Public Ip address from the AWS Web console. 

![IP copied](<Images/SC - 10 Ipv4 address copied.png>)

Open a web broswer of your choice and try to access Nginx by pasting the Public IP in the url. 

```bash 
http://54.167.68.55/
```
![browser result](<Images/SC - 11 url result.png>)

The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default. 

If you see the page sampled above, then your web server is now correctly installed and acessible through your firewall.  

## Step 3 - Installing MySQL

We need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database.

1. Again use `apt` to acquire and install this software

```bash 
sudo apt install mysql-server
```

when prompted, confirm installation by typing _Y_ , and then _ENTER_.

![installing sql](<Images/SC - 12 mysql install.png>)

2. Verify that mysql is running with the commands below

Type the command below to verify MySQL is running. 

```bash
sudo systemctl status mysql
```

![SQL status](<Images/SC - 13 mysql status.png>)

3. Log into MySQL Console 

When the installation is finished. Log in to MySQL console by typing 

```bash
sudo mysql
```

This will connect to the MYSQL server as the administrative database user **root**, which is inferred by the use of *sudo* when running this command. You will see output like this: 

![MySQL Login](<Images/SC - 14 mysql login.png>)

4. Set a Password for the Root User 

Before running the script you will set a password for the **root** user, using *mysql_native_password* as default authentication method. You can define the user paswword as anything figure. 

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '12345678';
```

Here the user's password is defined as "12345678"

![Mysql psswd set](<Images/SC - 15 mysql password.png>)

Exit the MySQL shell 

```bash
exit
```

5. Test Log into MySQL console 

When you're finished, test if you're able to log in to the MySQL console by typing 

```bash 
sudo mysql -p
```
Notice the `-p` flag in this command, which will prompt you for the password. 

![login psswd test](<Images/SC - 16 mysql login-pswd test.png>)

To exit the MySQL console, type: 

```bash 
mysql> exit
```

Your MySQL server is now installed. 

## Step 4 - Installing PHP 

You have Nginx installed to serve your content and MySQL installed to store and manage your data. You will install PHP to process code and generate dynamic content for the web server. 

1. Install PHP and PHP-MySQL

Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You'll need to install *php-fpm*, which stands for "PHP fastCGI process manager", and tell Nginx to pass PHP requests to this software for processing. Additionally, you will need *php-mysql*, a PHP module that allows PHP to communicate with MYSQL-based databases. Core PHP packages will automatically be installed as dependencies. 

To install these 2 packages at once, run: 

```bash 
sudo apt install php-fpm php-mysql
```

![PHP install](<Images/SC - 17 PHP install.png>)

When prompted, type *y* and press *Enter* to confirm installation. 

2. Confirm PHP version

To confirm the PHP version, Type the command below: 

```bash 
php -v
```

![PHP version](<Images/SC - 18 PHP Version.png>)

You now have your PHP components installed. Next, you will configure Nginx to use them 

## Step 5 - Configuring Nginx to use PHP processor 

When using the Nginx web server, we can create server blocks (similar to virtual hosts in apache) to encapsulate configuration details and host more than one domain on a single server. we will use projectLemp as the domain name. 

1. Create the root web directory for the domain as follows: 

On Ubuntu, Nginx has one server block enabled by default and is configured to serve documents out of a directory at */var/www/html*. while this works well for a single site, it can be become difficult to manage if you are hosting multiple sites. Instaed of modifying */var/www/html* we'll create a directory structure within /var/www for the domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites. 

Type this code to create a web directory for the domain

```bash 
sudo mkdir /var/www/projectLEMP
```
![creating project Lemp](<Images/SC - 19 Creation Project LEMP.png>)

2. Assign Ownership with $USER envirnoment variable, 

This assignment ownership will reference your current system user. Type the code below to do that 

```bash 
sudo chown -R $USER:$USER /var/www/projectLEMP 
```
![assignment ownership](<Images/SC - 19 Creation Project LEMP.png>)

3. Create and Open a new configuration file in Nginx's "Sites-available" directory using nano. 

```bash 
sudo nano /etc/nginx/sites-available/projectLEMP
```
To `save` use `ctrl + x` and press `y` then `enter`

![Nano projectLemp](<Images/SC - 20 Nano Project LEMP.png>)

Paste the bare-bones configuration in the config file

```bash 
# /etc/nginx/sites-available/projectLEMP
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

4. Unlink the default configuration file from the /sites-enabled/ directory: 

```bash 
sudo unlink /etc/nginx/sites-enabled/default 
```
Then will tell Nginx to use the configuration next time it is reloaded. 

![unlinking config file](<Images/SC - 24 linking config file.png>)

5. You can test your configuration for syntax error by typing: 

```bash
sudo nginx -t 
```
![Test result](<Images/SC - 23 config file test.png>)

If error is reported, go back to your configuration file to review its contents before continuing. 

6. Reload Nginx to apply the changes: 

```bash 
sudo systemctl reload nginx
```
Your website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected: 

```bash 
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/wwww/projectLEMP/index.html
```
![echo index file](<Images/SC - 23 Creating index file.png>)

![creating index file](<Images/SC - 25 Creating index file content.png>)

7. Result 
Now go to your browser and try to open your website URL using IP address 

```bash 
http://54.234.5.129/
```
![result page](<Images/SC - 26 Result Page.png>)

You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. once you do that, remeber to remove or rename the index.html file from your document root, as it would take precedence over an index.php file be default. 

**Your LEMP stack is now fully configured. in the next step, we'll create a PHP script to test that Nginx in fact able to handle .php files within your newly configured website.**


## Step 6 - Testing PHP with Nginx 
Your LEMP stack should now be completely set up. At this point, your LEMP stack is completely installed and fully operational. 

1. You can test it to validate that Nginx can correctly hand *.php* files off to your PHP processor. 
You can do this by creating a test PHP file in your document root. Open a new file called *info.php*
within your document root in your text editor:

```bash 
nano /var/www/projectLEMP/info.php 
```
![Creating info PHP](<Images/SC - 27 index page.png>)

Type the following lines into the new file. This is valid PHP code that will return information about your server: 

```bash
<?php
phpinfo();
```

You can npw access this page in your web browser by vising the public address you've set up in your Nginx configuration file, followed by /info.php 

![Result page](<Images/SC - 28 PHP result page.png>)

After checking the relevant information about your PHP server through that page, it is best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use `rm` to remove that file: 

```bash 
sudo rm /var/www/projectLEMP/info.php
```
![remove info php file](<Images/SC - 29 remove info php.png>)

## Step 7 - Retrieving data from MySQL database with PHP 
In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from DB and display it. 

1. Create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP. we will create a database named example_database and a user named example_user

2. Connect to MySQL console using root account:
```bash 
sudo mysql -u root -p 
```
![Opeening root user](<Images/SC - 31 Opening sql root user.png>)

3. Create a new database, run the following command from your MySQL console:

```bash 
mysql> CREATE DATABASE 'example_database';
```
![Creating example database](<Images/SC - 30 Creating example database.png>)

4. Create a new user named *example_user*, using mysql_native_password as default authentication method. 
we are defining this user's paasword as *12345678* 

```bash 
mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY '12345678';
```
![Creating example user](<Images/SC - 32 Creating example user.png>)

5. Grant user permission over example_database 

This will give the *example_user* full privileges over the *example_database* database. while preventing this user from creating or modifying other databases on your server. 

```bash 
mysql> GRANT ALL ON example_database. To 'example_user'@'%'; 
```
![Grant all user](<Images/SC - 32 Grant example user access to the database.png>)

6. Test if the new user has the proper permissions by logging into the mySQL console again, this time using the custom user credentials:

```bash
mysql -u example_user -p 
```
![Opening example user](<Images/SC - 33 Opening example user database.png>)

7. Confirm that you have access to the *example_database* database:

```bash 
mysql> SHOW DATABASES;
```

![Show Database](<Images/SC - 34 Show Database.png>)

8. Create a test table named todo_list. From the MySQL console, run the following statement: 

```bash
mysql> CREATE TABLE example_database.todo_list (
mysql> item_id INT AUTO_INCREMENT,
mysql> content VARCHAR(255),
mysql> PRIMARY KEY(item_id)
);
```
![Creating To-do list Table ](<Images/SC - 35 Create Table todo_list.png>)

9. Insert a few rows of content in the test table. 

```bash 
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My second important item");
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My third important item");
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My fourth important item");
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My fifth important item");
```

![Insering into Table](<Images/SC - 36 Insert into the table.png>)

10. To confirm that the data was successfully saved on your table, run: 

```bash 
mysql> SELECT * FROM example_database.todo_list;
```

![Select from Table](<Images/SC - 37 Select from the table.png>)

After confirming that you have valid data in your test table, you can exit mySQL console

```bash
mysql> exit
```

11. Create a PHP script that will connect to MySQL and query the database. 

Create a new PHP file in your custom web root directory. 

```bash 
nano /var/www/projectLEMP/todolist.php 
```
The PHP script connects to the MySQL database and queries for the content of the todo_list table, dispalys the results in a results in a list. If there is a problem with the database connection, it will throw an exception. 

```bash 
<?php
$user = "example_user";
$password = "1234567";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

![creating todolist PHP page](<Images/SC - 38 Nano PHP - Mysql.png>)

Save and close the file when you're done editing. 

12. You can now access this page in your web browser by visiting the domain name or Public IP address configured for your website, followed by /todolist.php:

```bash 
http://54.234.5.129/todolist.php
```

![Final result Page](<Images/SC - 39 Final Result.png>)

# Congratulations!

we have built a flexible foundation for serving PHP websites and applications to your visitors, using Nginx as web server and MySQL as database management system. 

