# Rails 6.0.0 skeleton app ready for capistrano deployment on Ubuntu 18.04

* Ruby version
2.6.4
* System dependencies
postgresql
nginx
passenger
rbenv
cerbot

* Configuration
create naked Ubuntu 18.04 LTS on any cloud provider

ssh root@1.2.3.4

1. adduser deploy
2. adduser deploy sudo
3. su deploy
4. cd ~
5. mkdir .ssh & cd .ssh
6. vim authorized_keys   -> copy public key (from machine you want to connect to via ssh)
7. sudo systemctl restart sshd
exit
8. ssh deploy@1.2.3.4
  
##### only allow deploy user login for optional increased security
1. sudo /etc/ssh/sshd_config
2. PermitRootLogin no
3. AllowUsers deploy

##### Install dependencies for compiiling Ruby along with Node.js and Yarn
1. curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
2. curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
3. echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
4. sudo apt-get update
5. sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev dirmngr gnupg apt-transport-https ca-certificates redis-server redis-tools nodejs yarn
7. sudo apt-get install libpq-dev *(required by postgresql)

##### Installing ruby via rbenv 

1. git clone https://github.com/rbenv/rbenv.git ~/.rbenv
2. echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
3. echo 'eval "$(rbenv init -)"' >> ~/.bashrc
4. git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
5. echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
6. git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars
7. exec $SHELL
8. rbenv install 2.6.4
9. rbenv global 2.6.4
10. ruby -v
11. gem install bundler
12. bundle -v

##### Installing NGINX & Passenger

1. sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
2. sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
3. sudo apt-get update
4. sudo apt-get install -y nginx-extras libnginx-mod-http-passenger
5. if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
6. sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
7. passenger-config validate-install
8. passenger-memory-stats

##### Now that we have NGINX and Passenger installed, we need to point Passenger to the correct version of Ruby.

1. sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
2. passenger_ruby /home/<appname>/.rbenv/shims/ruby;
3. sudo systemctl restart nginx
  
##### Create server block in nginx to host our site, make sure deploy user is owner of the root folder

1. sudo chown deploy:deploy /etc/nginx/sites-avaialble/appname
2. sudo vim /etc/nginx/sites-avaialble/appname
```
server {
  listen 80;
  listen [::]:80;

  server_name <appname>.domain;
  root /var/www/<appname>/current/public;

  if ($http_host ~* "^(?!www\.).*$") {
    return 301 https://www.$http_host$request_uri;
  }
  if ($scheme = http) {
    return 301 https://$host$request_uri;
  }

  passenger_enabled on;
  passenger_app_env production;

  location /cable {
    passenger_app_group_name myapp_websocket;
    passenger_force_max_concurrent_requests_per_process 0;
  }

  # Allow uploads up to 100MB in size
  client_max_body_size 100m;

  location ~ ^/(assets|packs) {
    expires max;
    gzip_static on;
  }
}
```

3. sudo systemctl restart nginx

##### CERTIFICATES certbot

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot python-certbot-nginx
sudo certbot --nginx
```

##### ENV Variables (improve and decide on how to manage .rbenv-vars vs secrets.yml vs system levels env variabales)
###### For Postgres
`DATABASE_URL=postgresql://<appname>:PASSWORD@127.0.0.1/myapp`

###### For MySQL
`DATABASE_URL=mysql2://<appname>:$omeFancyPassword123@localhost/myapp`
```
RAILS_MASTER_KEY=ohai
SECRET_KEY_BASE=1234567890

STRIPE_PUBLIC_KEY=x
STRIPE_PRIVATE_KEY=y
```

* Database creation

* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
cap production deploy
* ...
