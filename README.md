# These are the steps to deploy react and expressJs app with nginx
## Important
In this example i will use example.com as my website so change "example.com" or "example" to your website name
## If you don't have nodejs
To get this version, you can use the apt package manager. Refresh your local package index by typing:
```
sudo apt update
```
install nodejs and npm if you dont have them in the server
```
sudo apt install nodejs
sudo apt install npm
```

To check which version of Node.js 
```
nodejs -v
```
To check which version of npm
```
npm -v
```
#### Or 
Got to this website
https://tecadmin.net/install-latest-nodejs-npm-on-ubuntu/
and follow the instructions 
## step #1

install nginx
```
sudo apt-get install nginx
```

## step #2 
Clone your repository on your machine
you can eiter clone or copy you files
```
sudo mkdir /var/www/example
cd /var/www/example
```
Then what I like to do is to separate frontend from backend so 
```
sudo mkdir frontend
sudo mkdir backend
```
Make sure to clone your frontend in the frontend directory 
and backend in backend directory

## step #3
## For react
run the following commands to build your react app
```
npm install -g create-react-app
npm i
npm i serve -g
npm run build
```
To check if the bulid does works
```
serve -s build
```
Then ctrl + c to close it 
If step 3 caused you problems 
#### OR 
build it on your machine and run
```
npm install -g create-react-app
npm i
npm i serve -g
serve -s build
```
Then copy the file to the server
You could use scp you don't need the node_modules directery dont add it to the zip file
Example
```
scp file.zip remote_username@10.10.0.2:/tmp/
```
install unzip
```
sudo apt install unzip
```
unzip the file
```
unzip /tmp/file.zip -d /var/www/example/
```

## For expressJs
```
npm i
npm i pm2 -g
pm2 start server.js --name example
```
To check the status of the app
```
pm2 status
```

## step #4 you can skip this part if you are root
In your project directery
Since you wouldn’t want to use sudo every time you interact with files in /var/www/[your project name], let’s give your user privileges to these folders. 
Constantly using sudo increases the chances of accidentally trashing your system.
```
sudo gpasswd -a "$USER" www-data
sudo chown -R "$USER":www-data /var/www/example
find /var/www/example -type f -exec chmod 0660 {} \;
sudo find /var/www/example -type d -exec chmod 2770 {} \;
```
## step #5
Create a new site in sites available
this command will create a file that you can edit
```
cd /etc/nginx/sites-available
nano example.com
```
Configure new site
Open example.com in an editor of your choice and paste the following. 
You can safely use an ip address here.
You can also use both ip and domain.
you can copy the folowing code and change things in the bracket
##### For test deployment Without http
```
server {
  listen *:80;
  server_name example.com www.example.com
  server_name server IP; ## put your server IP example 111.111.111.11
  index index.html index.htm;
  client_max_body_size 100M; ## size of files can be sent in the body
  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log;

  location / {
    try_files $uri /index.html =404;
    root /var/www/example/frontend/build;
    }
  location /uploads/ {  ## if you have an upload file so enginx will serve them
    proxy_pass http://localhost:3000/;   ## this is the port you run your backend on
    root /var/www/example/backend/uploads;  ## location of uploads directory 
  }
  location /api/ { ## this is the place to connect you apis
    proxy_pass http://localhost:3000/; ## this is the port you run your backend on
  }

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
}
```
##### With https for real deployment
```
server {
  listen *:80;
  listen 443 ssl;
  listen [::]:443 ssl;
  server_name example.com www.example.com
  server_name server IP; ## put your server IP example 111.111.111.11
  index index.html index.htm;
  client_max_body_size 100M; ## size of files can be sent in the body
  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log;

  location / {
    try_files $uri /index.html =404;
    root /var/www/example/frontend/build;
    }
  location /uploads/ {  ## if you have an upload file so enginx will serve them
    proxy_pass http://localhost:3000/;   ## this is the port you run your backend on
    root /var/www/example/backend/uploads;  ## location of uploads directory 
  }
  location /api/ { ## this is the place to connect you apis
    proxy_pass http://localhost:3000/; ## this is the port you run your backend on
  }

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
}
```
##### If you are doing https follow the steps on the link but skip step 3
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04
### important 
* when you call your api in the frontend the base url will
* https://example.com/api/
* and when you save the uploaded file  base url
* https://example.com/uploads/
## step #6
Symlink config
Nginx needs to be told that a given site is active. To do this, create a symlink in sites-enabled
```
cd /etc/nginx/sites-enabled
ln -s ../sites-available/example.com .
```
## step #7
check if your configuration is ok
```
sudo nginx -t
```
Now start up Nginx!
```
sudo service nginx start
```
If you changed up your repository or made any changes to the configuration, you can restart Nginx with:
```
sudo service nginx restart
```
And you’re done! If you go to your browser and type in the IP address of your server or your domain, you should see your React app live!

# sources
* https://medium.com/@timmykko/deploying-create-react-app-with-nginx-and-ubuntu-e6fe83c5e9e7
* https://codeburst.io/how-to-setup-nginx-for-react-a504f38f95ed
* https://tecadmin.net/install-latest-nodejs-npm-on-ubuntu/
* https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04
