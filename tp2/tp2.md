# TP2 pt. 1 : Gestion de service

## I. Un premier serveur web

**1. Installation**

🌞 **Installer le serveur Apache**

    [ily@web_tp2_linux ~]$ apachectl status
    ● httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: >
       Active: active (running) since Wed 2021-10-06 11:20:05 CEST; 27min ago

    [ily@web_tp2_linux system]$ ss -anltp
    State     Recv-Q    Send-Q         Local Address:Port         Peer Address:Port    Process
    LISTEN    0         128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=838,fd=5))
    LISTEN    0         128                     [::]:22                   [::]:*        users:(("sshd",pid=838,fd=7))
    LISTEN    0         128                        *:80                      *:*        users:(("httpd",pid=868,fd=4),("httpd",pid=865,fd=4),("httpd",pid=864,fd=4),("httpd",pid=836,fd=4))

start apache on boot :

    sudo systemctl enable httpd
    Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

🌞 **TEST**

    [ily@web_tp2_linux ~]$ [ily@web_tp2_linux ~]$ curl localhost
    <!doctype html>
    <html>
      <head>
        <meta charset='utf-8'>
        <meta name='viewport' content='width=device-width, initial-scale=1'>
        <title>HTTP Server Test Page powered by: Rocky Linux</title>


![enter image description here](https://cdn.discordapp.com/attachments/889061317321838627/895252833664909342/testserver.png)

---
**2. Avancer vers la maîtrise du service**

🌞 **Le service Apache...**

    [ily@web_tp2_linux home]$ cat /usr/lib/systemd/system/httpd.service
    [Unit]
    Description=The Apache HTTP Server
    Wants=httpd-init.service
    After=network.target remote-fs.target nss-lookup.target httpd-init.service
    Documentation=man:httpd.service(8)

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

    [ily@web_tp2_linux conf]$ cat httpd.conf
    User apache

    [ily@web_tp2_linux conf]$ ps -ef
    apache       863     836  0 11:46 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache       864     836  0 11:46 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache       865     836  0 11:46 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    apache       868     836  0 11:46 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND


    [ily@web_tp2_linux www]$ ls -al
    drwxr-xr-x.  2 root root    6 Jun 11 17:35 html

🌞 **Changer l'utilisateur utilisé par Apache**

    [root@web_tp2_linux html]# cat /etc/passwd
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    ilyweb:x:1004:1004::/home/ilyweb:/sbin/nologin

    [ily@web_tp2_linux ~]$ cat /etc/httpd/conf/httpd.conf
    ServerRoot "/etc/httpd"
    Listen 80    
    Include conf.modules.d/*.conf
    User ilyweb
    Group ilyweb

		[ily@web_tp2_linux]$ ps -ef
    ilyweb       863     839  0 23:04 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    ilyweb       864     839  0 23:04 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    ilyweb       865     839  0 23:04 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    ilyweb       867     839  0 23:04 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND

🌞 **Faites en sorte que Apache tourne sur un autre port**

- edit port to 75

```
    [ily@web_tp2_linux ~]$ cat /etc/httpd/conf/httpd.conf
    Listen 75
```
- checking

```
    [ily@web_tp2_linux ~]$ sudo firewall-cmd --list-all
    public (active)
      ports: 22/tcp 75/tcp

    [ily@web_tp2_linux ~]$ apachectl status
    ● httpd.service - The Apache HTTP Server
       Status: "Started, listening on: port 75"

    [ily@web_tp2_linux ~]$ ss -anlpt
    State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
    LISTEN        0             128                        0.0.0.0:22                      0.0.0.0:*
    LISTEN        0             128                              *:75                            *:*
    LISTEN        0             128                           [::]:22                         [::]:*
```

 - test commande curl avec port 75 

```
[ily@web_tp2_linux ~]$ curl webserver.com:75
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
```

![enter image description here](https://cdn.discordapp.com/attachments/889061317321838627/895421429347123271/testserver2.png)

## II. Une stack web plus avancée

**1. Intro**

https://docs.rockylinux.org/guides/cms/cloud_server_using_nextcloud
https://www.howtoforge.com/how-to-install-nextcloud-on-rocky-linux/#installing-and-configuring-mariadb

![enter image description here](https://cdn.discordapp.com/attachments/889061317321838627/895592218298052650/error.png)
