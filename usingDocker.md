# These are the steps to deploy react and expressJs app with nginx and docker
## Important
In this example i will use example.com as my website so change "example.com" or "example" to your website name

## step #1

Install nginx
```
sudo apt-get install nginx
```
## step #2
Go to your react app directory and build your react app
```
npm i
```
```
npm run build
```
It wll generate a build file

## step #3
zip you build file and move it to your server 

```
scp file.zip remote_username@serverIP:/tmp/ 
```


## step #4
Clone your repository on your machine
you can eiter clone or copy you files
```
sudo mkdir /var/www/example
cd /var/www/example
```
Then create frontend directory
```
sudo mkdir frontend 
```


## step #4.5 you can skip this part if you are root
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
unzip you build file into your `/var/www/example/frontend`
```
unzip /tmp/file.zip -d /var/www/example/frontend
```
## step #6 
Build your docker file and save your docker file as .img
### important
Make sure you are serving your app on ip 0.0.0.0
```
docker build -t example .
```
```
docker save -o example.img example
```
Move it to the server
```
scp example.img remote_username@serverIP:/tmp/ 
```
## step #7 
load your docker files 
```
docker run -d -p 3000:3000 --net=host example
```
Adding `--net=host` @ill allow your container to connect to the local db
## step #8
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
  index index.html index.htm;
  client_max_body_size 100M; ## size of files can be sent in the body
  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log;

  location / {
    try_files $uri /index.html =404;
    root /var/www/example/frontend/build;
    }

  location /api/ { ## this is the place to connect you apis
    proxy_pass http://localhost:3000/; ## this is the port you run your backend on
  }

}
```
##### With https for real deployment
```
server {
  listen *:80;
  listen 443 ssl;
  listen [::]:443 ssl;
  server_name example.com www.example.com ## put your url
  index index.html index.htm;
  client_max_body_size 100M; ## size of files can be sent in the body
  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log;

  location / {
    try_files $uri /index.html =404;
    root /var/www/example/frontend/build;
    }
  location /api/ { ## this is the place to connect you apis
    proxy_pass http://localhost:3000/; ## this is the port you run your backend on
  }

}
```
##### if you are using socket io add this part
```
  location ~* \.io {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy false;

      proxy_pass http://localhost:3000; ## this is the port you run your backend on
      proxy_redirect off;

      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }
```

##### If you are doing https follow the steps on the link but skip step 3
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04
### important 
* when you call your api in the frontend the base url will be https://example.com/api/
* if you have more than one server you con duplicate these lines and change the word `api` and port number
```
  location /api/ { ## this is the place to connect you apis
    proxy_pass http://localhost:3000/; ## this is the port you run your backend on
  }
```
## step #9
Symlink config
Nginx needs to be told that a given site is active. To do this, create a symlink in sites-enabled
```
cd /etc/nginx/sites-enabled
ln -s ../sites-available/example.com .
```
## step #10
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
* https://docs.docker.com/engine/reference/commandline/build/#:~:text=The%20docker%20build%20command%20builds,a%20file%20in%20the%20context.
