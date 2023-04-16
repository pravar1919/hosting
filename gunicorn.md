1. SSH into your server

```
ssh root@104.248.231.241
```

2. Install & Start Supervisor

```
sudo apt-get update -y

sudo apt-get install supervisor -y

sudo service supervisor start
```

3. Install Gunicorn in our Working Directory's Virtualenv

```
/var/www/myproject/bin/python -m pip install gunicorn
```

4. Verify Gunicorn works

```
/var/www/myproject/bin/gunicorn --workers 3
```

5. Create a Supervisor Process

With supervisor, we can run step 4 automatically, restart it if it fails, create logs for it, and start/stop it easily.

Basically, this ensures that our web server will continue to run if you push new code, server reboots/restarts/goes down and back up, etc.

Of course, if a catastrophic error occurs (or if bad code is in your Django project) then this process might fail as well.

All supervisor processes go in:

```
/etc/supervisor/conf.d/
```

So, if you ever need to add a new process, you'll just add it there.

Let's create our project's gunicorn configuration file for supervisor.

```
touch /etc/supervisor/conf.d/myproject-gunicorn.conf
```

Now, let's add the base settings:

```
[program:myproject_gunicorn]
user=root
directory=/var/www/myproject/src/
command=/var/www/myproject/bin/gunicorn --workers 3 --bind unix:myproject.sock cfehome.wsgi:application

autostart=true
autorestart=true
stdout_logfile=/var/log/myproject/gunicorn.log
stderr_logfile=/var/log/myproject/gunicorn.err.log
```

6. Update Supervisor

```
sudo supervisorctl reread
sudo supervisorctl update
```

7. Check Our Supervisor Program/Process Status

```
sudo supervisorctl status myproject_gunicorn
```

A few other useful commands (again):

```
sudo supervisorctl start myproject_gunicorn
sudo supervisorctl stop myproject_gunicorn
sudo supervisorctl restart myproject_gunicorn
```
