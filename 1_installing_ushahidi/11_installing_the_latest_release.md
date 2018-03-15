## 1.1 Installing the latest release {#1-1-installing-on-heroku}

The release bundles are pre-built compressed files for you, which don't require further building or downloading. These files bundles are available from the [platform-release](https://github.com/ushahidi/platform-release/releases) repository in Github. The files are named ushahidi-platorm-release-vX.Y.Z.tar.gz .

The installation procedure will vary depending on your setup, but the requirements in all cases are

* A web server supporting PHP
  * This can be apache2, nginx or a hosting provider
* PHP invokable from command line
* The following PHP modules installed:
  * curl, json, mcrypt, mysqli, pdo, pdo\_mysql, imap and gd
* A MySQL database server

These instructions assume that you know how to create a database in your MySQL server and obtain user credentials with access to such database.

The instructions and example commands are written specifically for Debian Linux or a derivative of it \(Ubuntu, Mint, etc\). You may have to adjust some things if you are installing on a different flavour of Linux, or a different OS.

### 1.1.1 Apache 2 with mod\_php

1. Ensure `mod_rewrite` is installed and enabled in your apache server.

2. Copy into your document root the contents of the`html/`folder after unzipping the ushahidi-platform-release-vX.Y.Z.tar.gz bundle file.

3. The `dist/` folder contains the suggested configurations for the virtual host \(`apache-vhost.conf`\). The configs are quite default, you just need to ensure that there is an `AllowOverride` directive set to `All` for your document root \(where the app has been unzipped\).

4. Create a `platform/.env` file with your database credentials, such as:

   ```
   DB_HOST=<address of your MySQL server>

   DB_NAME=<name of the database in your server>

   DB_USER=<user to connect to the database>

   DB_PASS=<password to connect to the database>

   DB_TYPE=MySQLi
   ```

5. Run the database migrations, execute this command from the `platform` folder:

   ```
   ./bin/phinx migrate -c application/phinx.php
   ```

6. Ensure that the folders `logs`, `cache` and `media/uploads` under `platform/application` are all owned by the user that the web server is running as.

   * i.e. in Debian derived Linux distributions, this user is `www-data`, belonging to group `www-data`, so you would run:

     ```
     chown -R www-data:www-data platform/application/{logs,cache,media/uploads}
     ```

7. Set up the cron jobs for tasks like receiving reports and sending e-mail messages.

   * You'll need to know again which user your web server is running as. We'll assume the Debian standard 
     `www-data`
      here.
   * Run the command `crontab -u www-data -e` and ensure the following lines are present in the crontab:

     ```
     MAILTO=<your email address for system alerts>

     */5 * * * * cd <your document root>/platform && ./bin/ushahidi dataprovider outgoing >> /dev/null
     */5 * * * * cd <your document root>/platform && ./bin/ushahidi dataprovider incoming >> /dev/null
     */5 * * * * cd <your document root>/platform && ./bin/ushahidi savedsearch >> /dev/null
     */5 * * * * cd <your document root>/platform && ./bin/ushahidi notification queue >> /dev/null
     */5 * * * * cd <your document root>/platform && ./bin/ushahidi webhook send >> /dev/null
     ```

8. Restart your apache web server and access your virtual host. You should see your website and be able to login with the credentials user name `admin` and password `admin`

   * **Make sure to change the credentials. Specially if the website is exposed to be accessed by anyone other than you.**

### 1.1.2 nginx with PHP

The procedure is pretty similar to the one detailed for apache above, with the following exceptions.

* Step 1: `mod_rewrite` is specific for Apache, in nginx the module is named `ngx_http_rewrite_module`. It's usually included and enabled.
* Step 2: instead of configuring Apache, you would need to configure `nginx-site.conf` in the `dist` folder. You would usually drop this file in a place where it's included from the main configuration file. It assumes php-fpm is listening in port 9000 of localhost.
* The default php-fpm configuration should work. Most importantly, you need to ensure the `listen` directive matches the `fastcgi_pass`directive in the nginx host configuration file.
* Once you are done, restart both your nginx and php-fpm services.

### 1.1.3 Shared hosting \(Cpanel, Dreamhost, Bluehost, etc\)

In general, the instructions for apache can be taken as a guideline. Each shared hosting provider comes with their own set of particularities, so we can only provide general directions here. In all cases, you'll need to ensure that:

* Decompress the release file and place the contents of the `html` folder in the webroot of your shared hosting domain or subdomain.
* Create a database for your website and write the access details in the `.env` file \(as per step 4 of Apache 2 instructions\)
* You have command line access \(SSH\) in order to run the `phinx` database migration utility in step 5 of Apache 2 instrucions.
* A URL rewriting mechanism has to be in place so that
  * Requests to `/platform/api/v3/*` are to be forwarded to the `index.php` script inside`/platform/httpdocs`
  * When invoking that script, the `api/v3/*` part of the url should be passed to the script into the a `$_SERVER` or environment variable.
  * If your host uses Apache and supports `.htaccess` files, most of this should be taken care of for you.



