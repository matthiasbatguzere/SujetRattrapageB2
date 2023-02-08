# SujetRattrapageB2


0. Le lab

Nous avons configuré 3 VMs:
  
  -🖥️ web.peche.linux
  
  -🖥️ backup.peche.linux
  
  -🖥️ proxy.peche.linux
  

I. Setup

      1. Emby server
      
🌞 Mettez en place le serveur Emby en suivant la doc officielle

On lance la commande suivante pour installer Emby:

```
yum install https://github.com/MediaBrowser/Emby.Releases/releases/download/4.7.11.0/emby-server-rpm_4.7.11.0_x86_64.rpm
```

Puis on s'y connecte:

http://10.105.1.2:8096

🌞 Une fois en place, mettez en évidence :


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
 
 
 🌞 Accédez à l'interface Web
 
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

🌞 Configurer un répertoire pour accueillir vos médias

créer un répertoire /srv/media/:

```
[matt@localhost ~]$ sudo mkdir -p /srv/media/
```

faites le appartenir au bon user:

```
[matt@localhost ~]$ sudo chown emby /srv/media/
```

donnez lui les permissions les plus restrictives possibles, le minimumm pour que tout fonctionne correctement:

```







   
   


