LAB SETUP 1.5 - Instaliranje štampača na SHARE mašini


     0.  Podešavanje VirtualBox-a (enable usb setting)
         
         echo $USER                                                                .....na mašini na kojoj je instaliran VirualBox proverimo ime usera
         compgen -g                                                                .....proverimo da li postoji group users za vbox
               vboxusers                                                                .....ustanovimo da postoje vboxusers
         usermod -G vboxusers -a $USER                                             .....dodajemo našeg usera sa glavne mašine u grupu vboxusers
         reboot                                                                    .....restartujemo računar


     1.  Instaliranje cups servisa na SHARE mašini, za konfiguraciju štampača
     
         yum install cups                                                          .....instaliramo cups - program za podešavanje štampača
         systemctl enable --now cups.service                                       .....omogućimo i pokrenemo progran
         
         netstat -tulpn | grep cups                                                .....proveravamo da li servis radi i na kom portu
         
         
         iptables -vnL --line-number                                                 .....izvršimo proveru upisanih pravila firewall-a
         iptables -I INPUT 10 -m state --state NEW -p tcp --dport 631 -j ACCEPT      .....dodamo pravila da gw mašina sluša na portu 631           
         iptables-save > /etc/sysconfig/iptables                                     .....snimimo promene
         systemctl reload iptables.service                                           .....restartujemo iptables
          
         
         vim /etc/cups/cupsd.conf                                                  .....otvaramo konfiguracioni fajl za editovanje
         
           DefaultEncryption Never                                                      .....upisujemo u prvi red (a iznad MaxLogSize 0)
           MaxLogSize 0
           ...
           # Only listen for connections from the local machine.
           Listen localhost:631
           Listen 192.168.60.12:631                                                     .....ubacujemo novi reda sa ip adresom SHARE mašine koja sluša na portu 631
           Listen /var/run/cups/cups.sock
           ...
           # Restrict access to the server...
           <Location />
           Order allow,deny
           allow localhost                                                              .....ubacujemo red kojim dozvoljavamo pristup serveru sa localhosta i
           allow 192.168.60.0/24                                                        .....ubacujemo red kojim dozvoljavamo pristup svima iz datog rapona ip adresa
           </Location>
           ...
           # Restrict access to the admin pages...
           <Location /admin>
           Order allow,deny
           allow localhost                                                              .....dozvoljavamo pristup administrativnoj stranici sa localhosta i 
           allow 192.168.60.0/24                                                        .....dozvoljavamo pristup administrativnoj stranici svima iz datog raspona ip          
           </Location>                                                                       adresa
           ...
                                            esc -> shift: -> wq                         .....snimimo izmene u fajlu
          
          
     2.  Instaliranja printer drajvera (Brother HL-2030) na SHARE mašini: 
         
         wget https://download.brother.com/welcome/dlf006893/linux-brprinter-installer-2.2.1-1.gz        .....downloadujemo drajver install tool za Brother štampače
         gunzip linux-brprinter-installer-2.2.1-1.gz                                                     .....raspakujemo paket
         bash linux-brprinter-installer-2.2.1-1                                                          .....pokrenemo instalaciju
         service cups restart                                                                            .....restartujemo cups
         lpoptions -d HL2030                                                                             .....podesimo da Brother HL2030 bude default štampač
         reboot                                                                                          .....restartujemo SHARE mašinu
                                            
                                             
     3.  Podešavanje cups servisa sa druge mašine (klijent) na SHARE mašini (server) 
         (koristimo web browser klijenta)  
      
         http://192.168.60.12:631                                                      .....u grafičkom okruženju, ako ga ima, upišemo adresu i otvorimo cups 
                                                                                            stranicu za podešavanje štampača
         http://localhost:631                                                               
                                                                                             
         elinks http://192.168.60.12:631                                               .....ako nemamo grafički prikaz onda otvarmo sa elinksom sa druge mašine
         elinks http://localhost:631                                                   .....ako nemamo grafički prikaz onda otvarmo sa elinksom na samoj mašini:
                 
         Administration -> Add printers ->  USB PRINTER (ako smo podesili enable usb setting, štampač se pojavljuje na usb priključku) ili Internet Printing Protocol 
         (http) -> Continue ->  Connection: ipp://192.168.60.12:631/printers/HL2030 (ako smo podesili enable usb setting, štampač se pojavljuje na usb priključku) 
         ili 
         socket://hostname:9100 -> Continue ->
            Name: NEKOIMEŠTAMPAČA (Brother_HL2030)
            Description: ModelŠtampača (Brother_HL2030)
            Location: share 
         Continue -> 
            Make: odaberemo marku (proizvođača) štampača (Brother_HL2030)
         Continue->
            Model: odaberemo model izabrane marke štampača (za Brother_HL2030)
            Add Printer -> 
            Media size: A4
            Resolution: odaberemo rezoluciju
            Set Default Options                                                         .....instaliran i podešen štampač (na SHARE mašini)
            
            
     4.  Podešavanje štampača na klijentu (korisniku štampača sa druge mašine)
         (primer rađen na XUbuntu u VurtualBox-u)
          
         VirtualBox -> XUbuntu -> Settings -> Network -> Adapter 1 -> Host-only Adapter -> vboxnet1 -> Ok -> Start      .....dodamo xubuntu u mrežu 192.168.60.0/24
          
         Application Launcher -> Printers -> Add -> Network Printer -> Find Network Printer -> Host: 192.168.60.12 -> Find -> Devices: select printer 
         (Brother_HL2030)-> Forward -> Choose Driver: select driver (Gutenprint driver) -> User: root, Password: password -> Authenticate ->  Select the printer's 
         manufacturer -> Forward -> Select the printer's model -> Forward -> Apply -> Print Test Page             .....podesimo da štampač sa WEB mašine bude default 
                                                                                                                       štampač XUbuntu-a (to se može sprovesti nad 
                                                                                                                       svim mašinama u mreži)
                                                                                                                                                                                   
     5.  Podešavanja na SHARE mašini
       
         cupsaccept NAZIVŠTAMPAČA                                                      .....prihvatimo štampač
         cupsenable NAZIVŠTAMPAČA                                                      .....omogućimo štampač
         lpoptions -d NAZIVŠTAMPAČA                                                    .....odredimo default štampač
         lpstat -p -d                                                                  .....prikaz štampača u sistemu
         lpq                                                                           .....prikaz aktuelnog stanja u redu za štampanje
          
         lpr /path/to_folder/nekifajl                                                  .....štampamo nekifajl
         lprm 3                                                                        .....uklanjamo fajl za štampanje pod rednim brojem 3                                                                                                      
          
           
           
        