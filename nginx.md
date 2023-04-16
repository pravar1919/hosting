## 1. SSH into your server

```
ssh root@104.248.231.241
```

Replace root@104.248.231.241 with your user / ip

## 2. Install Nginx and ufw

```
sudo apt-get update -y

sudo apt-get install nginx ufw -y
```

## 3. Enable ufw Firewall Defaults

We want ssh and Nginx to work. Nginx Full allows for both http (80) and https (443) connections.

```
sudo ufw allow ssh

sudo ufw allow 'Nginx Full'
```

Enable. Ensure you have ssh above otherwise you will lose your connection and not get it back.

```
sudo ufw enable
```

Check your status

```
sudo ufw status numbered
```

You should see this:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] Nginx Full                 ALLOW IN    Anywhere
[ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 4] Nginx Full (v6)            ALLOW IN    Anywhere (v6)
```

## 4. Remove Nginx Default

After you install nginx like we did in step 2, you can go directly to your server's ip address. You should see some default nginx page.

We can remove that with:

```
rm /etc/nginx/sites-enabled/default
```

## 5. What's my ip?

```
localip=$(hostname  -I | cut -f1 -d' ')
echo $localip
```

Grab this value for the next step. Mine was 104.248.231.241. This should match what you ssh into like in step 1.

## 6. Nginx Configuration

Now, we have gunicorn as our project server. Nginx needs to use the guincorn socket. From this post we set the socket file to be located on /var/www/myproject/src/myproject.sock

We'll set our server_name to 104.248.231.241 for now. We'll do a custom domain as well as https in another post.

Create your nginx conf file

```
nano /etc/nginx/sites-available/myproject.conf
```

Add the following with your details.

Replace proxy_pass with http://unix:/your/local/gunicorn/socket/file.sock Replace server_name your your ip address from Step 5.

```
server {
    server_name 104.248.231.241;
    listen 80;
    listen [::]:80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/myproject/src/myproject.sock;
        proxy_buffer_size       128k;
        proxy_buffers           4 256k;
        proxy_read_timeout      60s;
        proxy_busy_buffers_size 256k;
        client_max_body_size    2M;
    }
}
```

Now that you have your configuration in sites-available it's time to link it to sites-enabled

```
ln -s /etc/nginx/sites-available/myproject.conf /etc/nginx/sites-enabled/myproject.conf
```

## 7. Reload Daemon and Nginx

```
systemctl daemon-reload

sudo systemctl reload nginx
```

## 8. Update our repo post-receive

if you have gone through <a href="post-receive.md">this</a>

```
cd /var/repo/myproject.git/hooks/
nano post-recieve
```

```
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f


/var/www/myproject/bin/python -m pip -r /var/www/myproject/src/requirements.txt

systemctl daemon-reload

sudo systemctl reload nginx
```

Now when you push a new version of your code, you're nginx system will automatically reload (assuming no errors) without the system missing a beat.
