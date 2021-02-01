1. 下载解压MySQL的压缩包
2. 配置MySQL环境变量
3. 创建ini配置文件
4. 进入bin目录下，运行mysqld -install
5. 初始化mysql的配置，mysqld --initialize
6. 启动mysql的服务，net start mysql，可以进入mysql

7. 修改密码：在ini配置文件[mysqld]下加入 skip-grant-tables，重启mysql服务，运行mysql -u root -p直接进入mysql
8. use mysql;  update user set authentication_string=password("xxxxxx") where user="root";