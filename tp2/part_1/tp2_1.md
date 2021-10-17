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

![enter image description here](https://cdn.discordapp.com/attachments/889061317321838627/897174514557923339/itworks.png)

**C. Finaliser l'installation de NextCloud**

    [root@web_tp2_linux httpd]# mysql -u nextcloud -h10.102.1.12 -p
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 19
    Server version: 5.5.5-10.3.28-MariaDB MariaDB Server
        
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | nextcloud          |
    +--------------------+
    2 rows in set (0.01 sec)
    
    mysql> use nextcloud
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A
    
    Database changed
    mysql> show tables;
    +-----------------------------+
    | Tables_in_nextcloud         |
    +-----------------------------+
    | oc_accounts                 |
    | oc_accounts_data            |
    | oc_activity                 |
    | oc_activity_mq              |
    | oc_addressbookchanges       |
    | oc_addressbooks             |
    | oc_appconfig                |
    | oc_authtoken                |
    | oc_bruteforce_attempts      |
    | oc_calendar_invitations     |
    | oc_calendar_reminders       |
    | oc_calendar_resources       |
    | oc_calendar_resources_md    |
    | oc_calendar_rooms           |
    | oc_calendar_rooms_md        |
    | oc_calendarchanges          |
    | oc_calendarobjects          |
    | oc_calendarobjects_props    |
    | oc_calendars                |
    | oc_calendarsubscriptions    |
    | oc_cards                    |
    | oc_cards_properties         |
    | oc_collres_accesscache      |
    | oc_collres_collections      |
    | oc_collres_resources        |
    | oc_comments                 |
    | oc_comments_read_markers    |
    | oc_dav_cal_proxy            |
    | oc_dav_shares               |
    | oc_direct_edit              |
    | oc_directlink               |
    | oc_federated_reshares       |
    | oc_file_locks               |
    | oc_filecache                |
    | oc_filecache_extended       |
    | oc_files_trash              |
    | oc_flow_checks              |
    | oc_flow_operations          |
    | oc_flow_operations_scope    |
    | oc_group_admin              |
    | oc_group_user               |
    | oc_groups                   |
    | oc_jobs                     |
    | oc_known_users              |
    | oc_login_flow_v2            |
    | oc_mail_accounts            |
    | oc_mail_aliases             |
    | oc_mail_attachments         |
    | oc_mail_classifiers         |
    | oc_mail_coll_addresses      |
    | oc_mail_mailboxes           |
    | oc_mail_message_tags        |
    | oc_mail_messages            |
    | oc_mail_provisionings       |
    | oc_mail_recipients          |
    | oc_mail_tags                |
    | oc_mail_trusted_senders     |
    | oc_migrations               |
    | oc_mimetypes                |
    | oc_mounts                   |
    | oc_notifications            |
    | oc_notifications_pushtokens |
    | oc_oauth2_access_tokens     |
    | oc_oauth2_clients           |
    | oc_preferences              |
    | oc_privacy_admins           |
    | oc_properties               |
    | oc_recent_contact           |
    | oc_richdocuments_assets     |
    | oc_richdocuments_direct     |
    | oc_richdocuments_wopi       |
    | oc_schedulingobjects        |
    | oc_share                    |
    | oc_share_external           |
    | oc_storages                 |
    | oc_storages_credentials     |
    | oc_systemtag                |
    | oc_systemtag_group          |
    | oc_systemtag_object_mapping |
    | oc_talk_attendees           |
    | oc_talk_bridges             |
    | oc_talk_commands            |
    | oc_talk_guestnames          |
    | oc_talk_internalsignaling   |
    | oc_talk_rooms               |
    | oc_talk_sessions            |
    | oc_text_documents           |
    | oc_text_sessions            |
    | oc_text_steps               |
    | oc_trusted_servers          |
    | oc_twofactor_backupcodes    |
    | oc_twofactor_providers      |
    | oc_user_status              |
    | oc_user_transfer_owner      |
    | oc_users                    |
    | oc_vcategory                |
    | oc_vcategory_to_object      |
    | oc_webauthn                 |
    | oc_whats_new                |
    +-----------------------------+
    99 rows in set (0.00 sec)
    
    mysql> SELECT User, Host, Password FROM mysql.user;
    ERROR 1142 (42000): SELECT command denied to user 'nextcloud'@'10.102.1.11' for table 'user'



