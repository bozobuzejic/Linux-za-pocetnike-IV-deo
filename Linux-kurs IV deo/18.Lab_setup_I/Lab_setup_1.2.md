LAB SETUP 1.2 (konfiguracija mreže od 4 OS (CentOS 7) u VirualBox-u) - SSH

  a. Podešavanje GW mašine
     
     a.1.   systemctl stop firewalld
            systemctl disable firewalld
            systemctl mask firewalld                     .....isključimo firewalld servis da ne dolazi do konflikta sa iptables servisom
            
     a.2.   yum install epel-release.noarch
            yum update
            yum install iptables-services.x86_64
            yum install vim                              .....izvršimo instalaciju repozitorijuma za CentOS i update sistema
            
     a.3.   systemctl start iptables
            systemctl enable iptables                    .....pokretanje iptables i podešavanje da se pokreće sa pokretanjem sistema po default-u
                rpm -qc iptables-services                     .....izlistava fajlove u kojima su podešavanja iptables (na CentOS na primer)
                
     a.4.  *-   iptables -t nat -A POSTROUTING -s 192.168.60.0/24 -o enp0s8 -j MASQUERADE   .....podešavanje da se sav saobraćaj iz mreže sa datim skupom IP adresa 
                                                                                                NAT-uje (da može da izadje na internet)
                                                                                             
          *-  iptables -vnL --line-numbers                                                 .....izlistamo zadata pravila za podešavanje saobraćaja
              iptables -D FORWARD 1                                                        .....iz filter table izbrišemo lanac FORWARD (prosleđivanje) pod brojem 1
            
              vim /etc/sysctl.conf             ili                                         .....otvorimo označeni fajl i dodamo na dnu red:
              vim etc/sysctl.d/sysctl.conf 
                                   net.ipv4.ip_forward = 1                                      .....omogućimo forward
                                   
          *-   iptables-save > /etc/sysconfig/iptables                                     .....snimimo promene koje smo napravili
          
     a.5.    vim /etc/hosts                                                                 ....otvorimo označeni fajl i ispod postojećeg teksta upisujemo
                            192.168.60.2  gw                                                   (podešavanje hosts fajla)
                            192.168.60.10 web
                            192.168.60.11 db
                            192.168.60.12 share
            
     a.6.    mkdir .ssh                                        .....kreiramo u root direktorijumu u kome smo .ssh folder u koji ćemo smestiti config fajl za prečice
                                                                   ka drugim mašinamaa
                                                                   
            vim .ssh/config                                   .....sa vim otvaramo novi fajl (koji ćemo posle editovanja snimiti u .ssh folder) i upisujemo
            
                    Host gw                                     .....alias 
                         User root                              .....navodimo ime naloga 
                         Hostname 192.168.60.2                  .....navodimo IP adresu 
                         Port 22                                .....navodimo broj porta 
                         #IdentityFile ~/.ssh/id_rsa            .....putanju do ključa zakomentarišemo (u slučaju da zatreba)   
            
                    Host web                                   
                         User root                              
                         Hostname 192.168.60.10                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa            
                         
                    Host db                                   
                         User root                              
                         Hostname 192.168.60.11                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa      
                                                                
                    Host share                                   
                         User root                              
                         Hostname 192.168.60.12                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa  

                                           esc -> :wq                             .....snimimo tekst i dokument 
                         
                         
                         
  b. Podešavanje WEB, DB i SHARE mašina (dati postupak se realizuje na svakoj mašini)
  
  
     b.1.   systemctl stop firewalld
            systemctl disable firewalld
            systemctl mask firewalld                     .....isključimo firewalld servis da ne dolazi do konflikta sa iptables servisom
            
  
     b.2.    vi /etc/selinux/config                           .....sa vi editorom otvorimi konfiguracioni fajli (da bi onemogućili selinux) upisujemo:
                                #SELINUX=enforcing                 .....zakomentarišemo (stavimo znak ispred #) i upišemo novi red da ga onemogućimo:
                                SELINUX=permissing                   
       
     b.3.   nmtui                             .....pokrenemo nmtui i obavimo podešavanje dns servera
     
                  Edit a connection --> enp0s3 --> edit --> DNS servers upisati 147.91.8.6 --> ok --> back --> Activate a connection -->  
                  deactivate - activate (dupli klik na enp0s3) --> back --> ok
                  
     b.4.   yum install iptables-services.x86_64              .....instaliramo iptables servis
            systemctl enable --now iptables                   .....omogućimo i pokrenemo servis
            iptables -I INPUT 4 -m state --state NEW -p tcp --dport 9999 -j ACCEPT     ....lanac koji dopušta rad na portu 9999
            iptables-save > /etc/sysconfig/iptables           .....snimimo promenu (trajno)
   
     b.5.   systemctl status sshd.service                     .....proveravamo da li je instaliran ili aktivan ssh servis (yum install install openssh-server)
            systemctl start sshd.service
            systemctl enable sshd.service                          .....pokrenemo ssh servis i omogućimo da se startuje automatski  
            systemctl restart sshd.service                    .....restartujemo servis
            
     b.6.   vim /etc/ssh/sshd_config                          .....otvorimo sshd_config fajl i upišemo ispod linije #Port 22 Port 9999:
    
                                #Port 22
                                Port 9999                     .....snimimo i izađemo iz fajla
                                                 
     b.7.   vim /etc/hosts                                    .....otvorimo označeni fajl i ispod postojećeg teksta upisujemo
                            192.168.60.2  gw                       (podešavanje hosts fajla)
                            192.168.60.10 web
                            192.168.60.11 db
                            192.168.60.12 share                                                 
                                
     b.8.   mkdir .ssh                                        .....kreiramo u root direktorijumu u kome smo .ssh folder u koji ćemo smestiti config fajl za prečice
                                                                   ka drugim mašinamaa
                                                                   
            vim .ssh/config                                   .....sa vim otvaramo novi fajl (koji ćemo posle editovanja snimiti u .ssh folder) i upisujemo
            
                    Host gw                                     .....alias 
                         User root                              .....navodimo ime naloga 
                         Hostname 192.168.60.2                  .....navodimo IP adresu 
                         Port 22                                .....navodimo broj porta 
                         #IdentityFile ~/.ssh/id_rsa            .....putanju do ključa zakomentarišemo (u slučaju da zatreba)   
            
                    Host web                                   
                         User root                              
                         Hostname 192.168.60.10                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa            
                         
                    Host db                                   
                         User root                              
                         Hostname 192.168.60.11                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa      
                                                                
                    Host share                                   
                         User root                              
                         Hostname 192.168.60.12                 
                         Port 9999                               
                         #IdentityFile ~/.ssh/id_rsa  

                                        esc -> :wq            .....snimimo tekst i dokument  
                                    
     b.9.   yum install epel-release.noarch
            yum update
            yum install vim                                   .....izvršimo instalaciju repozitorijuma za CentOS, vim-a i update sistema
            
            
    
  c. Kreiranje sertifikacionog ključa (ključ kreiramo na gw mašini i "šaljemo" ga na web, db i share mašinu)
    
     c.1    ssh-keygen -t rsa                 .....komanda kojom se generiše bezbednosni ključ (prilikom izrade možemo stviti password na sam ključ)
     
     c.2    ssh-copy-id web                   .....komande kojim se ključ sa gw mašine šalje na web, db i share mašinu što će gw mašini omogućiti da se uloguje
            ssh-copy-id db                         u navedene mašine bez traženja passworda
            ssh-copy-id share                      (.ssh/authorized_keys password se kopira u authorized_keys fajl u .ssh folderu
                                                  
            
                                       
*01 - Podešavanje web servera (WEB mašina) - APACHE web server (sa postavljanjem 2 virtual hosta: wp.web i int.web)

         1.       yum groups install "Basic Web Server"                           .....instalirati Apache web server
         2.       systemctl enable --now httpd.service                            .....komanda da se httpd.service startuje podrazumevano sa pokretanjem sistema
         3.       yum install elinks                                              .....instaliramo tekstualni web browser na sve mašine (gw, web, db i share) 
         
         4.       Na gw mašini dodati pravila za iptables.service:
         
                  Napomena: pre dodavanja pravila sa komandom: iptables -vnL --line-numbers         .....proveriti rene brojeve već postavljenih pravila
         
             4.1.     iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT          .....da gw sluša na portu 80 (http)
             4.2.     iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT         .....da gw sluša na portu 443 (https)
             4.3      iptables-save > /etc/sysconfig/iptables                                       .....snimimo izmene
             4.4      systemctl restart iptables.service                                            .....restartujemo servis
             
             
         5.       Na web mašini dodati pravila za iptables.service:
         
             Napomena: pre dodavanja pravila sa komandom: iptables -vnL --line-numbers              .....proveriti rene brojeve već postavljenih pravila
         
             5.1.     iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT          .....da web sluša na portu 80 (http)
             5.2.     iptables -I INPUT 7 -m state --state NEW -p tcp --dport 443 -j ACCEPT         .....da web sluša na portu 443 (https)
             5.3      iptables-save > /etc/sysconfig/iptables                                       .....snimimo izmene
             5.4      systemctl restart iptables.service                                            .....restartujemo servis
                 
            
         6.        Na sve 4 mašine (gw, web, db i share):
             
             6.1   vim /etc/hosts                                                 .....otvorimo postojeći fajl i upišemo:
                       
                       192.168.60.10     wp.web int.web 
                       
                   systemctl reload httpd.service                                 .....uradimo reload sshd servisa na web mašini    
                       
         7.        Na web mašini (kreiramo foldere, fajlove i konfigurišemo hostove wp.web i int.web):
        
             7.1.     mkdir /var/www/wp                                           .....kreiramo folder wp
                      vim /var/www/wp/index.html                                  .....vimom kreiramo novi dokument index.html u koji upisujemo:
                              <h1>Ovo je mesto za wp.web prezentaciju</h1>        
                                                                          esc -> shift: -> wq   -  snimimo fajl 
                                                                                  
             7.2.    mkdir /var/www/int                                           .....kreiramo folder int
                      vim /var/www/int/index.html                                 .....vimom kreiramo novi dokument index.html u koji upisujemo:
                              <h1>int.web prezentacija</h1>        
                                                                          esc -> shift: -> wq   -  snimimo fajl 
                                                                          
         8.  8.1.    vim /etc/httpd/conf.d/wp.conf                                .....kreiramo konfiguracioni fajl wp.conf u koji upisujemo:
        
                              <VirtualHost *:80>
                                    ServerAdmin webmaster@wp.web
                                    DocumentRoot "/var/www/wp/"
                                    ServerName wp.web
                                    CustomLog logs/wp.web-access.log common
                                    ErrorLog logs/wp.web-error.log
                              </VirtualHost>                              esc -> shift: -> wq   -  snimimo fajl
                              
                              
                
             8.2.    vim /etc/httpd/conf.d/int.conf                               .....kreiramo konfiguracioni fajl int.conf 
             
             
                 Varijante:
                 
                 8.2.1.       vim /etc/httpd/conf.d/int.conf                      .....varijanta u kojoj pristup web strani i folderu /docs/ ima samo jedna IP adresa 
                                                                                       i jedan korisnik sa odgovarajućim Login imenom i passwordom
                                                                                       (jedino sa naznačene adrese je dozvoljena autentifikacija - niko drugi ne može
                                                                                       da priđe)
                                                                                           
                              <VirtualHost *:80>
                                    ServerAdmin webmaster@int.web
                                    DocumentRoot "/var/www/int/"
                                    ServerName int.web
                                    CustomLog logs/int.web-access.log common
                                    ErrorLog logs/int.web-error.log
                              <Location "/">
                                   <RequireAll>
                                        Require ip 192.168.60.2                   .....ili neka druga
                                        AuthType Basic
                                        AuthName "Linux kurs"
                                        AuthUserFile /etc/httpd/htpasswd
                                        Require valid-user
                                  </RequireAll>
                                        Options indexes FollowSymLinks            .....opcija omogućuje učitavanje i fajlova iz foldera docs koji nisu index.html
                             </Location>
                             <Location /var/www/int/docs/>
                                  <RequireAll>
                                        Require all denied
                                  </RequireAll>
                             </Location>
                             </VirtualHost>                                                esc -> shift: -> wq   -  snimimo fajl 
                             
                             
                8.2.2.       vim /etc/httpd/conf.d/int.conf                       .....varijanta u kojoj pristup web strani imaju svi, folderu /docs/ ima direktan 
                                                                                       pristup (bez login-a) samo jedna ip adresa (192.168.60.2) a ostali samo 
                                                                                       ako znaju Login ime i password                                       
                             <VirtualHost *:80>
                                    ServerAdmin webmaster@int.web
                                    DocumentRoot "/var/www/int/"
                                    ServerName int.web
                                    CustomLog logs/int.web-access.log common
                                    ErrorLog logs/int.web-error.log
                             <Location />
                                        Require all granted
                                        Options indexes FollowSymLinks
                             </Location>
                             <Location /docs/>
                                        Require ip 192.168.60.2
                                        AuthType Basic
                                        AuthName "Linux kurs"
                                        AuthUserFile /etc/httpd/htpasswd
                                        Require valid-user
                             </Location>
                             </VirtualHost>                                                 esc -> shift: -> wq   -  snimimo fajl 
       
       
                8.2.3.       vim /etc/httpd/conf.d/int.conf                       .....varijanta u kojoj pristup web strani imaju svi a folderu /docs/ imaju svi sa   
                                                                                       odgovarjućim Login imenom i passwordom
                                                                                       
                             <VirtualHost *:80>
                                    ServerAdmin webmaster@int.web
                                    DocumentRoot "/var/www/int/"
                                    ServerName int.web
                                    CustomLog logs/int.web-access.log common
                                    ErrorLog logs/int.web-error.log
                             <Location />
                                  <RequireAll>
                                        Require all granted
                                  </RequireAll>
                                    Options indexes FollowSymLinks
                             </Location>
                             <Location /docs/>
                                        AuthType Basic
                                        AuthName "Linux kurs"
                                        AuthUserFile /etc/httpd/htpasswd
                                        Require valid-user
                             </Location>
                             </VirtualHost>                                                    esc -> shift: -> wq   -  snimimo fajl 
                                                                 
                                                                         
               
         9.    a. ili b. (alternativno)
        
                    a.   mkdir /var/www/int/docs                                 .....kreiramo folder docs
                         vim /var/www/int/docs/NekiNaslov.txt                    .....sa vim kreiramo NekiNaslov.txt fajl u koji upisujemo tekst po želji:
                                    Upisujemo nešto
                                                                          esc -> shift: -> wq   -  snimimo fajl
                                      
                    b.   iskopiramo neki dokument u folder:  /var/www/int/docs    
                    
                    
       10.    htpasswd -c /etc/httpd/htpasswd  Kurs  /ili neko drugo ime          .....kreiramo user i password za pristup fajlovima u /var/www/html/docs/
              password: password  ->  enter  ->  password                         
                 
                 
       11.    Proba:
        
              elinks http://wp.web                                                .....isprobamo sa elinks-om wp.web                     
              elinks http://int.web/docs/                                         .....isprobamo sa elinks-om int.web  



*02 - Podešavanje database servera (DB mašina) - instaliranje MySQL iz yum repozitorijuma za CentOS 7


      https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/
      https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html          .....stranice sa uputsvima za instalaciju mysql servera za yum repo
      https://dev.mysql.com/downloads/file/?id=484922                                   .....stranica za preuzimanje linka za download za yum repo (za RedHat linux-e)

      https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm              .....link za download repozitorijuma

      1.  yum install wget                                                         .....instaliramo wget da bi smo mogli da pokrenemo download uz pomoć linka 
          wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm     .....downloadujemo program
          yum localinstall mysql80-community-release-el7-3.noarch.rpm                        .....pokrećemo instalaciju repozitorijuma za mysql
          yum install mysql-community-server                                                      .....instaliramo server                          
          systemctl status mysqld.service                                                         .....proverimo status servisa
          systemctl start mysqld.service                                                          .....starujemo servis

      2.  sudo grep 'temporary password' /var/log/mysqld.log                            .....ulazimo u mysql i iščitavamo liniju sa fabričkim passwordom:

                   019-05-28T18:00:10.349515Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Mm=h=Hqrw8J?    
       
      3.  mysql -uroot -p                                                               .....komanda za ulazak u program mysql
                        password: Mm=h=Hqrw8J?                                               .....na zahtev, upisujemo password (Mm=h=Hqrw8J?)
                              Welcome to the MySQL monitor.
                              -----------------------------
                              -----------------------------
      4.  mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'P4ssw0rd!';               .....upisujemo novi password: P4ssw0rd!  
          mysql> flush privileges;                                                           .....potvrđujemo izmenu

      5.  mysql> show databases;                                                        .....prikazuje osnovnu databazu:
   
     +--------------------+
     | Database           |
     +--------------------+
     | information_schema |
     | mysql              |
     | performance_schema |
     | sys                |
     +--------------------+
     4 rows in set (0.05 sec)
     
      6.  mysql> SELECT User,Host FROM mysql.user;                                      .....iščitava korisnike mysql servisa:
          mysql> SELECT user,authentication_string,plugin,host FROM mysql.user;         .....detaljnije iščitava  

      7.  mysql> ^DBye      (ctrl D, exit)                                              .....izlazimo iz programa

                 
*03 - Podešavanja DB i WEB mašina - instaliranje PhP i WORDPRESS-a 

      a)  DB Mašina - MySQL
       
          a.1  Podešvanje MySQL servisa
             
               mysql -uroot -p                                                          .....ulazimo u program mysql
                             password: P4ssw0rd!
                             
               mysql> create database wordpress;                                                              .....kreiramo databazu wordpress                                           
               mysql> CREATE USER 'wp'@'192.168.60.10' IDENTIFIED BY 'P4ssw0rd!';                             .....kreiramo usera wp'@'192.168.60.10
               mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wp'@'192.168.60.10';                            .....useru dajemo sve privilegije nad databazom 
                                                                                                                   wordpress
               mysql> ALTER USER 'wp'@'192.168.60.10' identified with mysql_native_password by 'P4ssw0rd!';   .....upisujemo novu komandu za istog usera
               mysql> flush privileges;                                                                       .....potvrdimo privilegije
               mysql> exit                                                                                    .....izađemo iz programa
               
               
          a.2  Podešavanje iptables firewall servisa  
          
               iptables -I INPUT 6 -m state --state NEW -p tcp --dport 3306 -j ACCEPT                         .....omogućimo slušanje na portu 3306 za mysql
               iptables-save > /etc/sysconfig/iptables                                                        .....potvrdimo dodati lanac u iptables
               systemctl restart iptables.service                                                             .....restartujemo servis
               
               
      b)  WEB mašina - PhP - MySQL
      
          b.1  Instaliranje PhP
             
               https://rpms.remirepo.net/wizard/                                        .....uz pomoć wizarda dolazimo do stranice sa uputstvima i potrebnim     
                                                                                             linkovima za download i instalaciju
               yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm       .....instaliramo repozitorijum
               yum install yum-utils                                                    .....instaliramo dodatne alate i programe
               yum-config-manager --enable remi-php73                                   .....maskira starije verzije PhP i njegovih pratećih dodataka  
               yum install php                                                          .....instaliramo php
               php -v                                                                   .....proveravamo verziju PhP
               
               
          b.2  Instaliranje dodatnih paketa za php:
          
               yum install php-fpm
               systemctl enable --now php-fpm.service
               yum install php-mysql
                    
      
          b.3  Instaliranje WordPress
          
               cd /var/www/wp/                                                          .....premestimo se u folder wp na web mašini
               wget https://wordpress.org/latest.tar.gz                                 .....sa interneta downloadujemo instalacione fajlove za wordpress u folder 
                                                                                             wp
               tar xfzv latest.tar.gz                                                   .....raspakujemo instalaciju - kreira se folder wordpress u folderu wp
               chown apache:apache wordpress -R                                         .....promenimo usera-vlasnika foldera wordpress (apache:apache)
               systemctl reload httpd.service                                           .....restartujemo apache servis da potvrdimo izmene
               
               
          b.4  Instalirnje mysql klijenta 
             
               yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm   .....pokrećemo instalaciju repozitorijuma za mysql 
               yum search mysql | grep client                                                     .....učitava listu mysql klijenata za instaliranje da odaberemo
               yum install mysql-community-client.x86_64                                          .....instaliramo odabranog mysql klijenta
               
               Provera:
               mysql -u wp -h db -p                                                               .....ulazimo u wordpress database na db mašini (iz web mašine)
     
              
          b.5  Dodatno podešavanje 
              
               vim /etc/httpd/conf.d/wp.conf                                            .....otvorimo za editovanje konfiguracioni fajl za web stranu koju smo ranije
                                                                                             kreirali
               <VirtualHost *:80>
                    ServerAdmin webmaster@wp.web
                    DocumentRoot /var/www/wp/wordpress/                                      .....dodali smo /wordpress/ u putanju DocumentRoot 
                    Options indexes                                                          .....dodali smo liniju koja omogućuje učitavanje fajlova raznih formata
                    ServerName wp.web  
                    CustomLog logs/wp.web-access.log common
                    ErrorLog logs/wp.web-error.log
               </VirtualHost>
                                                      esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                                                                
               systemctl reload httpd.service                                                     .....restartujemo apache servis da potvrdimo izmene    
                 
               Provera iz GW mašine:
               elinks http://wp.web                                                               .....ako otvori ulaznu stranu za podešavanje wordpress i dođe
                                                                                                       do strane za konfiguraciju wp onda je sve podešeno
                                                                                                       
               Napomena: Provera se može izvršiti i iz drugog sistema koji ima grafičko okruženje
                         i pristup mreži iz web browswera:
                         
                         vim /etc/hosts                                                 .....na drugom sistemu otvorimo postojeći fajl i upišemo:
                             192.168.60.10  wp.web int.web 
                                                esc -> shift: -> wq                          .....snimimo izmene u fajlu
                                                
                         http://wp.web                                                  .....u web browser-u upišemo i otvara nam stranicu na WEB mašini da dovršimo 
                                                                                             konfiguraciju wordpress-a (dajemo naslov stranice, user name, password)
                                                                                             
                                                                                             
                                          