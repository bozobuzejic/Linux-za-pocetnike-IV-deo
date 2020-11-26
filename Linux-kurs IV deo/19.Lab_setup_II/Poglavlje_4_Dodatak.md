DODATAK


Postavljanje 2 virtualna hosta bez sertifikata na server mašinu

Primeniti 4.1.1.        ->  (kreirati virtdomen.com)

Primeniti 4.2.2.1.      ->  (kreirati nekidom.rs)


    sudo vim /etc/apache2/sites-available/bezcert.com.conf                              .....kreiramo novi konfiguracioni fajl sa datim podešavanjima 
        <VirtualHost *:80>                                                                   (koji će važiti umesto /etc/apache2/sites-available/000-default.conf)  
         ServerAdmin webmaster@virtdomen.com
         ServerName virtdomen.com
         ServerAlias www.virtdomen.com
         DocumentRoot /var/www/virtdomen.com
         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
    
        <VirtualHost *:80>                                                                    
         ServerAdmin webmaster@nekidom.rs
         ServerName nekidom.rs
         ServerAlias www.nekidom.rs
         DocumentRoot /var/www/nekidom.rs
         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
        <Location />
            <RequireAll>
                Require all granted
            </RequireAll>
                Options indexes FollowSymLinks                                        .....ceo blok alternativno -> omogućuje učitavanje   web stranice koja 
       </Location>                                                                         nema naziv index.html
       </VirtualHost>  
    
                                                  esc -> shift: -> wq    -  snimimo fajl
                                                  
                                                  
    cd /etc/apache2/
    sudo a2ensite bezcert.com.conf                                                        .....omogućimo sajt koji smo konfigurisali fajlom bezcert.com.conf
    sudo a2dissite 000-default.conf                                                       .....onemogućimo default konfiguracioni fajl   (000-default.conf)
    ls sites-enabled/
    sudo apache2ctl configtest                                                            .....proverimo sintaksu konfiguracionog fajla
    sudo systemctl restart apache2                                                        .....restartujemo apache servis

    sudo vim /etc/hosts                                                                   .....otvorimo fajl sa ip adresama i hostovima (na svim mašinama) i upišemo:
        192.168.2.22    server  virtdomen.com  nekidom.rs     
                                                 esc -> shift: -> wq    -  snimimo fajl
                                                 
                                                 
    Provera:           Na clientu:  elinks http://virtdomen.com     elinks www.virtdomen.com
                                    elinks http://nekidom.rs        elinks www.nekidom.rs           (http://nekidom.rs/lab/  ->  ako je u /var/www/nekidom.rs/lab/)
                                
                                    elinks https://virtdomen.com
                                    elinks https://nekidom.rs                                -> ako probamo sa https -> ne radi jer nema dodeljnog bezbednosnog 
                                                                                            sertifikata         
                                                                                            
---
Postavljanje još 2 virtualna hosta sa sertifikatom na server mašinu

    sudo mkdir /var/www/mojprimer.com
    sudo mkdir /var/www/primermoj.org

    sudo vim /var/www/mojprimer.com/index.html
     <html>
        <head>
            <title>Ovo je sajt mojprimer.com!</title>
        </head>
        <body>
            <h1>Evo nas!  Sertifikovan mojprimer.com virtual host sad radi</h1>
        </body>
        </html>
                                                 esc -> shift: -> wq    -  snimimo fajl
                                                 
    sudo vim /var/www/primermoj.org/index.html
    <html>
       <head>
            <title>Evo ga primermoj.org!</title>
        </head>
        <body>
            <h1>Tu smo:  sertifikovan primermoj.org virtual host sad radi</h1>
        </body>
        </html>
                                                 esc -> shift: -> wq    -  snimimo fajl
                                                 

    sudo chown -R $USER:$USER /var/www/mojprimer.com  
    sudo chmod -R 755 /var/www/mojprimer.com  
    sudo chown -R $USER:$USER /var/www/primermoj.org  
    sudo chmod -R 755 /var/www/primermoj.org  

    sudo mkdir -p /etc/apache2/ssl/mojprimer.com 
    sudo mkdir -p /etc/apache2/ssl/primermoj.org

    sudo a2enmod ssl

    sudo service apache2 restart

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/mojprimer.com/apache.key -out /etc/apache2/ssl/ mojprimer.com/apache.crt
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/primermoj.org/apache.key -out /etc/apache2/ssl/primermoj.org/apache.crt


    sudo vim /etc/apache2/sites-available/mojprimer.com.conf
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName mojprimer.com
        DocumentRoot /var/www/mojprimer.com
        ServerAlias www.mojprimer.com mojprimer.com
    </VirtualHost>
    <IfModule mod_ssl.c>
    <VirtualHost *:443>

        ServerAdmin webmaster@localhost
        ServerName mojprimer.com
        DocumentRoot /var/www/mojprimer.com
        ServerAlias www.mojprimer.com mojprimer.com
        #   SSL Engine Switch:
        #   Enable/Disable SSL for this virtual host.
        SSLEngine on

        #   A self-signed (snakeoil) certificate can be created by installing
        #   the ssl-cert package. See
        #   /usr/share/doc/apache2.2-common/README.Debian.gz for more info.
        #   If both key and certificate are stored in the same file, only the
        #   SSLCertificateFile directive is needed.
        SSLCertificateFile /etc/apache2/ssl/mojprimer.com/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/mojprimer.com/apache.key
    </VirtualHost>
    </IfModule>
                                                                  esc -> shift: -> wq    -  snimimo fajl
                                                                  
    sudo vim /etc/apache2/sites-available/primermoj.org.conf
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName primermoj.org
        DocumentRoot /var/www/primermoj.org
        ServerAlias www.primermoj.org primermoj.org
    </VirtualHost>
    <IfModule mod_ssl.c>
    <VirtualHost *:443>

        ServerAdmin webmaster@localhost
        ServerName primermoj.org
        DocumentRoot /var/www/primermoj.org
        ServerAlias www.primermoj.org primermoj.org

        #   SSL Engine Switch:
        #   Enable/Disable SSL for this virtual host.
        SSLEngine on

        #   A self-signed (snakeoil) certificate can be created by installing
        #   the ssl-cert package. See
        #   /usr/share/doc/apache2.2-common/README.Debian.gz for more info.
        #   If both key and certificate are stored in the same file, only the
        #   SSLCertificateFile directive is needed.
        SSLCertificateFile /etc/apache2/ssl/primermoj.org/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/primermoj.org/apache.key
    </VirtualHost>
    </IfModule>
                                                                 esc -> shift: -> wq    -  snimimo fajl


    sudo vim /etc/apache2/ports.conf                                                      .....alternativno ako hoćemo da dodamo slušanje na još nekom portu (8080)
    Listen 8080              
                                                                 esc -> shift: -> wq    -  snimimo fajl    

    sudo a2ensite mojprimer.com
    sudo a2ensite primermoj.org

    sudo vim /etc/hosts
    sudo service apache2 restart

Na klijentu:

    sudo vim /etc/hosts 
    192.168.2.22    server  virtdomen.com  nekidom.rs mojprimer.com primermoj.org 
    192.168.2.22    www.virtdomen.com www.nekidom.rs www.primermoj.org www.mojprimer.com
                                                                 esc -> shift: -> wq    -  snimimo fajl  


Provera:          

                   Na clientu:  elinks http://mojprimer.com     elinks www.mojprimer.com 
                                elinks http://primermoj.org     elinks www.primermoj.org 
