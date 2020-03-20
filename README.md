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
```
adduser deploy
adduser deploy sudo
su deploy
cd ~
mkdir .ssh & cd .ssh
vim authorized_keys   -> copy public key (from machine you want to connect to via ssh) (change ownership?)
sudo systemctl restart sshd
exit
```
ssh deploy@1.2.3.4

##### only allow deploy user login for optional increased security
```
sudo /etc/ssh/sshd_config
AllowUsers deploy
```

##### Install dependencies for compiiling Ruby along with Node.js and Yarn
```
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev dirmngr gnupg apt-transport-https ca-certificates redis-server redis-tools nodejs yarn
```
##### Installing ruby via rbenv 

```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars
exec $SHELL
rbenv install 2.6.4
rbenv global 2.6.4
ruby -v
gem install bundler
bundle -v
```

##### Installing NGINX & Passenger
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y nginx-extras libnginx-mod-http-passenger
if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
passenger-config validate-install
passenger-memory-stats
```

##### Now that we have NGINX and Passenger installed, we need to point Passenger to the correct version of Ruby.
```
sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
passenger_ruby /home/<appname>/.rbenv/shims/ruby;
sudo systemctl restart nginx
```

##### Create server block in nginx to host our site, make sure deploy user is owner of the root folder
```
sudo chown <appname>:<appname> /etc/nginx/sites-avaialble/<appname>
sudo vim /etc/nginx/sites-avaialble/<appname>
sudo ln -s /etc/nginx/sites-available/test.com /etc/nginx/sites-enabled/
```

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

`sudo systemctl restart nginx`

##### CERTIFICATES certbot

```
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
RAILS_MASTER_KEY=ohai (is that the same as secret key base below and is it required?)
SECRET_KEY_BASE=1234567890

STRIPE_PUBLIC_KEY=x
STRIPE_PRIVATE_KEY=y
```

* Postgresql installation
```
sudo apt-get install postgresql postgresql-contrib libpq-dev
```

* Database initialization
```
sudo su - postgres
createuser --pwprompt deploy
createdb -O deploy myapp
exit
```

##### manually connect to database by running: 
`psql -U deploy -W -h 127.0.0.1 -d myapp`

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
cap production deploy
* ...
