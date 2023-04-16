Step 1. Install git locally and create local repo

Then,

```
mkdir Dev
cd Dev
mkdir myproject
cd myproject
git init
```

Step 2. SSH into Live Server

```
ssh root@104.248.231.241
```

Step 3. Install git on Server

```
sudo apt-get install git -y
```

Step 4. Create bare git repo

```
cd /var/
mkdir repo
cd /var/repo/
mkdir myproject.git
cd myproject.git
```

Starting fresh?

```
git init --bare
```

Using previous?

```
git clone https://github.com/pravar1919/tcaller.git . --bare
```

Step 5. Add post-receive hook

Hooks are very useful for doing things automatically while working with git.

In our case, we need a hook for after a successful push aka post-receive to unpack our code into a server-side working directory.

Make initial working directory

View all hook samples

```
ls -al /var/repo/myproject.git/hooks/
```

Create the actual post-receive hook

```
cd /var/repo/myproject.git/hooks/
nano post-receive
```

In post-receive

```
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f
```

Make hook executable

```
cd /var/repo/myproject.git/hooks/
chmod +x post-receive
```

Step 7. Add to local git repo

```
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
```

Step 8. Make Changes Locally and Push

```
cd ~/Dev/myproject
echo "hello there" > test.txt
git add 'test.txt'
git commit -m "Local file test"
git push live master
```

Step 10. Need that code on another local computer?

```
cd path/to/local/dir
git init
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
git pull live master
```
