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

``
yum install https://github.com/MediaBrowser/Emby.Releases/releases/download/4.7.11.0/emby-server-rpm_4.7.11.0_x86_64.rpm
``

Puis on s'y connecte:

http://10.105.1.2:8096

🌞 Une fois en place, mettez en évidence :

``
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
``

