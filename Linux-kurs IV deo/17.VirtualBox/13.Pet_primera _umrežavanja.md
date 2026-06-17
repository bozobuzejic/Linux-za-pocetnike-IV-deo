DODATAK (5 primera umrežavanja)

*Povezivanje u mrežu dve virtuelne mašine u VirtualBox-u  ----  varijanta 1    ---    NAT networks  
      
        Primer: Povezivanje XUbuntu i ArchLinux mašina instaliranih u VirtualBox
        
        1.  File -> Preferences -> Network -> Adds New NAT networks
        2.  Kliknuti na novootvorenu NatNetwork -> Edit selected NatNetwork
        3.  U kućicu Network Name upisati novi naziv mreže (na primer LAB 1)
        4.  U kućicu CIDR upisati adresu 172.25.1.0/24 i pritisnuti OK (za izlazak iz edit prozora) 
        5.  Pritisnuti OK i zatvoriti prozor Preferences
        6.  Izabrati prvi OS (na datom primeru: XUbuntu) i klikniti na dugme Settings ->
        7.  Settings -> Network ->  Adapter 1 -> u padajućem meniju Atached to: izabrati opciju NAT Network 
        8.  U padajućem meniju Name: izabrati LAB 1 -> OK (izlazimo iz podešavanja)
        9.  Izabrati drugi OS (na datom primeru: Arch) i klikniti na dugme Settings ->
        10. Ponoviti sve iz tačaka 7. i 8.
        11. Startovati oba sistema
        12. Nakon logovanja sistema otvoriti (kao root) terminale  na svakom i proveriti da li su aktivni mrežni uređaji na njima ->
        13. Kucati: ip a               .....videti da li na uređaju enp0s3 postoji linija (ili slična) sa novododeljenom adresom: 
        
              a)     inet 172.25.1.4/24 brd 172.25.1.255 scope global dynamic noprefixroute enp0s3  --> dodeljena adresa je: 172.25.1.4   (na XUbuntu-u)  
              b)     inet 172.25.1.5/24 brd 172.25.1.255 scope global dynamic noprefixroute enp0s3  --> dodeljena adresa je: 172.25.1.5   (na Arch-u)
                                           
        14. Kucati: nmcli con show     .....proveriti na obe mašine da li je veza aktivna (ako je tekst zelene boje, veza je aktivna) 
        15. Proveriti na oba sistema da li konekcije rade:
               
              a.     Na terminalu XUbuntu-a kucati:   ping 172.25.1.5   (proveravamo vezu ka Arch-u)    --> ako vidimo odgovor, veza je dobra
              b.     Na terminalu Arch-a kucati:      ping 172.25.1.4   (proveravamo vezu ka XUbunu-u)  --> ako vidimo odgovor, veza je dobra
              
        16. Pišemo:   systemctl status sshd.service                   .....proveravamo na oba OS da li je instaliran i da li je aktivan SSH (ili SSHD) servis,
                                                                           (ako nije instaliran, onda ga instaliramo; ako nije aktivan, aktiviramo ga)
        17. Zadajemo komande za povezivanje preko komandnih linija: 
        
              a-     U XUbuntu:  ssh userArch@172.25.1.5      (da se povežemo sa Arch-om preko njegove adrese 172.25.1.5)      ---userArch je ime usera sa Arrch-a
                                 password: - upisujemo password koji smo dodelili useru na Archu
              
              b-     U Arch:     ssh userXUbuntu@172.25.1.4   (da se povežemo sa XUbuntuom-om preko njegove adrese 172.25.1.4) ---userXUbuntu je ime usera sa 
                                                                                                                                  XUbuntu-a
                                 password: - upisujemo password koji smo dodelili useru na XUbuntu
                                 
        18. Ako je sve u redu, komandna linija XUbuntu-a se nakon povezivanja prebacuje u korisnički nalog na ArchLinux-u a
            komandna linija Arch-a u korisnički nalog XUbuntu-a
        19. Mašine se mogu povezati i preko File Managera oba sistema (to je sftp konekcija):
           
              a. Otvorimo File Manager na XUbuntu-u, kliknemo na dugme Browse Network i u Location Selectoru (View -> Location Selector ->) upišemo adresu:
              
                     ssh://172.25.1.5      ---> pritisnemo dugme Connect -> u prozorčić koji se otvori upišemo ime usera sa Arch-a i njegov password      -> OK
                     
              b. Otvorimo File Manager na Arch-u, kliknemo u polje Connect to server upišemo adresu:
              
                     ssh://172.25.1.4      ---> pritisnemo dugme Connect -> u prozorčić koji se otvori upišemo ime usera sa XUbuntu-a i njegov password   -> OK       
             
    Napomena: Ako hoćemo da  pristupamo sa realne mašine ili sa interneta nekoj mašini u VB, preko NAT network, potrebno je podesiti port forwarding. 
    
              Posle tačke 3, pritisnuti dugme Port Forwarding i uneti (primer):
              
              Name: ssh    Protocol: TCP    Host IP:         Host port: 2020      Guest IP:172.25.1.5         Guest Port: 22
              Name: http   Protocol: TCP    Host IP:         Host port: 8020      Guest IP:172.25.1.5         Guest Port: 80      
     
     
*Povezivanje u mrežu dve virtuelne mašine u VirtualBox-u  ----  varijanta 2    ---  Host Network Manager 

        Primer: Povezivanje XUbuntu i ArchLinux mašina instaliranih u VirtualBox
        
        1.  File -> Host Network Manager -> Create
        2.  Configure Adapter Manully -> IPv4 Adress: 192.168.56.1 -> IPv4 Network Mask: 255.255.255.0
        3.  DHPC Server -> Enable server -> Server adress: 192.168.56.100 -> Server mask:255.255.255.0 -> Lower address bound -> 192.168.56.101 ->
            Upper address bound -> 192.168.56.254 -> Aplly -> Close
        4.  Izabrati prvi OS (na datom primeru: XUbuntu) i klikniti na dugme Settings ->
        5.  Settings -> Network ->  Adapter 1 -> u padajućem meniju Atached to: izabrati opciju Host-Only Adapter 
        6.  U padajućem meniju Name: izabrati Vboxnet0 -> OK (izlazimo iz podešavanja)
        7.  Izabrati drugi OS (na datom primeru: Arch) i klikniti na dugme Settings ->
        8.  Ponoviti sve iz tačaka 4. i 5.
        9.  Startovati oba sistema
        10. Zadajemo komande za povezivanje preko komandnih linija: 
        
            a-   U XUbuntu: ssh userArch@192.168.56.102 (da se povežemo sa Arch-om preko njegove adrese 192.168.56.102)        ---userArch je ime usera sa Arrch-a
                            password: - upisujemo password koji smo dodelili useru na Archu
              
            b-   U Arch:    ssh userXUbuntu@192.168.56.101 (da se povežemo sa XUbuntuom-om preko njegove adrese 192.168.56.101)---userXUbuntu je ime usera sa 
                                                                                                                                  XUbuntu-a
                            password: - upisujemo password koji smo dodelili useru na XUbuntu
                                 
                                 

*Povezivanje u mrežu računara i OS Linux virtuelne mašine iz VirtualBox-a    ---   varijanta 3   ---   NAT (sa podešavanjem porta)

             
        Primer: Manjaro računar i XUbuntu iz VirtualBox-a
 
        1.  U glavnom prozoru VirtualBox-a, izaberati virtualnu mašinu (sa liste) 
        2.  Settings -> Network -> Adapter x -> u padajućem meniju Atached to: izabrati opciju NAT -> Advanced -> Port Forwarding
        3.  Adds new port forwarding rule -> upisati u polja:
     
                  Name: ssh    Protocol: TCP    Host IP:127.0.0.1         Host port: 2200      Guest IP:10.0.2.15         Guest Port: 22
                 (Name: ssh    Protocol: TCP    Host IP:                  Host port: 2222      Guest IP:10.0.2.15         Guest Port: 22)   -> alternativo za 
                                                                                                                                               OpenSuse
        4.  Adds new port forwarding rule -> upisati u polja:
     
                  Name: http   Protocol: TCP    Host IP:127.0.0.1         Host port: 8000      Guest IP:10.0.2.15         Guest Port: 80
                  
        5.  OK -> zatvaramo prozor za pravila 
        6.  Čekiramo polje Cable Connected (na kartici Adapter x)
        7.  OK -> zatvarmo settings virtuelne mašine
        8.  Pokrenuti virtuelnu mašinu (XUbuntu u ovom slučaju)
    
        9.  Povezati računar (realni) sa virtuelnom mašinom (XUbuntu) -> u komandnu liniju upisati:
      
                  ssh -p 2200 XUbuntuuser@localhost                       (ssh -p 2222 opensuseuser@localhost       ->    za opensuse)                                                        
                  password: - upisujemo password koji smo dodelili useru na XUbuntu
    
            (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na XUbuntu-u) 
        
    
        10. Obratno, ako povezujemo virtualnu mašinu iz VB sa realnom mašinom (Manjaro), -> ukomandnu liniju upisati:
    
                  ssh Manjarouser@192.168.yyy.yyy                                       ....adresa na eno1 (ili wlo1 ako je realni računar na wi-fi)
                  password: - upisujemo password koji je dodeljen useru na Manjaro linuxu
    
            (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na Manjaro-u)
        
    Napomena: Bez port forwardinga  mašina iz VB preko NAT opcije pristupa na realnu mašinu i na internet ali ne može sa realne mašine i sa interneta
              da se pristupi mašini u VB      
*Povezivanje u mrežu računara i OS Linux virtuelne mašine iz VirtualBox-a    ---   varijanta 4   ---   Bridged Adapter 

        Primer: MX Linux računar i OpenSuse iz iz VirtualBox-a

        1.  U glavnom prozoru VirtualBox-a, izaberati virtualnu mašinu OpenSuse 
        2.  Settings -> Network -> Adapter x -> u padajućem meniju Atached to: izabrati opciju Bridged Adapter
        3.  OK -> zatvaramo prozor za pravila 
        4.  Čekiramo polje Cable Connected (na kartici Adapter x)
        5.  OK -> zatvarmo settings virtuelne mašine
        6.  Pokrenuti virtuelnu mašinu (OpenSuse  u ovom slučaju)
        7.  Podešavanje firewall-a na OpenSuse:
    
                  firewall-cmd --permanent --add-port=22/tcp                      .....dozvoljavamo komunikaciju preko porta 22
                  firewall-cmd --reload
                  
        8.  Povezati računar LM Linux (realni) sa virtuelnom mašinom (OpenSuse) -> u komandnu liniju upisati:
    
                  ssh bozosus@192.168.0.106                                       .....adresa na enp0s3 adapteru                                     
                  password: - upisujemo password koji smo dodelili useru na Opensuse
    
            (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na OpenSuse-u) 
        
    
        9.  Obratno, ako povezujemo virtualnu mašinu iz VB sa realnom mašinom (MX Linux), -> u komandnu liniju upisati:
    
                  ssh bozobuzejic@192.168.0.116                                       ....adresa na eno1 (ili wlo1 ako je realni računar na wi-fi)
                  password: - upisujemo password koji je dodeljen useru na MX linuxu
    
            (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na MX Linuxu)
        
              
*Povezivanje u mrežu računara i više OS Linux virtuelnih mašina iz VirtualBox-a    ---   varijanta 5   ---   Bridged Adapter 
     
        Primer: Manjaro računar i 4 CentOS 7 mašine (gw - gateway, web - web server, db - data base server, share -share server)
 
        1.  U glavnom prozoru VirtualBox-a, izaberati virtualnu mašinu CentOS 7 (gw iz Lab setup 1 sekcije)
        2.  Settings -> Network -> Adapter x -> u padajućem meniju Atached to: izabrati opciju Bridged Adapter
        3.  OK -> zatvaramo prozor za pravila 
        4.  Čekiramo polje Cable Connected (na kartici Adapter x)
        5.  OK -> zatvarmo settings virtuelne mašine
        6.  Pokrenuti virtuelnu mašinu (CentOS 7 - gw  u ovom slučaju)
        7.  Povezati računar (realni) sa virtuelnom mašinom (CentOS 7  gw) -> u komandnu liniju upisati:
      
                      I   ssh gw_COS7user@192.168.xxx.xxx      .....povezujemo se na gw, adresa na enp0s8 adapteru                                     
                          password:zzzzzzzzzz                       .....upisujemo password koji smo dodelili useru na gwCOS7
    
            (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na gw_COS7) 
        
        7.1.Nakon uspostavljene veze sa gw, moguće je, preko njega, otvaranjem novih prozora terminala Manjara, uspostaviti vezu sa ostalim mašinama iz Lab setup 1
            sekcije:    
                     II   ssh webuser@168.xxx.xxy              
                          password:zzzzzzzzzz                  .....povezujemo se sa web
                    III   ssh dbuser@168.xxx.xyy               
                          password:zzzzzzzzzz                  .....povezujemo se sa db
                     IV   ssh shareuser@168.xxx.xyy               
                          password:zzzzzzzzzz                  .....povezujemo se sa share 
                         
        8.   Obratno, ako povezujemo virtualnu mašinu iz VB sa realnom mašinom (Manjaro), -> u komandnu liniju upisati:
    
                  ssh Manjarouser@192.168.yyy.yyy                                       ....adresa na eno1 (ili wlo1 ako je realni računar na wi-fi)
                  password: - upisujemo password koji je dodeljen useru na Manjaro linuxu
    
             (Ako je sve u redu, komandna linija našeg OS se nakon povezivanja prebacuje u korisnički nalog na Manjaro-u, sve mašine iz Lab setup 1 sekcije takođe
             imaju pristup na komandnu liniju Manjara)
---  
<html>
<body>

<h>Modeli povezivanja u mrežu u VirtualBox-u:</h>

<img src="https://www.nakivo.com/blog/wp-content/uploads/2019/07/VirtualBox-network-settings-%E2%80%93-Comparison-of-VirtualBox-Network-Modes.png" >

</body>
</html>
