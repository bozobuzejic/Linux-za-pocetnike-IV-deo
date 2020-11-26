POGLAVLJE 4:


*CONFIGURE SERVER   (konfiguracija ssh servisa, apache2 servisa, nfs servera i email servera)

     sudo hostnamectl set-hostname server                                                                .....menjamo ime hosta 
     password: password


     sudo vim /etc/hosts                                                                                 .....podešavanje hosts fajla
        127.0.1.1     server.bozo.com server                                                             .....umesto starog, upisujemo: serverbozo.com server
                                                                                                         .....dopisujemo navedene hostove          
        192.168.2.2   router
        192.168.1.11  client
        ...................................     - >    kasnije možemo dodavati još mašina u mrežu 
        192.168.1.102 guest                                                           
                                                     esc -> shift: -> wq    -  snimimo fajl                                                     
                                                     
     sudo vim /etc/network/interfaces                                                                     .....konfigurišemo network interfaces 
       (# The primary network interfaces
        auto enp0s3
        iface enp0s3 inet dhcp                                                                            .....ispod ove linije dodati po potrebi (ako ne prepoznaje 
                                                                                                               google host):
        dns-nameservers 8.8.8.8 8.8.4.4                                                                        .....liniju dns sa google ip adresama (8.8.8.8 i 8.8.4.4))
    
        # The internal interface on netb                                                                       .....dodajemo ceo blok
        auto enp0s8
        iface enp0s8 inet static
            address 192.168.2.22
            netmask 255.255.255.0
            network 192.168.2.0
            broadcast 192.168.2.255
                                                    esc -> shift: -> wq    -  snimimo fajl
        
Instaliranje sshd servisa:  

    sudo apt-get install openssh-server                                                                  .....instaliramo sshd servis
    sudo systemctl enable sshd.service                                                                   .....omogućimo servis
    sudo systemctl start sshd.service                                                                    .....startujemo sshd

        
    sudo vim /etc/network/interfaces                                                                     .....dodatno konfigurišemo network interfaces        
            post-up route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.2.2 dev enp0s8
            pre-down route del -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.2.2 dev enp0s8
                                                    esc -> shift: -> wq    -  snimimo fajl 
                                                    
        
    sudo systemctl restart networking.service                                                            .....restartujemo  networking servis


Podešavnje UFW (firewall):

    sudo ufw enable
    sudo ufw allow ssh                                                                                .....omogućimo ssh saobraćaj  
    sudo ufw allow http                                                                               .....omogućimo http saobraćaj
    sudo ufw allow https                                                                              .....omogućimo https saobraćaj
    ili
    sudo ufw allow 'OpenSSH'
    sudo ufw allow 'Apache Full'

Izrada ključeva:

    ssh-keygen                                                                                           .....na server mašini generišemo ključ
    ssh-copy-id -i client                                                                                .....sa server mašine pošaljemo authorized_key na client mašinu
    ssh-copy-id -i router                                                                                .....sa server mašine   pošaljemo authorized_key na router mašinu
    ...................................     - >    kasnije možemo dodavati još mašina u mrežu
    ssh-copy-id -i guest

Instaliranje apache2 servera:

    sudo apt update
    sudo apt install apache2                                           
    service apache2 enable                                                                               .....instaliramo, omogućimo i   pokrenemo apache2 (http) server

    vim /etc/apache2/apache2.conf                                                                        .....otvorimo apache konfiguracioni fajl i na kraj fajla dodamo  
                                                                                                              red:
        ServerName localhost
                                                    esc -> shift: -> wq    -  snimimo fajl
    service apache2 reload                                                                               .....ponovo učitamo apache2  (http) server


Instaliranje web browser-a:

    sudo apt install elinks                                                                              .....instaliramo elinux: web browser za mašine bez grafičkog 
                                                                                                              okruženja
    (apt install lynx)


    sudo reboot                                                                                          .....restartujemo mašinu

Provera:

                   Na routeru:  ping 192.168.1.11     Na clientu:  ping 192.168.1.1      Na serveru:  ping 192.168.2.2         
                                ping 192.168.2.22                  ping 192.168.2.22                  ping 192.168.1.11                      
                                ping google.com                    ping google.com                    ping google.com                              
                                ssh client                         ssh router                         ssh router
                                ssh server                         ssh server                         ssh client
                                elinks http://server               elinks http://server               elinks http://localhost 
                               (elinks http://192.168.2.22)       (elinks http://192.168.2.22)       (elinks http://192.168.2.22)
                                                                                                     (elinks http://server)
                                                                                                     (elinks http://10.0.2.15)
                                                                                                     
                                                                                                     
POGLAVLJE 4.1.

Apache2

4.1.1.

Postavljanje virtualnog hosta bez bezbednosnog sertifikata na SERVER:

    sudo mkdir /var/www/virtdomen.com                                                     .....kreiramo novi folder koji navodimo u konfiguracionom fajlu za  
                                                                                               virtdomen.com (DocumentRoot /var/www/virtdomen.com)
    sudo chown -R $USER:$USER /var/www/virtdomen.com                                      .....dodeljujemo vlasništvo nad folderom     useru mašine SERVER                                                     
    sudo chmod -R 755 /var/www/virtdomen.com                                              .....dajemo navedene permisije (755) nad folderom


    sudo vim /var/www/virtdomen.com/index.html                                            .....u kreiranom folderu virtdomen.com pravimo novi fajl index.html i 
                                                                                               upisujemo:
        <html>
        <head>
            <title>Welcome to virtdomen.com!</title>
        </head>
        <body>
            <h1>Success!  The virtdomen.com virtual host is working!</h1>
       </body>
       </html>
                                                   esc -> shift: -> wq    -  snimimo fajl

4.1.2. 

    sudo vim /etc/apache2/sites-available/virtdomen.com.conf                              .....kreiramo novi konfiguracioni fajl sa datim podešavanjima 
        <VirtualHost *:443>                                                                    (koji će važiti umesto /etc/apache2/sites-available/000-default.conf)  
             ServerAdmin webmaster@virtdomen.com
             ServerName virtdomen.com
             ServerAlias www.virtdomen.com
             DocumentRoot /var/www/virtdomen.com
             ErrorLog ${APACHE_LOG_DIR}/error.log
             CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
                                                  esc -> shift: -> wq    -  snimimo fajl
                                                  
                                                  
4.1.3.

    cd /etc/apache2/

    sudo a2ensite virtdomen.com.conf                                                      .....omogućimo sajt koji smo konfigurisali fajlom virtdomen.com.conf
    sudo a2dissite 000-default.conf                                                       .....onemogućimo default konfiguracioni fajl  (000-default.conf)
    ls sites-enabled/
    sudo apache2ctl configtest                                                            .....proverimo sintaksu konfiguracionog fajla

    sudo systemctl restart apache2                                                        .....restartujemo apache servis



Podešavanje - dodavanje naziva hosta na SERVER, ROUTER I CLIENT mašinama:

    sudo vim /etc/hosts                                                                   .....otvorimo fajl sa ip adresama i hostovima (na svim mašinama) i upišemo:
        192.168.2.22    server  virtdomen.com       
                                                 esc -> shift: -> wq    -  snimimo fajl
                                                 
                                                 
                                                 
    Provera:         
    
        Na routeru:  elinks http://virtdomen.com     
        Na clientu:  elinks http://virtdomen.com     
                   

                   
POGLAVLJE 4.2.            


4.2.1. 

Konfiguracija virtuelnog hosta sa sigurnosnim sertifikatom


Varijanta 1.


Kreiranje web stranice sa sertifikatom: 

4.2.1.1.

    Primeniti 4.1.1.                                                                   .....postupak primenjujemo na već konfigurisanom   virtuelnom hostu
    Preskočiti 4.1.2.


    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt  
     Output
          Country Name (2 letter code) [AU]:RS
          State or Province Name (full name) [Some-State]:Belgrade
          Locality Name (eg, city) []:Neki grad
          Organization Name (eg, company) [Internet Widgits Pty Ltd]:Neka firma  
          Organizational Unit Name (eg, section) []:Deo firme               
          Common Name (e.g. server FQDN or YOUR name) []:www.virtdomen.com
          Email Address []:webmaster@virtdomen.com

      
4.2.1.2.

    sudo vim /etc/apache2/conf-available/ssl-params.conf
         SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
         SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
         SSLHonorCipherOrder On
         # Disable preloading HSTS for now.  You can use the commented out header line that includes
         # the "preload" directive if you understand the implications.
         # Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
         Header always set X-Frame-Options DENY
         Header always set X-Content-Type-Options nosniff
         # Requires Apache >= 2.4
         SSLCompression off
         SSLUseStapling on
         SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
         # Requires Apache >= 2.4.11
         SSLSessionTickets Off
                                                              esc -> shift: -> wq    -  snimimo fajl


4.2.1.3.

    sudo vim /etc/apache2/sites-available/default-ssl.conf
         ServerAdmin webmaster@virtdomen.com                 -> umesto localhost 
         ServerName www.virtdomen.com                        -> ubacti liniju
         DocumentRoot /var/www/virtdomen.com                 -> promeniti
         .........................
         #SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
         #SSLCertificateKeyFile   /etc/ssl/private/ssl-cert-snakeoil.key    -> zakomentarisati linije
         SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
         SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key       -> dodamo dve linije
                                                             esc -> shift: -> wq    -  snimimo fajl
                                                             
4.2.1.4

Konfiguracija permanentne redirekcije http u https

    sudo vim /etc/apache2/sites-available/000-default.conf
         <VirtualHost *:80>
            . . .
            Redirect "/" "https://virtdomen.com/"            -> ubaciti liniju                                
            . . .
         </VirtualHost>
                                                              esc -> shift: -> wq    -  snimimo fajl

4.2.1.5

    cd /etc/apache2/   

    sudo a2enmod ssl
    sudo a2enmod headers                                                             
 
    sudo a2ensite default-ssl
    sudo a2ensite 000-default.conf 

    sudo a2enconf ssl-params.conf                                                           .....komandama a2en.... omogućimo navedene fajlove (enable)

    sudo apache2ctl configtest

    sudo systemctl restart apache2


    Provera: 
                   Na routeru:  elinks https://virtdomen.com               Na routeru:  elinks http://virtdomen.com
                   Na clientu:  elinks https://virtdomen.com               Na clientu:  elinks http://virtdomen.com 
                  
 
 

4.2.2.                    

Varijanta 2.


    Primeniti 4.1.1.                                                                   .....postavimo virtuelni host bez sertifikata
    Primeniti 4.1.2.                                                                   .....konfigurišemo host bez sertifikata


4.2.2.1.

Kreiranje web stranice sa sertifikatom: 

    sudo mkdir /var/www/nekidom.rs                                                     .....kreiramo novi folder koji navodimo u    konfiguracionom fajlu za nekidom.rs
                                                                                            (DocumentRoot /var/www/nekidom.rs)
    sudo chown -R $USER:$USER /var/www/nekidom.rs                                      .....dodeljujemo vlasništvo nad folderom useru mašine SERVER                                                     
    sudo chmod -R 755 /var/www/nekidom.rs                                              .....dajemo navedene permisije (755) nad folderom


    sudo vim /var/www/nekidom.rs/index.html                                            .....u kreiranom folderu virtdomen.com pravimo novi fajl index.html i upisujemo:
        <html>
        <head>
            <title>PROBA PROBA!</title>
        </head>
        <body>
            <h1>Dobro dosli u PROBU nekidom.rs</h1>
        </body>
        </html>
                                                   esc -> shift: -> wq    -  snimimo fajl

4.2.2.3.

    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -pkeyopt rsa_keygen_pubexp:65537 -out cakey.pem          -> enter
ls -l cakey.pem
    openssl req -new -x509 -key cakey.pem -out cacert.pem -days 1095

     Output
          Country Name (2 letter code) [AU]:RS
          State or Province Name (full name) [Some-State]:Belgrade
          Locality Name (eg, city) []:Obrenovac
          Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.  ->napisati nesto
          Organizational Unit Name (eg, section) []:Ministry of Water Slides               ->napisati nesto
          Common Name (e.g. server FQDN or YOUR name) []:www.nekidom.rs
          Email Address []:webmaster@nekidom.rs

      
4.2.2.4.

    cd
    mkdir demoCA
    mkdir demoCA/certs
    mkdir demoCA/crl
    mkdir demoCA/newcerts
    mkdir demoCA/private
    touch demoCA/index.txt
    echo 02 > demoCA/serial

    mv cacert.pem demoCA/
    mv cakey.pem demoCA/private/


4.2.2.5.

    sudo vim /usr/lib/ssl/openssl.cnf
         # For the CA policy
           [ policy_match ]
           countryName             = match
           stateOrProvinceName     = optional
           organizationName        = optional
           organizationalUnitName  = optional
           commonName              = supplied
           emailAddress            = optional
   
                                                  esc -> shift: -> wq    -  snimimo fajl
                                                                                                       
4.2.2.6.                                                    

    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -pkeyopt rsa_keygen_pubexp:65537 -out privkey-www.nekidom.rs.pem

    openssl req -new -key privkey-www.nekidom.rs.pem -out certreq-www.nekidom.rs.csr

      Output
          Country Name (2 letter code) [AU]:RS
          State or Province Name (full name) [Some-State]:Belgrade                         -> ne mora
          Locality Name (eg, city) []:Obrenovac                                            -> ne mora
          Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.  -> napisati nesto ne mora
          Organizational Unit Name (eg, section) []:Ministry of Water Slides               -> napisati nesto
          Common Name (e.g. server FQDN or YOUR name) []:www.nekidom.rs                    -> mora
          Email Address []:webmaster@www.nekidom.rs                                        -> ne mora
      
    openssl ca -in certreq-www.nekidom.rs.csr -out cert-www.nekidom.rs.pem
    yes
    yes

    cp demoCA/cacert.pem cert-ourca.crt

    openssl verify -CAfile cert-ourca.crt cert-www.nekidom.rs.pem
    sudo cp cert-www.nekidom.rs.pem /etc/ssl/certs/
    sudo cp cert-ourca.crt /etc/ssl/certs/
    sudo cp privkey-www.nekidom.rs.pem /etc/ssl/private/

    cd /etc/apache2/
    cd sites-available/

    sudo cp default-ssl.conf nekidom.rs.conf


4.2.2.7.

    sudo vim nekidom.rs.conf
       ServerAdmin webmaster@nekidom.rs                 -> umesto localhost 
          ServerName www.nekidom.rs:443                 -> ubacti liniju
       DocumentRoot /var/www/nekidom.rs                -> promeniti
       .........................
       #SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
       #SSLCertificateKeyFile   /etc/ssl/private/ssl-cert-snakeoil.key                     -> zakomentarisati linije
       SSLCertificateFile       /etc/ssl/certs/cert-www.nekidom.rs.pem
       SSLCertificateKeyFile    /etc/ssl/private/privkey-www.nekidom.rs.pem  
       SSLCACertificateFile     /etc/ssl/certs/cert-ourca.crt                              -> dodati tri linije 
                                                  esc -> shift: -> wq    -  snimimo fajl
      
4.2.2.8.  

    cd ..
    (cd /etc/apache2/)
    sudo a2enmod ssl
    ls sites-enabled/
    sudo a2ensite nekidom.rs                     
    ls sites-enabled/
    sudo systemctl reload apache2

  
4.2.2.9.                   

Podešavanje klijenta

Na Clientu

    scp 192.168.2.22:/home/bozoubuntu/cert-ourca.crt .
    ls
    sudo mkdir /usr/share/ca-certificates/extra
    sudo cp cert-ourca.crt /usr/share/ca-certificates/extra/
    sudo dpkg-reconfigure ca-certificates
    yes
    čekirati sa space dugmetom extra/cert-ourca.crt
    tab -> OK
    
    
   POGLAVLJE 4.3. 
NFS

4.3.1.

Konfiguracija NFS servera:

    sudo apt install aptitude
    sudo aptitude update && aptitude install nfs-kernel-server

    sudo vim /etc/default/nfs-common
         NEED_IDMAPD=YES
                                                                 esc -> shift: -> wq    -  snimimo fajl

    (sudo vim /etc/hosts
         127.0.1.1       server.bozo.com server                                                           .....ako nije urađeno, dopisti domain name - server.bozo.com)
                                                                 esc -> shift: -> wq    -  snimimo fajl 
                                                                  
    sudo vim /etc/idmapd.conf                                                                             .....upisujemo domain name servera
         Domain = server.bozo.com
                                                                 esc -> shift: -> wq    -  snimimo fajl
     
a) Konfiguracija za deljenje sa root privilegijama

     sudo mkdir /srv/nekifolder1/
     sudo mkdir /srv/nekifolder1/nekifolder2                                                               .....kreiramo folder sa pravima root usera čiji će se sadržaj 
                                                                                                                mountovati na klijentima
           
    (sudo chown nobody:nogroup /srv/nekifolder1/nekifolder2
    sudo chmod 777 /srv/nekifolder1/nekifolder2                                                            .....alternativno, menjamo vlasništvo i permisije nad folderom 
                                                                                                                koji će se mountovati na klijenta)
                                                                                                           
      sudo vim /srv/nekifolder1/nekifolder2/probadoc.txt                                                    .....kreiramo probni fajl u nekifolder2 koji če se mountovati
        
      Napišemo nešto u dokument kog formiramo za probu 
                                                                 esc -> shift: -> wq    -  snimimo fajl
      

b) Konfiguracija za deljenje bez root privilegia
     
    mkdir /home/bozoubuntu/share                                                                          .....kreiramo folder za deljenje bez root privilegija, kao user

    vim share/userproba.txt                                                                               .....kreiramo probni txt fajl
    Napišemo nešto u dokument kog formiramo za probu
                                                                 esc -> shift: -> wq    -  snimimo fajl
    
    sudo vim /etc/exports                                                                                 .....u folder exports upisujemo koji folderi će se mountovati
         /srv/nekifolder1/nekifolder2  192.168.1.0/24(rw,sync,no_root_squash)                                  i na koje klijente sa dodatim pravilima u zagradama
         /srv/nekifolder1/nekifolder2  192.168.2.0/24(rw,sync,no_root_squash)
         /srv/nekifolder1/nekifolder2  192.168.0.1/24(rw,sync,no_root_squash)
         #
         /home/bozoubuntu/share  192.168.1.0/24(rw,sync,no_subtree_check)
         /home/bozoubuntu/share  192.168.2.0/24(rw,sync,no_subtree_check)
         /home/bozoubuntu/share  192.168.0.1/24(rw,sync,no_subtree_check)
                                                                 esc -> shift: -> wq    -  snimimo fajl

     
    sudo exportfs -a                                                                                      .....primenimo podešavanje

    showmount -e 192.168.2.22                                                                             .....na serveru proveriti šta je predviđeno za mountovanje

    sudo systemctl restart nfs-kernel-server                                                              .....restartujemo nfs servis
     
     
     
4.3.2.

Komande na klijentu 

    sudo apt install nfs-common                                                                                          .....instaliramo nfs na klijenta
    (apt-get install nfs-common portmap)


*Mount i umount u tekućoj sesiji (ručno mountovanje)

a) Konfiguracija za mountovanje foldera sa root privilegijama

    sudo mount 192.168.2.22:/srv/nekifolder1/nekifolder2/ /mnt                                                           .....naredba za mountovanje sadržaja nekifolder2 
                                                                                                                              na /mnt 
    (sudo touch /mnt/4.test.txt                                                                                          .....proba: dodajemo 4.test.txt fajl u folder 
                                                                                                                              gde se vrši mountovanje (fajl pripada 
                                                                                                                              root-u))                                                                                                                          
    sudo umount 192.168.2.22:/srv/nekifolder1/nekifolder2/ /mnt                                                          .....umount
    sudo umount /mnt

    sudo mount -t nfs4 -o proto=tcp,port=2049 server.bozo.com:/srv/nekifolder1/nekifolder2 /home/bozoubuntu/nfssarutera  .....napraviti neki folder (nfssarutera) na 
                                                                                                                              klijentu u koji će se mountovati 
    (sudo mount -t nfs 192.168.2.22:/srv/nekifolder1/nekifolder2 /mnt/nfsshare)                                               nekifolder2 sa servera
                                                                                                                              ako ne   mountujemo u /mnt                                                                                                                          
    sudo umount server.bozo.com:/srv/nekifolder1/nekifolder2/ /home/bozoubuntu/nfssarutera                               .....umount
    sudo umount /mnt



b) Konfiguracija za mountovanje foldera bez root privilegija

    sudo mkdir /home/bozoubuntu/nfsshare

    sudo mount 192.168.2.22:/home/bozoubuntu/share /home/bozoubuntu/nfsshare

    (touch /home/bozoubuntu/nfsshare/4.test.txt                                                     .....proba: dodajemo 4.test.txt fajl u folder gde se vrši mountovanje
                                                                                                         isti fajl se pojavljuje na serveru bez ograničenja)
    sudo umount /home/bozoubuntu/nfsshare



*Mount i umount prilikom dizanja sistema (fstab) -> trajno mountovanje


    sudo vim /etc/fstab
        192.168.2.22:/srv/nekifolder1/nekifolder2     /mnt    nfs     _netdev         0 0
        192.168.2.22:/home/bozoubuntu/share           /home/bozoubuntu/nfsshare   nfs     _netdev         0 0
                                                                 esc -> shift: -> wq    -  snimimo fajl
 
   
*Autofs 

(alatka za mountovanje tačke montiranja domaćina na mount point klijenta - sa ograničenim trajanjem) 
 
 
    sudo apt-get install cifs-utils autofs                                                                 .....automatizacija mountovanja (bez izmene fstab fajla)

    sudo mkdir /nfsshare2/                                                                                 .....u direktorijumu / kreiramo mount point na klijentu
                                                                                                                za autofs mountovanje
    sudo vim /etc/auto.master                                                                              .....otvoriti konfiguracioni fajl
        #/misc  /etc/auto.misc                                                                                  .....zakomentarisati liniju
         /nfsshare2 /etc/auto.nfs --timeout=120                                                                 .....dopisati liniju kojom se određuje mount point na 
                                                                                                                     klijentu /nfsshare2
                                                                 esc -> shift: -> wq    -  snimimo fajl                                                                      
    cp /etc/auto.misc /etc/auto.nfs                                                                        .....iskopiramo auto.misc u   auto.nfs da zadržimo model 
                                                                                                                podešavanja

    sudo vim /etc/auto.nfs                                                                                 .....otvorimo fajl za konfiguraciju
         share        -fstype=nfs     192.168.2.22:/home/bozoubuntu/share                                       .....dopišemo liniju na kraju dokumenta (sadržaj foldera 
                                                                                                                     share na serveru će se mountovati u folder
                                                                                                                     /nfsshare2/share na klijentu
                                                                 esc -> shift: -> wq    -  snimimo fajl                                              
    systemctl restart autofs.service                                                                       .....restartujemo autofs servis                                       
     

    cd /nfsshare2/share/                                                                                   .....premestimo se u mount point folder -> automatski se 
                                                                                                                mountuje /home/bozoubuntu/  share sa servera na       
                                                                                                                /nfsshare2/share/ klijenta
                                                                                                           
                                                                                                                                                                                                                      



POGLAVLJE 4.4. (konfiguracija ROUTER mašine) - DNS, DHCP

   
   Važno:
   
    *  VirtualBox -> File -> Host Network Manager -> vboxnet2 -> DHCP server = odčekirati -> Aplly -> Close   .....odčekiramo dhcp server na VB jer ne sme da rade 
                                                                                                                   dva u isto vreme na istim mašinama

    1. Instaliranje i podešavanje DNS servisa na GW mašini
   
       1.1 Instaliranje servisa
    
           apt-get installl bind9 bind9utils                                           .....instaliramo bind na ubuntu
           systemctl enable bind9                                                      .....omogućimo servis i pokrenemo ga
          
          
       1.2 Podešavanje firewall-a
      
          
           sudo ufw allow 53                                                           .....dodamo pravila da router mašina sluša na portu 53
           sudo ufw service restart                                                    .....restartujemo firewall
          
          
       1.3 Podešavanje konfiguracionog fajla (sekcija options)   
      
           vim /etc/bind/named.conf                                                    .....otvorimo konfiguracioni fajl za bind i dodamo ceo blok
             options {
                      #listen-on port 53 { 127.0.0.1; };                                    .....zakomentarišemo liniju
                      listen-on port 53 { any; };                                      .....dodamo liniju
                      #listen-on-v6 port 53 { ::1; };                                       .....zakomentarišemo liniju
                      listen-on-v6 port 53 { none; };                                  .....dodamo liniju
                      directory       "/etc/bind";                                                                  /var/cache/bind
                      dump-file       "/etc/bind/cache_dump.db";                                             /var/cache/bind/ cache_dump.db    .....putanja u ubuntu-u   
                      statistics-file "/etc/bind/named_stats.txt";                                             .....
                      memstatistics-file "/etc/bind/named_mem_stats.txt";
                      recursing-file  "/etc/bind/named.recursing";
                      secroots-file   "/etc/bind/named.secroots";
                      #allow-query     { localhost; };                                     .....zakomentarišemo liniju
                      allow-query     { localhost; 192.168.1.0/24; };                 .....dodamo liniju
                      #dodato:                                                             .....dodamo i zakomentarišemo liniju
                      forwarders { 8.8.8.8; 8.8.4.4; };
                      forward only;                                                    .....dodamo liniju
                      };
                                        esc -> shift: -> wq                            .....snimimo izmene u fajlu   
                                        
           named-checkconf                                                             .....izvršimo proveru sintakse          
           systemctl start bind9                                                       .....restartujemo servis    
 
          
       1.4 Podešavanje konfiguracionog fajla - podešavanje zone  
      
           vim /etc/bind/named.conf.default-zones                                      .....otvorimo konfiguracioni fajl za zone u bind-u
             zone "lab." IN {
                 type master;
                 file "lab";               ili "/etc/bind/lab"
                 allow-update { none; };
             };                                                                        .....unesemo promene u sekciju zone
                                       esc -> shift: -> wq                             .....snimimo izmene u fajlu  
           named-checkconf                                                             .....izvršimo proveru sintakse                                    
            
            
       1.5 Kreiranje fajla koji definiše detalje zone
                     
           cd /etc/bind
           cp named.empty lab                                                           .....preimenujemo fajl named.empty u lab (nema u ubuntu-u)
           ili 
           vim lab                                                                      .....kreiramo novi fajl
             $TTL 3H
             $ORIGIN lab.
             lab.       IN SOA  router.lab. admin.lab. (
                                                    1       ; serial
                                                    1D      ; refresh
                                                    1H      ; retry
                                                    1W      ; expire
                                                    3H )    ; minimum
             lab.        NS      router.lab.
             router      A       192.168.1.2
             client      A       192.168.1.11
             server      A       192.168.1.12
             ...................................     - >    kasnije možemo dodavati još mašina u mrežu
             guest       A       192.168.1.13
                                       esc -> shift: -> wq                          .....snimimo izmene u fajlu
                                       
                                       
           chown :bind lab                                                          .....promenimo grupu korisnika fajla lab iz root u bind
                                                                       
                                                                                                   
           named-checkzone lab lab                                                  .....proverimo da li je sve upisano kako treba i dobijemo:
                 zone lab/IN: loaded serial 1
                 OK
                 
           systemctl restart bind9                                                  .....restartujemo servis   
                 
          
      1.6 Podešavanje hosts fajla (na router, client i server mašinama)
     
          vim /etc/hosts                                                            .....otvorimo hosts fajl i dodamo:                                                                                    
            192.168.1.2  router.lab
            192.168.1.11 client.lab
            192.168.1.12 server.lab
            ...................................     - >    kasnije možemo dodavati još mašina u mrežu
            192.168.1.13 guest.lab
                                       esc -> shift: -> wq                          .....snimimo izmene u fajlu
                                       
                                                     
     2.Instaliranje i podešavanje DHCP servisa na router mašini
   
       2.1 Instaliranje servisa
      
           apt-get install isc-dhcp-server.service                                   .....instaliramo dhcp servis
          
       2.2 Podešavanje konfiguracionog fajla      
          
           vim /etc/dhcp/dhcpd.conf                                                  .....otvorimo konfiguracioni fajl za dhcpd.service
             option domain-name-servers 192.168.1.2;
             option domain-search "lab";
             default-lease-time 14400;                                                    .....lease time: 4 sata
             max-lease-time 14400;
             ddns-update-style none;
             authoritative;
             log-facility local4;
             subnet 192.168.1.0 netmask 255.255.255.0 {                                   .....definišemo ip adrese cele mreže
             range 192.168.1.101 192.168.1.120;                                           .....definišemo raspon mogućih adresa koje dhcp može da dodeljuje (101-200)
             option routers 192.168.1.2;
             }
             host router {                                                                        
                hardware ethernet 08:00:27:31:27:c8;                                      .....upisujemo mac adresu adaptera (enp0s3)
                fixed-address 192.168.1.2;                                                .....postavljamo statičku ip adresu
             }
             host client {
                hardware ethernet 08:00:27:db:36:12;                                      .....upisujemo mac adresu adaptera (enp0s3)
                fixed-address 192.168.1.11;                                               .....postavljamo statičku ip adresu
             }
             host server {
                hardware ethernet 08:00:27:ab:dd:be;                                      .....upisujemo mac adresu adaptera (enp0s3)
                fixed-address 192.168.1.12;                                               .....postavljamo statičku ip adresu
             } 
             ...................................     - >    kasnije možemo dodavati još mašina u mrežu
             host guest {
                hardware ethernet 08:00:27:f9:71:4b;
                fixed-address 192.168.1.13;
             }
                                       esc -> shift: -> wq                           .....snimimo izmene u fajlu  
                                       
           dhcpd -t -cf /etc/dhcp/dhcpd.conf                                         .....proverimo da li je sve upisano kako treba i dobijemo ispis:
         
            Internet Systems Consortium DHCP Server 4.2.5
            Copyright 2004-2013 Internet Systems Consortium.
            All rights reserved.
            For info, please visit https://www.isc.org/software/dhcp/
            Not searching LDAP since ldap-server, ldap-port and ldap-base-dn were not specified in the config file
            
           systemctl enable --now dhcpd.service                                           .....obavezno restartujemo servis 
           sudo systemctl enable isc-dhcp-server.service                                       na ubuntu
          
       2.2 Provera ispravnosti rada:
           (uraditi na svim mašinama)
          
           ip a
           sudo dhclient -r                                                         .....isključimo sve stare klijentske procese
           ip a
           sudo dhclient                                                            .....ponovo uključimo dhklijente
           ip a
          
           ip route                                                                 .....proverimo da li je rutu dodelio dhcp
          
          Primer provere:
          
           ip route get 8.8.8.8                                                     .....na client-u ovom komandom tražimo da pokaže rutu do IP adrese google servera
                  8.8.8.8 via 192.168.1.2 dev enp0s3  src 192.168.1.11                   .....ispravan odgovor: ruta vodi preko routera do clienta 
                 
           cat /var/lib/dhcp/dhclient.leases                                        .....na mrežnom adapteru enp0s3  mašina (client i server) pojavljuju se 
                                                                                         novododeljene statičke ip adrese 
                                                    
           Na client i server mašinama (na routeru ostaje manual):                                                                              
                                                                                        
           nmtui -> Edit a connection -> enp0s3 -> Edit -> IPv4 CONFIGURATION -> automatic -> Ok -> back -> Activate a connection -> enp0s3 -> Deactivate ->
           Activate-> Back -> Ok
         

         

POGLAVLJE 4.5.

Podešavanje e-mail servisa (konfiguracija ROUTER mašine)


    aptitude update && aptitude postfix dovecot-imapd dovecot-pop3d


4.5.1   

Postfix kao relay host za gmail server (slanje preko autorizovanog relay hosta)
 
    systemctl status postfix.service                                                   .....proverimo status postfix servisa
 
    vim /etc/postfix/main.cf
       myhostname = router.lab                                                                   .....dodamo liniju
       mydomain = lab                                                                       .....dodamo liniju
       myorigin = $mydomain                                                                 .....otkomentarišemo liniju 
       alias_maps = hash:/etc/aliases
       alias_database = hash:/etc/aliases
       inet_interfaces = all                                                                .....otkomentarišemo liniju
      #inet_interfaces = localhost                                                               .....zakomentarišemo liniju
       inet_protocols = ipv4                                                                .....na kraj linije umesto all dodajemo ipv4
      #mydestination = $myhostname, localhost.$mydomain, localhost                               .....zakomentarišemo liniju
       mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain               .....otkomentarišemo liniju
       mynetworks = 192.168.1.0/24, 127.0.0.0/8                                             .....otkomentarišemo i upišemo umesto fabrički date mreže
       relayhost = [smtp.gmail.com]:587                                                     .....podešavanje kad idemo preko gmail servisa
       smtp_use_tls = yes
       smtp_sasl_auth_enable = yes
       smtp_sasl_security_options =
       smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
       smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt                                      .....na ubuntu: /etc/ssl/certs/ca-certificates.crt
       smtpd_tls_security_level = may                                                       .....dodajemo na kraj dokumenta
       mailbox_size_limit = 0
       recipient_delimiter = +
       home_mailbox = Maildir/
       ..............................                                                            .....dodato posle konfigurisanja dovecot-a
       smtpd_sasl_type = dovecot
       smtpd_sasl_path = private/auth
       broken_sasl_auth_clients = yes
       smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
       
                                                esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                
 
    *Gmail podržava "App Password": koristiti Google’s 2-Step verification umesto gmail passwod-a. Generisati "App Password": https://myaccount.google.com/
     apppasswords -> kliknuti “Select App” → “Other”, upisati “postfix” i pritisnuti “Generate” -> dobija se password od 16 karaktera koji se upiše u konfiguracioni 
     fajl.  
     
     vim /etc/postfix/sasl_passwd   
         [smtp.gmail.com]:587    nekiaccount@gmail.com:dfrnijgzwejqqwlo                 .....podesimo gmail server da radi sa našim nalogom i dobijenim passwordom od 
                                                                                              16 karaktera
                                                 esc -> shift: -> wq                         .....snimimo izmene u fajlu   
                                                 
     postmap /etc/postfix/sasl_passwd                                                   .....heširamo fajl sa passwordom: ovo kreira /  etc/postfix/sasl_passwd.db 
                                                                                             (binarni fajl)
     chmod 600 /etc/postfix/sasl_passwd                                                 .....promenimo permisije sasl_passwd fajlu da mu se ograniči pristup
 
     systemctl restart postfix.service                                                  .....restarujemo postfix servis
    
    
     Firewall za email servis

     sudo ufw allow 25                                                                  .....dodamo dozvolu za port 25
     sudo ufw restart
    
    
    PROBA:   
    mail -s "Test subject" nekinalog@hotmail.com                                  .....sa gmail naloga šaljemo mail nekom sa hotmail nalogom
            TEKST PORUKE
            ctrl D                                                                     .....poruka poslata
    

    4.5.2   Konfiguracija dovecot-a (IMAP i POP3 servera)


    apt -y install dovecot-core dovecot-pop3d dovecot-imapd

    vim /etc/dovecot/dovecot.conf
        protocols = imap pop3
   
    netstat -tupan | grep dovecot    - >  pokazuje da treba da se otvore portovi 110(POP3), 143(IMAP), 993(IMAPS) i 995(POPS3) 
   
    vim /etc/dovecot/conf.d/10-mail.conf
        mail_location = mbox:~/mail:INBOX=/var/mail/%u
    
    vim /etc/dovecot/conf.d/10-auth.conf
        disable_plaintext_auth = no
   
    openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/mail_server.key -out /etc/ssl/certs/mail_server.crt
                                                                 
    vim /etc/dovecot/conf.d/10-ssl.conf 
        ssl_key = </etc/ssl/private/mail_server.key 
 
    vim /etc/dovecot/conf.d/10-master.conf                                                    .....otkomentarisati
        unix_listener auth-userdb{
        mode = 0666
        user = postfix
        group = postfix
        }
        unix_listener /var/spool/postfix/private/auth {
        mode = 0666
        }

    vim /etc/postfix/main.cf                                                                  .....dodati  na kraj dokumenta 
        smtpd_sasl_type = dovecot
        smtpd_sasl_path = private/auth
        broken_sasl_auth_clients = yes
        smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
   
    vim /etc/dovecot/conf.d/15-mailboxes.conf
        namespace inbox {
       #These mailboxes are widely used and could perhaps be created automatically:
        mailbox Drafts {
           auto = create
           special_use = \Drafts
        }
        mailbox Junk {
           auto = create
           special_use = \Junk
        }
        mailbox Trash {
           auto = create
           special_use = \Trash
        }
        mailbox Sent {
           auto = create
           special_use = \Sent
        }
        mailbox "Sent Messages" {
           auto = create
           special_use = \Sent
        }
                                                    
                                                    

   
   Poglevlje 4.6
   
   Mail server podešavanje (Postfix, Dovecot, MySQL, SpamAssasin)
   
   4.6.1 *Mysql
  
     sudo apt install mysql-server
     Password: P4ssw0rd!
     mysql -u root -p 
     password: P4ssw0rd!
     mysql >mysqladmin -p create servermail                                                     .....Imenujemo bazu podataka `servermail` (možete i bilo koje drugo ime)
   
     mysql> show databases;                                                                      .....Proba
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | servermail         |
    | sys                |
    +--------------------+
    5 rows in set (0.00 sec)
  
     mysql > GRANT SELECT ON servermail.* TO 'routermail'@'127.0.0.1' IDENTIFIED BY 'password';   .....Kreiramo usera: routermail i određujemo mu password: password
     mysql > FLUSH PRIVILEGES;                                                                    .....Aktiviramo privilegije
   
     mysql> SELECT User,Host FROM mysql.user;                                                     .....Proba
    +------------------+-----------+
    | User             | Host      |
    +------------------+-----------+
    | routermail       | 127.0.0.1 |
    | debian-sys-maint | localhost |
    | mysql.session    | localhost |
    | mysql.sys        | localhost |
    | root             | localhost |
    +------------------+-----------+
    5 rows in set (0.00 sec)

     mysql > USE servermail;                                                                 .....Uzimamo kreitranu bazu "servetmail"
   
     mysql > CREATE TABLE `virtual_domains` (                                                .....U bazi `servermail` kreiramo tabelu `virtual_domains` 
           `id`  INT NOT NULL AUTO_INCREMENT,
           `name` VARCHAR(50) NOT NULL,
           PRIMARY KEY (`id`)
           ) ENGINE=InnoDB DEFAULT CHARSET=utf8;                                              .....U bazi `servermail` kreiramo tabelu `virtual_users`
     mysql > CREATE TABLE `virtual_users` (                                                         
           `id` INT NOT NULL AUTO_INCREMENT,
           `domain_id` INT NOT NULL,
           `password` VARCHAR(106) NOT NULL,
           `email` VARCHAR(120) NOT NULL,
           PRIMARY KEY (`id`),
           UNIQUE KEY `email` (`email`),
           FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
           ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     mysql > CREATE TABLE `virtual_aliases` (                                                  .....U bazi `servermail` kreiramo tabelu `virtual_aliases`
           `id` INT NOT NULL AUTO_INCREMENT,
           `domain_id` INT NOT NULL,
           `source` varchar(100) NOT NULL,
           `destination` varchar(100) NOT NULL,
           PRIMARY KEY (`id`),
           FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
           ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
     mysql> show tables;                                                                        .....Proba
    +----------------------+
    | Tables_in_servermail |
    +----------------------+
    | virtual_aliases      |
    | virtual_domains      |
    | virtual_users        |
    +----------------------+
    3 rows in set (0.00 sec)

     mysql > INSERT INTO `servermail`.`virtual_domains`                                        .....U bazi `servermail`, u tabelu `virtual_domains`                                        
             (`id` ,`name`)                                                                    .....dodajemo domene koje želimo:
             VALUES                                                                            .....ovde je dat samo primarni domen 'cyberpunk.rs'
             ('1', 'cyberpunk.rs'),                                                            .....i FQDN 'mail.cyberpunk.rs'
             ('2', 'mail.cyberpunk.rs');
           
     mysql> SELECT * FROM virtual_domains;
    +----+-------------------+
    | id | name              |
    +----+-------------------+
    |  1 | cyberpunk.rs      |
    |  2 | mail.cyberpunk.rs |
    +----+-------------------+
    2 rows in set (0.00 sec)
   
     mysql > INSERT INTO `servermail`.`virtual_users`                                                            .....U bazi `servermail`, u tabelu `virtual_users`
           (`id`, `domain_id`, `password` , `email`)                                                                  upisujemo adresu e-pošte i lozinke povezane za 
                                                                                                                      svaki domen                                        
           VALUES                                                                                                     (upisati sa svojim specifičnim podacima)
           ('1', '1', ENCRYPT('password1', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'client@cyberpunk.rs'),
           ('2', '1', ENCRYPT('password2', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'server@cyberpunk.rs'),
           ('3', '1', ENCRYPT('password3', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'guest@cyberpunk.rs'),
           ('4', '2', ENCRYPT('password1', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'client@mail.cyberpunk.rs'),
           ('5', '2', ENCRYPT('password2', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'server@mail.cyberpunk.rs'),
           ('6', '2', ENCRYPT('password3', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'guest@mail.cyberpunk.rs');
           
    mysql> SELECT * FROM virtual_users;                                                                                                                       
    .....Proba
    +----+-----------+------------------------------------------------------------------------------------------------------------   +--------------------------+
    | id | domain_id | password                                                                                                   | email                    |
    +----+-----------+------------------------------------------------------------------------------------------------------------   +--------------------------+
    |  1 |         1 | $6$2c696b0ce6e9bfea$jioq1RRi/poFdMH6./YGmiZgrgUJOpLPcLlvrgP6V/RaC6ud4/z6WGczzfMXLCiEmY.wCeOuMvLKU2TwFSnkg. |   client@cyberpunk.rs      |
    |  2 |         1 | $6$b21de4daed186e2a$Gt8htp3yXLLYoliwHqjWswQqvDb23WiiIgd/.RtEaWDIRTiXtBfSB2OZ.HTDOuHd92sUwIf4u72SA9W/ZAe5M1 |  server@cyberpunk.rs      |
    |  3 |         1 | $6$75f448329d59448f$t2cQ.2GSomVZgGNZ/MRqjihcj.u4t4Y0RRwV0XJAdLan5OJi1IVjFVPNLPw.0ptRlAkze1zWsIbohXwDHTeFv/ | guest@cyberpunk.rs       |
    |  4 |         2 | $6$66c6ac368b994408$ka7vh05SABKeZrtkYmo0ZCKClHd5EaEomIuXb3DvjswL7MvdPRhATVAsTXI0BZGP33OAdtB7ewQSAUS4mXckC1 | client@mail.cyberpunk.rs |
    |  5 |         2 | $6$c3a84d3ace081550$IF9Wv5jWnM.hWMm.bv05OUj.YB6L1nd.WPhgBUo0C0NhZ8PZfzK3CnI04TX5q/gRdNinTDSaL2Tzo8x48BDiC1 | server@mail.cyberpunk.rs |
    |  6 |         2 | $6$840c811c53d27ac4$ge6DelRKHh/TsFj67dYcgMAX/51mIbwryzPAxnixDCB/j9uK1FQ2VSI3bO0lz/cqoUWsNtKZKfuSehFPMNY6l. | guest@mail.cyberpunk.rs  |
    +----+-----------+------------------------------------------------------------------------------------------------------------+--------------------------+
    6 rows in set (0.00 sec)

     mysql > INSERT INTO `servermail`.`virtual_aliases`                                          .....U bazi `servermail`, u tabelu `virtual_aliases` 
           (`id`, `domain_id`, `source`, `destination`)                                               upisujemo adresu e-pošte (source)      
           VALUES                                                                                     koju prosleđujemo na drugu adresu e-pošte (destination)
           ('1', '1', 'client2@cyberpunk.rs', 'client@cyberpunk.rs'),
           ('2', '1', 'server2@cyberpunk.rs', 'server@cyberpunk.rs'),
           ('3', '1', 'guest2@cyberpunk.rs', 'guest@cyberpunk.rs'),
           ('4', '2', 'client2@mail.cyberpunk.rs', 'client@mail.cyberpunk.rs'),
           ('5', '2', 'server2@mail.cyberpunk.rs', 'server@mail.cyberpunk.rs'),
           ('6', '2', 'guest2@mail.cyberpunk.rs', 'guest@mail.cyberpunk.rs');
           
     mysql> SELECT * FROM virtual_aliases;                                                       .....Proba
    +----+-----------+---------------------------+--------------------------+
    | id | domain_id | source                    | destination              |
    +----+-----------+---------------------------+--------------------------+
    |  1 |         1 | client2@cyberpunk.rs      | client@cyberpunk.rs      |
    |  2 |         1 | server2@cyberpunk.rs      | server@cyberpunk.rs      |
    |  3 |         1 | guest2@cyberpunk.rs       | guest@cyberpunk.rs       |
    |  4 |         2 | client2@mail.cyberpunk.rs | client@mail.cyberpunk.rs |
    |  5 |         2 | server2@mail.cyberpunk.rs | server@mail.cyberpunk.rs |
    |  6 |         2 | guest2@mail.cyberpunk.rs  | guest@mail.cyberpunk.rs  |
    +----+-----------+---------------------------+--------------------------+
    6 rows in set (0.00 sec)


   4.6.2 *Postfix 
   
     sudo vim /etc/postfix/main.cf 
            # TLS parameters                                                                        .....zakomentarišemo TSL parametre
            #smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
            #smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
            #smtpd_use_tls=yes
            #smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
            #smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache 
            
            #smtpd_tls_cert_file=/etc/ssl/certs/dovecot.pem                                         .....i ove zakomentarišemo, ako ih ima
            #smtpd_tls_key_file=/etc/ssl/private/dovecot.pem
            #smtpd_use_tls=yes
            #smtpd_tls_auth_only = yes
             
            smtpd_sasl_type = dovecot                                                               .....ove upisujemo ako nisu
            smtpd_sasl_path = private/auth
            smtpd_sasl_auth_enable = yes
            smtpd_recipient_restrictions = permit_sasl_authenticated, permit_mynetworks, reject_unauth_destination
            
            #mydestination = example.com, hostname.example.com, localhost.example.com, localhost    .....zakomentarišemo
            mydestination = localhost                                                               .....dodamo
            myhostname = mail.cyberpunk.rs                                                          .....u ovom primeru (model: hostname.example.com)
            mydestination = localhost
            relayhost =
            mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
            mailbox_size_limit = 0
            recipient_delimiter = +
            inet_interfaces = all
            virtual_transport = lmtp:unix:private/dovecot-lmtp
            virtual_mailbox_domains = mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
            virtual_mailbox_maps = mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
            virtual_alias_maps = mysql:/etc/postfix/mysql-virtual-alias-maps.cf
            
     sudo vim /etc/postfix/mysql-virtual-mailbox-domains.cf                                       .....kreiramo fajl koji definiše vezu mysql i postfix
            user = routermail                                                                          user je definisan prilikom kreiranja baze servermail i 
            password = password                                                                        privilegija nad njom i password koji smo tamo dali        
            hosts = 127.0.0.1
            dbname = servermail
            query = SELECT 1 FROM virtual_domains WHERE name='%s'
            
     sudo service postfix restart
   
     postmap -q cyberpunk.rs mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf                  .....ako je izlaz rezultat broj 1, onda je konfiguracija dobra
                                                                                                       virtualni domen iz mysql baze smo povezali sa postfix-om
                                                                                                     
     sudo vim /etc/postfix/mysql-virtual-mailbox-maps.cf                                          .....kreiramo fajl za vezu mysql sa postfix za usere
            user = routermail 
            password = password
            hosts = 127.0.0.1
            dbname = servermail
            query = SELECT 1 FROM virtual_users WHERE email='%s'                                     
            
     sudo service postfix restart
   
     postmap -q client@cyberpunk.rs mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf              .....ako je izlaz rezultat broj 1, onda je konfiguracija dobra           
                                                                                                       virtualne usere iz mysql baze smo povezali sa postfix-om
                                                                                                     
     sudo vim /etc/postfix/mysql-virtual-alias-maps.cf                                            .....kreiramo fajl za vezu mysql sa postfix za aliase
            user = routermail
            password = password
            hosts = 127.0.0.1
            dbname = servermail
            query = SELECT destination FROM virtual_aliases WHERE source='%s'
            
     sudo service postfix restart   
     postmap -q client@cyberpunk.rs mysql:/etc/postfix/mysql-email2email.cf

     postmap -q client2@cyberpunk.rs mysql:/etc/postfix/mysql-virtual-alias-maps.cf               .....rezultat je client@cyberpunk.rs
     (postmap -q server2@cyberpunk.rs mysql:/etc/postfix/mysql-virtual-alias-maps.cf              .....rezultat je server@cyberpunk.rs)           -->>
                                                                                                       virtualne aliase iz mysql baze smo povezali sa postfix-om
   
     sudo vim /etc/postfix/master.cf
             submission inet n       -       -       -       -       smtpd                        .....otkomentarišemo
                 -o syslog_name=postfix/submission                                                .....otkomentarišemo
                 -o smtpd_tls_security_level=encrypt                                              .....otkomentarišemo
                 -o smtpd_sasl_auth_enable=yes                                                    .....otkomentarišemo
                 -o smtpd_client_restrictions=permit_sasl_authenticated,reject                    .....upišemo red 
       
       
   4.6.3 *Dovecot  
   
     sudo vim /etc/dovecot/dovecot.conf                                                           .....otkomentarisati ako je komentarisano
           !include conf.d/*.conf 
           !include_try /usr/share/dovecot/protocols.d/*.protocol 
            protocols = imap lmtp pop3
            
     sudo vim /etc/dovecot/conf.d/10-mail.conf
           #mail_location = mbox:~/mail:INBOX=/var/mail/%u                                        .....zakomentarisati
            mail_location = maildir:/var/mail/vhosts/%d/%n                                        .....upisati
            mail_privileged_group = mail                                                          .....otkomentarisati i upisati mail
            
            
     sudo mkdir -p var/mail/vhosts/cyberpunk.rs                                                   .....kreiramo folder za svaki domen koji smo otvorili u MySQL tabeli
     sudo mkdir -p var/mail/vhosts/mail.cyberpunk.rs 
   
     sudo groupadd -g 5000 vmail                                                                  .....kreiramo vmail korisnika i dodeljujemo ga grupi sa ID-om 5000
     sudo useradd -g vmail -u 5000 vmail -d /var/mail
   
     sudo chown -R vmail:vmail /var/mail                                                          .....menjamo vlasnika foldera mail u vmail
   
     sudo vim /etc/dovecot/conf.d/10-auth.conf
           disable_plaintext_auth = yes                                                           .....otkomentarisano
           auth_mechanisms = plain login                                                          .....otkomentarisano i dopisno login posle plain
           #!include auth-system.conf.ext                                                         .....zakomentarisno
          
     sudo vim /etc/dovecot/conf.d/auth-sql.conf.ext                                               .....kreiramo novi fajl u koji upisujemo sopstvene podatke za 
           passdb {                                                                                    autentifikaciju
           driver = sql
           args = /etc/dovecot/dovecot-sql.conf.ext
           }
           userdb {
           driver = static
           args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
           }       
           
    sudo vim /etc/dovecot/dovecot-sql.conf.ext                                                    .....unosimo podatke usera routermail
           driver = mysql
           connect = host=127.0.0.1 dbname=servermail user=routermail password=password           .....navodimo konemciju na routermail sa paswwordom password 
           default_pass_scheme = SHA512-CRYPT
           password_query = SELECT email as user, password FROM virtual_users WHERE email='%u';
           
     sudo chown -R vmail:dovecot /etc/dovecot
     sudo chmod -R o-rwx /etc/dovecot                                                             .....menjamo vlasnika i grupu foldera dovecot
   
     sudo vim /etc/dovecot/conf.d/10-master.conf                                                  .....konfiguracija 10-master.conf
          service imap-login {
          inet_listener imap {
          #port = 143
          port = 0
          }
          service lmtp {
          unix_listener /var/spool/postfix/private/dovecot-lmtp {
          mode = 0600
          user = postfix
          group = postfix
          }
          #inet_listener lmtp {
          # Avoid making LMTP visible for the entire internet
          #address =
          #port =
          #}
          } 
          service auth {
          unix_listener /var/spool/postfix/private/auth {
          mode = 0666
          user = postfix
          group = postfix
          }
          unix_listener auth-userdb {
          mode = 0600
          user = vmail
          #group =
          }
          #unix_listener /var/spool/postfix/private/auth {
          # mode = 0666
          #}
          user = dovecot
          }
          service auth-worker {
          # Auth worker process is run as root by default, so that it can access
          # /etc/shadow. If this isn't necessary, the user should be changed to
          # $default_internal_user.
          user = vmail
          }
         
     sudo vim /etc/dovecot/conf.d/10-ssl.conf                                                     ..... SSL konfiguracija fajla iz dovecot-a
          ssl = required
          ssl_cert = </etc/dovecot/dovecot.pem
          ssl_key = </etc/dovecot/private/dovecot.pem
          
   
     openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -pkeyopt rsa_keygen_pubexp:65537 -out /etc/dovecot/dovecot.pem             .....kreiramo sertifikat
     openssl req -new -x509 -key /etc/dovecot/dovecot.pem -out /etc/dovecot/private/dovecot.pem -days 1095
                                                                                                                                           .....ili  ->
     openssl req -new -x509 -days 1000 -nodes -out "/etc/dovecot/dovecot.pem" -keyout "/etc/dovecot/private/dovecot.pem"                 -->>

     Interaktivni izlaz: 
         Country Name (2 letter code) [AU]:RS
         State or Province Name (full name) [Some-State]:Belgrade
         Locality Name (eg, city) []:Obrenovac
         Organization Name (eg, company) [Internet Widgits Pty Ltd]:SavaT
         Organizational Unit Name (eg, section) []:Hotel
         Common Name (e.g. server FQDN or YOUR name) []:mail.cyberpunk.rs
         Email Address []:routermail@cyberpunk.rs

         
   4.6.4 *SpamAssasin
   
     apt-get install spamassassin spamc                                                                 .....instaliramo SpamAssasin
   
     adduser spamd --disabled-login                                                                     .....kreiramo korisnika za   SpamAssasin - spamd
                                                                                                        .....dobijemo sledeći ispis sa   interaktivnim delom:
         Adding user `spamd' ...
         Adding new group `spamd' (1002) ...
         Adding new user `spamd' (1002) with group `spamd' ...
         Creating home directory `/home/spamd' ...
         Copying files from `/etc/skel' ...
         Changing the user information for spamd
         Enter the new value, or press ENTER for the default
	     Full Name []: router spamd                                                                    .....ovde je upisano samo ime po izboru
         Room Number []: 
	     Work Phone []: 
	     Home Phone []: 
	     Other []: 
         Is the information correct? [Y/n] y
         
    sudo vim /etc/default/spamassassin
         ENABLED=1                                                                                     .....promeniti parametar sa 0 na 1
         SPAMD_HOME="/home/spamd/"                                                                     .....dodati red
        
         #OPTIONS="--create-prefs --max-children 5 --helper-home-dir"                                  .....zakomentarisati ovu liniju a upisati sledeći red:
         
         OPTIONS="--create-prefs --max-children 5 --username spamd --helper-home-dir /home/spamd/ -s /home/spamd/spamd.log"
         
         #PIDFILE="/var/run/spamd.pid"                                                                 .....zakomentarisati
         PIDFILE="${SPAMD_HOME}spamd.pid"                                                              .....dodati red
         CRON=1                                                                                        .....promeniti parametar sa 0 na 1
            
    sudo vim /etc/spamassassin/local.cf                                                                .....fajl za postavljanje pravila protiv spam-a
        rewrite_header Subject ***** SPAM _SCORE_ *****
        report_safe             0
        required_score          5.0
        use_bayes               1
        use_bayes_rules         1
        bayes_auto_learn        1
        skip_rbl_checks         0
        use_razor2              0
        #use_dcc                0                                                                    
        use_pyzor               0
        
    sudo vim /etc/postfix/master.cf 
          smtp      inet  n       -       -       -       -       smtpd                         .....ispod ovog postojećeg reda dodati:
              -o content_filter=spamassassin                                                         što znači da svaki mail mora proći kroz SpamAssassin proveru 
            
         spamassassin unix -     n       n       -       -       pipe                                 
           user=spamd argv=/usr/bin/spamc -f -e /usr/sbin/sendmail -oi -f ${sender} ${recipient}      .....dodati dva reda na kraj dokumenta 
                                                                                                           (važno je da se drugi red uvuče za dva prazna polja 
                                                                                                           (2xtab))
        
     service spamassassin start                                                                       .....starujemo servis
     service postfix restart                                                                          .....restartujemo servis
  



   