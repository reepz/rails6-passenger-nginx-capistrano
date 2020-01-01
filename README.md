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
ssh root@1.2.3.4

adduser <appname>
adduser <appname> sudo
su <appname>
cd ~
mkdir .ssh & cd .ssh
vim authorized_keys   -> copy public key (from machine you want to connect to via ssh)
sudo systemctl restart sshd
exit
ssh <appname>@1.2.3.4
* Database creation

* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
cap production deploy
* ...
