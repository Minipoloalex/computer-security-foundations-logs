### WordPress CTF

First we had to navigate through the website to find the users that existed, the Wordpress version and the plugins versions.  
In the landing page we already can see that there are at least two users, and one of them is the admin user.  
After searching for a while, we found that if you go to Shop -> WordPress Hosting -> Additional Information you can see the Wordpress version and the plugins versions:

- Wordpress version: 5.8.1
- WooCommerce version: 5.7.1
- Booster for WooCommerce version: 5.4.3

After searching the web for vulnerabilities in Wordpress and WooCommerce unsuccessfully, we found a critical vulnerability in the Booster for WooCommerce plugin, which allows access to the admin panel without authentication. The CVE is CVE-2021-34646 and we finally found the correct flag.  

We then searched for exploits online and we came across https://exploit-db.com/exploits/50299 which is a python script that, provided the URL and the admin user ID, outputs a link to the admin panel. We then used the script to get the link and we were able to access the admin panel and see the hidden flag in the private posts.