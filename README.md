*Developed by Mina Hafzalla*

# Automated Tor Network Detection for Nginx
Accepting online payments can be challenging if you are facing high volume of fraudulent transactions; combating online fraud requires various security measures and tactics.

Huge number of fraud payments come from proxy servers that hackers use to hide their real locations and IP addresses. [Tor](https://www.torproject.org/) is one of the very popular proxy and anonymity service providers that hackers misuse to place online fraud orders.

Today, I'm introducing a server-side script written in Bash which of its purpose is to restrict users from checking out on a website if it's being accessed using Tor. The script doesn't block users from navigating a website through Tor; however it only blocks acccess to the checkout process and applies a redirection to another page informing users that orders over proxies are not allowed.

<br />

# Requirements
1. Linux Server.
2. Nginx Web Server.
3. [Geo](http://nginx.org/en/docs/http/ngx_http_geo_module.html) Module for Nginx.
4. SSH access with root/sudo privileges. 
5. Bash Shell. 

*Note: Script has been implemented and tested on CentOS servers. It should work on all other Linux distributions.*

<br />

# How it Works
The script fetches Tor network's IP addresses, defines a custom variable and adds Nginx syntax, then exports the results to another text file. Furthermore, we would need to include that text file inside Nginx main configuration file along with other configuration rules in order for the script to work; detailed explanation follows.

<br />

# Implementation

### Step 1 → Core Code
```shell
cd /usr/local/nginx/conf; wget "https://check.torproject.org/cgi-bin/TorBulkExitList.py?ip=1.1.1.1" -O toripspage.txt; sed '1,3d' toripspage.txt > torips.txt; sed -e 's/$/ 1;/' -i torips.txt; sed -i '1s/^/geo $tor_user {\ndefault 0;\n/' torips.txt; sed -i '$s/$/\n}/' torips.txt
```
The script is developed assuming that Nginx installation path is `/usr/local/nginx`, make sure to modify that accordingly. First, we have to create an *(`.sh`)* file in order to put the code inside it and get it ready for execution.

1. Open your terminal and run `vi /usr/local/nginx/conf/fetchtorips.sh`
2. Press the "i" key on keyboard to start inserting.
3. Copy the above code and paste it in the current file.
4. Press the "Esc" key on keyboard to exit the editing environment.
5. Type `:wq` then press "Enter" on keyboard to save and exit.
6. Run the following code to make script executable `chmod +x /usr/local/nginx/conf/fetchtorips.sh`


### Step 2 → Cron Job
We need to create a cron job to schedule script execution on daily basis to keep our list up-to-date with new Tor IP addresses.

1. Run the command `vi /etc/crontab` via terminal.
2. Press the "i" key on your keyboard to start inserting.
3. Copy this code `0 5 * * * root bin/bash /usr/local/nginx/conf/fetchtorips.sh` and paste it.
4. Press the "Esc" key on your keyboard to exit the editing environment.
5. Type `:wq` and press "Enter" on keyboard to save and exit.

*Note: This will run the cron job at 5:00 am every day.*


### Step 3 → Nginx Configuration
Last thing we need to do is to include our script inside Nginx main configuration file *(`nginx.conf`)* and tell Nginx to restrict access to the checkout page whenever the website is being accessed through Tor.

##### 1. Part (a) → Including the IPs list file in Nginx configuration:
   1. Run the command `vi /usr/local/nginx/conf/nginx.conf` to open nginx configuration file.
   2. Press the "i" key on your keyboard to start inserting/editing.
   3. Copy this line `include torips.txt;` and paste it inside the `http` directive.

##### 2. Part (b) → Restricting access to checkout if Tor is in use:

Assuming your checkout page looks like this: `https://example.com/cart.php`. We would need to use a rewrite rule to tell Nginx if a user is accessing the `cart.php` page through Tor, apply redirection to an error page.

   4. Inside *(`nginx.conf`)* file, under the `Location` directive, add the following code:
   
          if ($tor_user) {
          rewrite ^/cart.php http://example.com/error.php;
          }
          
   5. Press the "Esc" key on your keyboard to exit the editing environment.
   6. Type `:wq` and press "Enter" on keyboard to save and exit.
          
This will block access to the checkout page only and not the whole website. 
*Error.php is your custom page that displays your custom message to users. My preferred message is "Sorry, orders over proxies are not allowed", but you are free to change it to whatever suits your needs.*

   7. Lastly, run this command `/usr/local/nginx/sbin/nginx -s reload` to restart Nginx in order for changes to take effect.

<br />

# Feedback
Should you have any feedback or questions please leave a comment or contact me at mina@netmoly.com
