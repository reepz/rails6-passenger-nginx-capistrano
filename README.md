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

#####

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


* Database creation

* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
cap production deploy
* ...
