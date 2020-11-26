LAB SETUP 1.3 (konfiguracija WEB, SHARE i DB mašina) - FTP, NSF, SAMBA


1.Podešavanje WEB mašine: 
    
          yum install vsftpd                                                             .....instaliramo ftp server
          yum install lftp                                                               .....instaliramo alatku za testiranje konektivnosti
          
          useradd -m bozaftp                                                             .....dodamo nekog usera 
          passwd bozaftp                                                                 .....zadamo komandu za password
                password: password                                                            .....dva puta potvrdimo da je password password
                
          lftp bozaftp@localhost                                                         .....pristupamo serveru kao bozaftp user sa lokalnog hosta (provera)
          service vsftpd restart                                                         .....restartujemo ftp servis
          
          iptables -vnL --line-number                                                    .....proverimo pravila zaštitnog zida da vidimo na koje pozicije da ubacimo
                                                                                              nova pravila
          iptables -I INPUT 8 -m state --state NEW -p tcp --dport 20 -j ACCEPT
          iptables -I INPUT 9 -m state --state NEW -p tcp --dport 21 -j ACCEPT           .....ubacujemo dva lanca da omogućimo pristup kroz portove 20 i 21
          iptables-save > /etc/sysconfig/iptables                                        .....snimimo podešavanje
          systemctl reload iptables.service                                              .....ponovo pokrenemo servis iptables
  
          chown -R bozaftp /var/www/int/                                                 .....menjamo vlasnika foldera int (imenujemo usera bozaftp za vlasnika)
          usermod -d /var/www/int bozaftp                                                .....menjamo user korisnika nad navedenim folderom
          
          service vsftpd restart                                                         .....restartujemo ftp servis
          lftp bozaftp@localhost                                                         .....pristupamo serveru kao bozaftp user sa lokalnog hosta 
          ls- la                                                                         .....ako je sve u redu, prikazuje iz lftp sadržaj foldera /var/www/int/
          
          yum install samba                                                              .....instalirati sambu 
          systemctl enable --now smb.service                                             .....omogućiti i pokrenuti sambu
          yum install cifs-utils                                                         .....instaliramo podršku za cifs fajl sistem 
          
          
2.Podešavanje SHARE mašine:

    2.1                                                                         Instaliranje i podešavnje nfs servisa

          yum install nfs-utils                                                          .....instaliramo nfs utils-e
          systemctl enable nfs-server                                                    .....omogućimo nfs server
          
          mkdir /srv/backups/
          mkdir /srv/backups/database                                                    .....kreiramo direktorijum backups i u njemu database folder
          
          vim /etc/exports                                                               .....otvaramo exports konfiguracioni fajl sa vim i u njega upisujemo:
          
               /srv/backups/database  192.168.60.0/24(rw,sync,no_root_squash)                 .....omogućimo korisnicima iz navedene mreže da imaju pristup
                                                                                                   fajlovima u data bazi
                                                  esc -> shift: -> wq                         .....snimimo izmene u fajlu    
                                                  
          systemctl start nfs-server                                                     .....starujemo nfs server 
          
          showmount -e localhost                                                         .....izlistavamo dostupne nfs share fajlove i dobijemo ispis:
          
                Export list for localhost:
                /srv/backups/database 192.168.60.0/24
          
          mount 192.168.60.12:/srv/backups/database/ /mnt                                .....privremeno (dok traje sesija) mountujemo data bazu na mnt direktorijum
          ls -la /mnt                                                                    .....proveravamo da li se sadržaj database foldera pojavljuje u mnt folderu
                                                                                              ako je sve u redo dobili smo dati iskaz:
               total 0
               drwxr-xr-x.  2 root root   6 Jun  4 19:07 .
               dr-xr-xr-x. 17 root root 224 May  9 19:06 ..                                   .....folder je prazan jer u njemu nema još fajlova


          vim /etc/fstab                                                                 .....otvaramo fstab ako hoćemo da se database mountuje uvek prilikom    
                                                                                              pokretanja sistema i upisujemo:

               192.168.60.12:/srv/backups/database     /mnt    nfs     _netdev         0 0         
          
                                                  esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                
          
          nfsstat -s                                                                     .....provera verzije nfs (ako je verzija NFSv4) imamo fiksirane portove i
                                                                                              nema potrebe za dodatnim podešavnjima

          iptables -vnL --line-number                                                    .....proverimo pravila zaštitnog zida da vidimo na koje pozicije da ubacimo
                                                                                              nova pravila
          iptables -I INPUT 8 -m state --state NEW -p tcp --dport 2049 -j ACCEPT
          iptables -I INPUT 9 -m state --state NEW -p tcp --dport 111 -j ACCEPT          .....ubacujemo dva lanca da omogućimo pristup kroz portove 2049 i 111
          iptables-save > /etc/sysconfig/iptables                                        .....snimimo podešavanja
          systemctl reload iptables.service                                              .....ponovo pokrenemo servis iptables
          
          
    2.2   Instaliranje i podešavnje samba servisa 
   
   
          yum install samba                                                              .....instaliramo samba servis
          

          vim /etc/samba/smb.conf                                                        .....otvorimo konfiguracionifajl za sambu i dodamo na kraj teksta putanju
                                                                                              ka database folderu wordpress
          [wordpress]
                    comment = wordpress database
                    path = /srv/backups/wordpress
                    read only = no                                                            .....ovaj red dodat kasnije da bi se obezbedila mogućnost editovanja u  
                                                                                                   njemu
                                                  esc -> shift: -> wq                         .....snimimo izmene u fajlu
          systemctl restart smb.service                                                  .....restartujemo servis
          
          mkdir /srv/backups/wordpress                                                   .....napravimo direktorijum na naznačenoj putanji  
          
          ls -ld /srv/backups/wordpress                                                  .....pogledamo permisije foldera wordpress
          chmod 777 /srv/backups/wordpress                                               .....uradimo i izmenu permisija (777 - sve dozvoljeno svima koji pristupaju)
          touch /srv/backups/wordpress/test_fajl                                         .....testiramo tako što kreitamo test fjl - ako uspemo, podešavanje je  
                                                                                              dobro
          
          testparm                                                                       .....komanda kojom izvršimo proveru urađenog (izlistava se prikaz stanja
                                                                                              koje kreira smb.conf konfiguracioni fajl za sambu
          
                Load smb config files from /etc/samba/smb.conf
                rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
                Processing section "[homes]"
                Processing section "[printers]"
                Processing section "[print$]"
                Processing section "[wordpress]"
                Loaded services file OK.
                Server role: ROLE_STANDALONE
          
          systemctl enable --now smb.service                                            .....kad je sve u redu omogućimo i pokrenemo samba servis
          systemctl status smb.service                                                  .....proverimo status smb servisa
          
          useradd -G users wp_backup                                                    .....kreiramo novog usera pod nazivom wp_backup
          smbpasswd -a wp_backup                                                             .....dajemo useru wp_backup password
          
                 New SMB password: password
                 Retype new SMB password: password                                                .....dva puta potvrdimo da je password password i dobijemo ispis:
                 Added user wp_backup.
          
          usermod -d /srv/backups/wordpress wp_backup                                   .....menjamo user korisnika nad navedenim folderom (imenujemo wp_backup)
          chown wp_backup:wp_backup /srv/backups/wordpress/ -R                          .....menjamo vlasnika foldera wordpress (imenujemo usera wp_backup za 
                                                                                             vlasnika)

          iptables -I INPUT 10 -m state --state NEW -p tcp --dport 139 -j ACCEPT
          iptables -I INPUT 11 -m state --state NEW -p tcp --dport 445 -j ACCEPT        .....ubacujemo još dva lanca da omogućimo pristup kroz portove 139 i 445
          iptables-save > /etc/sysconfig/iptables                                       .....snimimo podešavanja
          systemctl reload iptables.service                                             .....ponovo pokrenemo servis iptables
          
          yum install cifs-utils                                                        .....instaliramo podršku za cifs fajl sistem bez čega je nemoguće mountovati
                                                                                             folder wordpress na mnt
          
          mount -o username=wp_backup //localhost/wordpress /mnt                        .....provera: user wp_backup mountuje wordpress folder na direktorijum mnt
          
          
          Proba sa udaljene mašine (sa GW, WB i DB mašina):
          
          yum install samba                                                             .....instalirati sambu na GW i DB
          systemctl enable --now smb.service                                            .....omogućiti i pokrenuti sambu na GW i DB
          yum install cifs-utils                                                        .....instaliramo podršku za cifs fajl sistem na GW i DB
          setenforce 0                                                                  .....trenutno onemogućimo selinux na GW (na ostalim mašinam je isključen)
          mount -o username=wp_backup //192.168.60.12/wordpress /mnt                    .....zadamo mount komandu i mountujemo wordpress folder sa SHARE mašine u 
                                                                                             /mnt folder GW, WEB i DB mašine
         
          
          
 3.Podešavanje WEB mašine

    3.1   Podešavanje backup-a za /var/www/wp/wordpress
         
          mount -o username=wp_backup //192.168.60.12/wordpress /mnt                   .....mountujemo privremeno /srv/backups/wordpress sa Share mašine na /mnt 
                                                                                            direktorijum
          mkdir /mnt/backup                                                            .....a onda u njemu kreiramo backup direktorijum (u koji će biti snimani
                                                                                            backup fajlovi)

    3.2   Podešavanje automatskog mountovanja prilikom pokretanja sistema

          vim /etc/fstab                                                               .....otvorimo fstab i upišemo:

                //192.168.60.12/wordpress   /mnt   cifs   credentials=/root/.creds,_netdev   0 0     
            

          vim /root/.creds                                                             .....otvorimo novi dokument i upišemo:
          
            user=wp_backup
            password=password    
            
          chmod 700 /root/.creds                                                       .....podesimo permisije da root korisnik ima rwx prava nad fajlom .creds
          
                                                                                            
    3.3   Izrada skripte
   
          vim /root/backup.sh                                                          .....sa vim pravimo shell skriptu u root direktorijumu (/) koja će zadavati 
                                                                                            backup komandu; u nju upisujemo:
            #!/bin/bash
            TIME=`date +%b-%d-%y`                                                           .....timestamp (zapisuje vreme u navedenom formatu u naslov tar.gz fajla)
            FILENAME=backup-wordpress-$TIME.tar.gz                                          .....naziv komprimovanog fajla u koji se smeštaju podaci
            SRCDIR=/var/www/wp/wordpress                                                    .....izvor (folder čiji sadržaj se backu-uje)             
            DESDIR=/mnt/backup                                                              .....destinacija (folder u koji će se snimiti backup fajl  ->
            tar -cPpzf $DESDIR/$FILENAME $SRCDIR                                                 -> /srv/backups/wordpress/backup/)
                                                                                                 
                                                esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                    
          Proba:
          
          bash /root/backup.sh                                                         .....pokrenemo skriptu (skripta napravi tar.gz fajl u /mnt/backup 
                                                                                            direktorijumu a pošto smo se prikopčali na SHARE mašinu i njen /mnt 
                                                                                            direktorijum, backup se učitao na toj mašini)
                              
                                                                                            
    3.4   Podešavanje automatskog izvršenja backup-a                                                                                      
          
          crontab -e                                                                   .....otvaramo cron editor da odredimo vreme izvršenja shell skripte backup.sh
                                                                                            
            # M H DOM M DOW CMND                                                            
            00 20 * * * /bin/bash /root/backup.sh                                           .....upisujemo da se svakog dana u 20 časova komandom bash pokreće 
                                                                                                 skripta backup.sh (koja će indukovati kreiranje backup tar.gz fajla)
                                   
             esc -> shift: -> wq                                                       .....snimimo izmene u fajlu i pojavljuje se potvrda da smo instalirli novi
                                                                                            crontab zadatak: crontab: installing new crontab 
                                                                         
                  
4.Podešavanje DB mašine (mount i backup za database i wordpress database sa mysql servera (SHARE mašina) na DB mašini)

                                              
    4.1   Instaliranje nfs servisa  
   
          yum install nfs-utils                                                        .....instaliramo nfs utils-e
          systemctl enable nfs-server                                                  .....omogućimo nfs server
          
  
    4.2   Mountovanje database foldera (sa SHARE mašine) u koji smeštamo podatke sa DB mašine
   
     *Varijanta 1 (fstab)
     
          vim /etc/fstab                                                               .....otvaramo fstab ako hoćemo da se folder database sa SHARE mašine mountuje   
                                                                                            uvek prilikom pokretanja sistema i upisujemo:
                                                                       
            192.168.60.12:/srv/backups/database     /mnt    nfs     _netdev         0 0         
          
                                              esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                                                                                                      
     *Varijanta 2 (ručno mountovanje)
      
          mount 192.168.60.12:/srv/backups/database /mnt                               .....mountujemo /srv/backups/database sa SHARE mašine na /mnt direktorijum  DB 
                                                                                            mašine, ručno


     
    4.3   Konfigurisanje crond servisa - mount i backup wordpress database 
   
          
     *Varijanta 1 (crontab -e)
       
          crontab -e                                                                   .....otvaramo cron editor da odredimo zadatak da se svakog 
                                                                                            dana u 20.00 časova vrši backup wordpress database iz mašine DB i 
                                                                                            prosleđuje u mašinu Share u folfer /srv/backups/database/:
                                                                                            (mysqldump - naredba kojom se vrši backup podataka)
                                                                                                                                                                    
          00 20 * * * root mysqldump -u root -p P4ssw0rd! wordpress | gzip > /mnt/database_wordpress_`/usr/bin/date +\%b\%d\%Y`.sql.gz      .....upisujemo naredbu
                                                                            
                                                                                            
                                            esc -> shift: -> wq                         .....snimimo izmene u fajlu                                                          
                                                                                            
          systemctl reload crond.service                                                .....ponovo pokrenemo  cron
                                                                                   

             
     *Varijanta 2 (/etc/crontab)                                                            
                                    
          vim /etc/crontab                                                             .....vimom otvorimo konfiguracionu datoteku za crontab i upišemo da se svakog 
                                                                                            dana u 20.00 časova vrši backup wordpress database iz mašine DB i 
                                                                                            prosleđuje u mašinu Share u folder /srv/backups/database/
                                                                                            (mysqldump - naredba kojom se vrši backup podataka):
                            
            00 20 * * * root mysqldump -u root -p P4ssw0rd! wordpress | gzip > /mnt/database_wordpress_`/usr/bin/date +\%b\%d\%Y`.sql.gz      .....upisujemo naredbu
       
          
          systemctl reload crond.service                                               .....ponovo pokrenemo cron



     *Varijanta 3 (shell skripta)
          
          vim /root/wpdbbackup.sh                                                      .....sa vim pravimo shell skriptu u /root/ direktorijumu  koja će zadavati 
                                                                                            backup komandu; u nju upisujemo:  
            #!/bin/bash
            PATH=/bin:/usr/bin:/usr/local/bin
            TODAY=`date +%b-%d-%Y` 
            mysqldump -u root -pP4ssw0rd! wordpress | gzip > /mnt/database_wordpress_$TODAY.sql.gz
            
                                           esc -> shift: -> wq                         .....snimimo izmene u fajlu

          crontab -e                                                                   .....otvaramo cron editor da odredimo vreme izvršenja shell skripte backup.sh
                                                                                            
            # M H DOM M DOW CMND                                                            
            00 20 * * * /bin/bash /root/wpdbbackup.sh    
            
          systemctl reload crond.service                                               .....ponovo pokrenemo cron
                  
          Proba:
          
            bash /root/wpdbbackup.sh                                                   .....pokrenemo skriptu (skripta napravi tar.sqlgz fajl u /srv/backups/        
                                                                                            database/ direktorijumu SHARE mašine)
                                           
                  
                  
