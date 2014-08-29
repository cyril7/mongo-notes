按照mongodb官方手册，安装so easy！

1. sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
2. echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
3. sudo apt-get update
4. sudo apt-get install mongodb-org

安装上述步骤，进行漫长的下载和等待后，即可安装成功。

mongodb将数据存储在 /var/lib/mongodb，log文件存储在/var/log/mongodb，默认使用mongodb用户，如果更改用户的话，记得修改数据文件和log文件的访问权限。

1. mongodb启动，sudo service mongod start
2. 查看mongodb是否启动成功，可以查看 /var/log/mongodb/mongod.log log文件
3. mongodb停止 sudo service mongod stop
4. mongodb重启 sudo service mongodb restart
5. 





