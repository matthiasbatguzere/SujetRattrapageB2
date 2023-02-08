# SujetRattrapageB2


0. Le lab

Nous avons configuré 3 VMs:
  
  -🖥️ web.peche.linux
  
  -🖥️ backup.peche.linux
  
  -🖥️ proxy.peche.linux
  

I. Setup

      1. Emby server
      
🌞 Mettre en place le serveur Emby en suivant la doc officielle

On lance la commande suivante pour installer Emby:

```
yum install https://github.com/MediaBrowser/Emby.Releases/releases/download/4.7.11.0/emby-server-rpm_4.7.11.0_x86_64.rpm
```

Puis on s'y connecte:

http://10.105.1.2:8096

🌞 Une fois en place, nous pouvons mettre en évidence :


          -quel service est démarré:
          
```
[matt@localhost ~]$ sudo systemctl status emby-server
● emby-server.service - Emby Server is a personal media server with apps on jus>
     Loaded: loaded (/usr/lib/systemd/system/emby-server.service; enabled; vend>
     Active: active (running) since Wed 2023-02-08 12:25:06 CET; 40s ago
   Main PID: 709 (EmbyServer)
      Tasks: 17 (limit: 4638)
     Memory: 194.5M
        CPU: 2.393s
     CGroup: /system.slice/emby-server.service
             └─709 /opt/emby-server/system/EmbyServer -programdata /var/lib/emb
```


          -comment consulter les logs de Emby:
          

```
[matt@localhost ~]$ sudo journalctl |grep emby-server
Feb 08 12:25:06 localhost.localdomain emby-server[709]: Info Main: Application path: /opt/emby-server/system/EmbyServer.dll
Feb 08 12:25:06 localhost.localdomain emby-server[709]: Info Main: Emby
Feb 08 12:25:06 localhost.localdomain emby-server[709]:         Command line: /opt/emby-server/system/EmbyServer.dll -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdetect -ffmpeg /opt/emby-server/bin/ffmpeg -ffprobe /opt/emby-server/bin/ffprobe -restartexitcode 3 -updatepackage emby-server-rpm_{version}_x86_64.rpm
```


           -quel processus sont lancés par ce service:
        
        
```
[matt@localhost ~]$ sudo ps -ef | grep emby-server
emby         709       1  0 12:25 ?        00:00:03 /opt/emby-server/system/EmbyServer -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdetect -ffmpeg /opt/emby-server/bin/ffmpeg -ffprobe /opt/emby-server/bin/ffprobe -restartexitcode 3 -updatepackage emby-server-rpm_{version}_x86_64.rpm
```

          -sur quel(s) port(s) ce(s) processus écoute(nt):
          
```
tcp    LISTEN  0       512                                 *:8096                 *:*       users:(("EmbyServer",pid=709,fd=193))
```

          -avec quel utilisateur est lancé le(s) processus:
          
 L'utilisateur utilisé se nomme emby
 
 
 🌞 Accéder à l'interface Web
 
On ouvre un port firewall:

```
[matt@localhost yum.repos.d]$ sudo firewall-cmd --add-port=8096/tcp --permanent
success
[matt@localhost yum.repos.d]$ sudo firewall-cmd --reload
success
[matt@localhost yum.repos.d]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 22/tcp 8096/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

On termine l'installation sur notre page d'installation:

http://10.105.1.2:8096

🌞 Configuration d'un répertoire pour accueillir nos médias

Création d'un répertoire /srv/media/:

```
[matt@localhost ~]$ sudo mkdir -p /srv/media/
```

On choisis le user a qui le fichier apprtiendra:

```
[matt@localhost ~]$ sudo chown emby /srv/media/
```

On lui donne la permission de lire le fichier:

```
[matt@localhost ~]$ sudo chmod +r /srv/media
```

On upload le morceau de musique sur la VM depuis l'invite de commande sur notre PC:

PS C:\WINDOWS\system32> scp "C:\Users\Byaku\Desktop\test.mp3" matt@10.105.1.2:/srv/media

Maintenant, nous pouvons aller sur l'interface web de Emby, créer une bibliotheque Musique, lui donner le chemin d'accés (/srv/media), et on refresh, on peut donc voir que notre musique apparait dans le dossier musique, et nous pouvons l'écouter :)


2. Reverse proxy


🌞 Mise en place de NGINX

```
[matt@localhost ~]$ sudo dnf install nginx
[sudo] password for matt:
Last metadata expiration check: 2:29:10 ago on Wed Feb  8 16:13:46 2023.
Package nginx-1:1.20.1-13.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

```
[matt@localhost ~]$ sudo systemctl start nginx
```


On regarde sur quel port nginx écoute:

```
[matt@localhost ~]$ sudo ss -laputrn | grep nginx
tcp   LISTEN 0      511                         0.0.0.0:80        0.0.0.0:*     users:(("nginx",pid=1270,fd=6),("nginx",pid=1269,fd=6))
tcp   LISTEN 0      511                            [::]:80           [::]:*     users:(("nginx",pid=1270,fd=7),("nginx",pid=1269,fd=7))
```


On ouvre un port pour les transfert vers nginx:
```
[matt@localhost ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```

```
[matt@localhost ~]$  ps -ef | grep nginx
root        1269       1  0 18:44 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1270    1269  0 18:44 ?        00:00:00 nginx: worker process
matt        1287    1230  0 18:49 pts/0    00:00:00 grep --color=auto nginx
```

➜ Configurer NGINX

création d'un fichier de configuration NGINX:

```
    [matt@proxy.peche.linux]$ sudo cat conf.d/proxy.conf
server {
# On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name web.tp2.linux;

    # Port d'écoute de NGINX
    listen 80;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.105.1.1:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
10.105.1.3        emby.peche.linux





3. Backup

🌞 Mettre en place un serveur NFS

On crée un repertoire backups/music/ dans notre machine 🖥️ backup.peche.linux

```
[matt@localhost ~]$ sudo mkdir -p /backups/music/
```









   
   


