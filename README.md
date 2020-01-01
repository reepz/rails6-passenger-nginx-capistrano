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

1. adduser <appname>
2. adduser <appname> sudo
3. su <appname>
4. cd ~
5. mkdir .ssh & cd .ssh
6. vim authorized_keys   -> copy public key (from machine you want to connect to via ssh)
7. sudo systemctl restart ssh6d
exit
8. ssh <appname>@1.2.3.4
  
* Database creation

* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions
cap production deploy
* ...
