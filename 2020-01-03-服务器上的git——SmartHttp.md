# 服务器上的git——SmartHttp

[toc]

## 一、原理

设置Smart Http需要在服务器上启用一个Git自带的名为`git-http-backend`的CGI脚本。

## 二、使用Apache作为CGI服务器

```
sudo apt-get install apache2 apache2-utils
-- 启动 mod_cgi mod_alias mod_env 
sudo a2enmod cgi alias env
```



## 三、将目录用户组设置为`www-data`

```
sudo chown -R www-data:www-data /home/parallels/mywp/git
```

## 四、让`git-http-backend`作为web服务器对 `/git` 路径请求的处理器

`vi /etc/apache2/apache2.conf`

```
SetEnv GIT_PROJECT_ROOT /home/parallels/mywp/git
SetEnv GIT_HTTP_EXPORT_ALL
ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
```

```
<Files "git-http-backend">
    AuthType Basic
    AuthName "Git Access"
    AuthUserFile /home/parallels/mywp/git/.htpasswd
    Require expr !(%{QUERY_STRING} -strmatch '*service=git-receive-pack*' || %{REQUEST_URI} =~ m#/git-receive-pack$#)
    Require valid-user
</Files>
```

重启apache服务器：

> systemctl restart apache2

## 五、添加授权用户

```
htpasswd -c /home/parallels/mywp/git/.htpasswd lifei01
```

测试添加一个用户为： lifei01 ，密码为 123456  的git用户

```shell
parallels@hefrankeleyn:/etc/apache2$ htpasswd -c /home/parallels/mywp/git/.htpasswd lifei01
New password: 
Re-type new password: 
Adding password for user lifei01
```

## 六、遇到的问题

```
error: remote unpack failed: unable to create temporary object directory
To http://10.211.55.3/git/oneGitPro.git
 ! [remote rejected] master -> master (unpacker error)
error: failed to push some refs to 'http://10.211.55.3/git/oneGitPro.git'
```

解决：

> sudo chown -R www-data:www-data /home/parallels/mywp/git

参考：

> https://www.aneasystone.com/archives/2018/12/build-your-own-git-server.html

