## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

In the previous project, we configured load balancer with Apache. Here, we shall configure load balancer with nginx and also use SSL/TLS to ensure that connections to our Web solutions are secure and information is encrypted in transit.

SSL and its newer version, TSL – is a security technology that protects connection from MITM (man in the middle) attacks by creating an encrypted session between browser and Web server. It uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

![ARCHY](https://user-images.githubusercontent.com/114196715/202039042-02853fa2-ad3f-4be6-991b-d78881433773.png)

### Task

This project consists of two parts:

1. Configure Nginx as a Load Balancer.
2. Register a new domain name and configure secured connection using SSL/TLS certificates.

STEP 1: CONFIGURE NGINX AS A LOAD BALANCER

* Using the ec2 instance earlier configured for use as an apache load balancer, uninstall apache and update '/etc/hosts' file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses.

![Hosts file](https://user-images.githubusercontent.com/114196715/202039142-7519a589-1d42-4b31-bef6-b82603f247d9.png)

* Do not forget to open HTTP traffic on port 80 and HTTPS on port 443 (to ensure secured connections)

* Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers.

` sudo apt update && sudo apt install nginx `

* Configure Nginx LB using Web Servers’ names defined in /etc/hosts

   Open the default nginx configuration file ` sudo vi /etc/nginx/nginx.conf ` and add the following lines to the HTTP section;

```
upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

```

![nginx conf file](https://user-images.githubusercontent.com/114196715/202039419-212fb6a1-f676-440c-82e8-39666fae3f8b.png)

* Test that the above nginx configuration is okay, ` sudo nginx -t `

* Ensure that the default nginx page is disabled. This is done by unlinking it from the site-enabled dircetory or by using the a2dissite command.

- ` sudo unlink /etc/nginx/sites-enabled/default `
OR
- ` sudo a2dissite /etc/nginx/sites-available/default `

* Restart nginx and make sure the server is up and running.

```
sudo systemctl restart nginx
sudo systemctl status nginx

```
Test that your nginx renders your web server page

![nginx IP](https://user-images.githubusercontent.com/114196715/202039560-8374673a-37db-41f0-92fd-ecaa14478be2.png)

STEP 2: REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

1. In order to get a valid SSL certificate, we need to register a new domain name.

2. Register a new domain name and assign an Elastic IP to your Nginx LB server, then associate your domain name with this Elastic IP. AN elastic IP ensures that we have an unchanging public IP in the event of stopping and restarting our instance.

3. Update A record in your registrar to point to Nginx LB using Elastic IP address.

![A record update](https://user-images.githubusercontent.com/114196715/202039713-3daa6624-d011-49ef-91dd-455e4a89ec8d.png)

Check that your web servers can be reached from your web browser using your domain name using HTTP protocol – ' http://<your-domain-name.com '.

REPORT: Site was unreachable as the domain name is yet to be updated in nginx configuration.


4. Configure Nginx to recognize your new domain name. Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com.

![updated config](https://user-images.githubusercontent.com/114196715/202039836-40c476f2-7bfc-4c62-b024-ae177a9f5e05.png)

Check again that your web servers can be reached from your web browser using your domain name using HTTP protocol – ' http://<your-domain-name.com '.

![loaded unsafe site](https://user-images.githubusercontent.com/114196715/202040525-9e214cd2-35e7-446e-abab-3fd20bcdd6f2.png)

5. Install certbot and request for an SSL/TLS certificate

- We will use snapd to install certbot. Make sure snapd is installed and the service is up and running: ` sudo systemctl status snapd `

- Install certbot: ` sudo snap install --classic certbot `

- Request your certificate, follow the certbot instruction. (you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file )

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx

```

![certbot installed](https://user-images.githubusercontent.com/114196715/202040039-2b007418-25ef-42de-9c79-57491833cbd1.png)

![certbot installed 2](https://user-images.githubusercontent.com/114196715/202040126-d55ea507-b69e-4847-9fdc-a63aca10e391.png)

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>. We should be able to access our website using HTTPS protocol.

![https successful](https://user-images.githubusercontent.com/114196715/202040352-8d1d1189-c1e2-4fce-8fd0-eaa1d4bd51ea.png)

A padlock pictogram appears in the browser’s search string, signifying secured connection. Clicking on the padlock icon brings the details of the certificate issued for our website.

ATTACH SCREENSHOT

6. Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

` sudo certbot renew --dry-run `

It is best practice to have a scheduled job that helps to run the renew command periodically. For this, we shall configure a cronjob to run the command twice a day;

- edit the crontab file with the following command: ` crontab -e ` and add the following line:

` * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1 `

One can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

