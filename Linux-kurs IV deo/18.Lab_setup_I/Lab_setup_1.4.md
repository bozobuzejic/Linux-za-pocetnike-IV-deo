LAB SETUP 1.4 (konfiguracija GW mašine) - DNS, DHCP

   
    Važno:
   
    *  VirtualBox -> File -> Host Network Manager -> vboxnet1 -> DHCP server = odčekirati -> Aplly -> Close   .....odčekiramo dhcp server na VB jer ne smeju da rade 
                                                                                                                  dva u isto vreme na istim mašinama


    1. Instaliranje i podešavanje DNS servisa na GW mašini
   
      1.1 Instaliranje servisa
      
          yum install bind bind-utils                                                 .....instaliramo bind (named.service)
          systemctl enable --now named.service                                        .....omogućimo servis i pokrenemo ga
          
          
      1.2 Podešavanje firewall-a
      
          iptables -vnL --line-number                                                 .....izvršimo proveru upisanih pravila firewall-a
          iptables -I INPUT 7 -m state --state NEW -p tcp --dport 53 -j ACCEPT        
          iptables -I INPUT 8 -m state --state NEW -p udp --dport 53 -j ACCEPT        .....dodamo pravila da gw mašina sluša na portu 53
          iptables-save > /etc/sysconfig/iptables                                     .....snimimo promene
          systemctl reload iptables.service                                           .....restartujemo iptables
          
          
      1.3 Podešavanje konfiguracionog fajla (sekcija options)   
      
          vim /etc/named.conf                                                         .....otvorimo konfiguracioni fajl za named.service
          (vim /etc/bind/named.conf.options)
            options {
                     #listen-on port 53 { 127.0.0.1; };                                    .....zakomentarišemo liniju
                     listen-on port 53 { any; };                                      .....dodamo liniju
                     #listen-on-v6 port 53 { ::1; };                                       .....zakomentarišemo liniju
                     listen-on-v6 port 53 { none; };                                  .....dodamo liniju
                     directory       "/var/named";                                                                  
                     dump-file       "/var/named/data/cache_dump.db";                                                 
                     statistics-file "/var/named/data/named_stats.txt";                                            
                     memstatistics-file "/var/named/data/named_mem_stats.txt";
                     recursing-file  "/var/named/data/named.recursing";
                     secroots-file   "/var/named/data/named.secroots";
                     #allow-query     { localhost; };                                      .....zakomentarišemo liniju
                     allow-query     { localhost; 192.168.60.0/24; };                 .....dodamo liniju
                     #dodato:                                                              .....dodamo i zakomentarišemo liniju
                     forwarders { 147.91.8.6; 147.91.8.62; };                         .....dodamo liniju  (server na ETF)
    alternativno  -> forwarders { 8.8.8.8; 8.8.4.4; };
                     forward only;                                                    .....dodamo liniju
                     };
                                        esc -> shift: -> wq                           .....snimimo izmene u fajlu   
                                        
          named-checkconf                                                             .....izvršimo proveru sintakse          
          systemctl restart named.service                                             .....restartujemo servis    
 
          
      1.4 Podešavanje konfiguracionog fajla - podešavanje zone  
      
          vim /etc/named.conf                                                         .....ponovo otvorimo konfiguracioni fajl za named.service
          (vim /etc/bind/named.conf.options)    
            zone "lab." IN {
                type master;
                file "lab";                    
                allow-update { none; };
            };                                                                        .....unesemo promene u sekciju zone
                                       esc -> shift: -> wq                            .....snimimo izmene u fajlu  
          named-checkconf                                                             .....izvršimo proveru sintakse                                    
            
            
      1.5 Kreiranje fajla koji definiše detalje zone
                     
          cd /var/named                                                             .....uđemo u folder named
          cp named.empty lab                                                        .....preimenujemo fajl named.empty u lab (nema u ubuntu-u)
          chown :named lab                                                          .....promenimo grupu korisnika fajla lab iz root u named
      
          vim lab                                                                   .....otvorimo lab fajl i u njega upišemo (iskopiramo):
          
            $TTL 3H
            $ORIGIN lab.
            lab.       IN SOA  gw.lab. admin.lab. (
                                                    1       ; serial
                                                    1D      ; refresh
                                                    1H      ; retry
                                                    1W      ; expire
                                                    3H )    ; minimum
            lab.        NS      gw.lab.
            gw      A       192.168.60.2
            wb      A       192.168.60.10
            wp.web  CNAME   web
            db      A       192.168.60.11
            share   A       192.168.60.12

                                       esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                                                                   
          named-checkzone lab lab                                                  .....proverimo da li je sve upisano kako treba i dobijemo:
                                                                                        
                 zone lab/IN: loaded serial 1
                 OK
                 
          systemctl restart named.service                                          .....restartujemo servis   
                 
          
     1.6 Podešavanje hosts fajla (na GW, WEB, DB i SHARE mašinama
     
          vim /etc/hosts                                                           .....otvorimo hosts fajl i dodamo:                                                                                    
            192.168.60.2  gw.lab
            192.168.60.10 web.lab
            192.168.60.11 db.lab
            192.168.60.12 share.lab
                                       esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                       
                                                     
    2. Instaliranje i podešavanje DHCP servisa na GW mašini
   
   
      2.1 Instaliranje servisa
      
          yum install dhcp                                                         .....instaliramo dhcp servis
          (sudo apt-get install isc-dhcp-server -y                                      .....ubuntu)
          
      2.2 Podešavanje konfiguracionog fajla      
          
          vim /etc/dhcp/dhcpd.conf                                                 .....otvorimo konfiguracioni fajl za dhcpd.service
            option domain-name-servers 192.168.60.2;
            option domain-search "lab";
            default-lease-time 14400;                                                   .....lease time: 4 sata
            max-lease-time 14400;
            ddns-update-style none;
            authorative;
            log-facility local4;
            subnet 192.168.60.0 netmask 255.255.255.0 {                                 .....definišemo ip adrese cele mreže
            range 192.168.60.101 192.168.60.200;                                        .....definišemo raspon mogućih adresa koje dhcp može da dodeljuje (101-200)
            option routers 192.168.60.2;
            }
            host gw {                                                                        
               hardware ethernet 08:00:27:31:27:c8;                                     .....upisujemo mac adresu adaptera (enp0s3)
               fixed-address 192.168.60.2;                                              .....postavljamo statičku ip adresu
            }
            host web {
               hardware ethernet 08:00:27:db:36:12;                                     .....upisujemo mac adresu adaptera (enp0s3)
               fixed-address 192.168.60.10;                                             .....postavljamo statičku ip adresu
            }
            host db {
               hardware ethernet 08:00:27:ab:dd:be;                                     .....upisujemo mac adresu adaptera (enp0s3)
               fixed-address 192.168.60.11;                                             .....postavljamo statičku ip adresu
            }
            host share  {
               hardware ethernet 08:00:27:6e:0e:67;                                     .....upisujemo mac adresu adaptera (enp0s3)    
               fixed-address 192.168.60.12;                                             .....postavljamo statičku ip adresu
            }
                                       esc -> shift: -> wq                         .....snimimo izmene u fajlu  
                                       
          dhcpd -t -cf /etc/dhcp/dhcpd.conf                                        .....proverimo da li je sve upisano kako treba i dobijemo ispis:
         
            Internet Systems Consortium DHCP Server 4.2.5
            Copyright 2004-2013 Internet Systems Consortium.
            All rights reserved.
            For info, please visit https://www.isc.org/software/dhcp/
            Not searching LDAP since ldap-server, ldap-port and ldap-base-dn were not specified in the config file
            
          systemctl enable --now dhcpd.service                                          .....obavezno restartujemo servis 
          sudo systemctl enable isc-dhcp-server.service                                      na ubuntu
          
      2.3 Provera ispravnosti rada (sa klijentske masine):
          (uraditi na svim mašinama)
          
          ip a
          sudo dhclient -r                                                         .....isključimo sve stare klijentske procese
          ip a
          sudo dhclient                                                            .....ponovo uključimo dhklijente
          ip a
          
          ip route                                                                 .....proverimo da li je rutu dodelio dhcp
          
          cat /var/lib/dhclient/dhclient.leases                                    .....na mrežnom adapteru enp0s3 klijentskih mašina (web, db, share) pojavljuju se 
                                                                                        novododeljene statičke ip adrese 
          
          
          Na web, db, share mašinama (na gw ostaje manual):
          
          nmtui -> Edit a connection -> enp0s3 -> Edit -> IPv4 CONFIGURATION -> automatic -> Ok -> back -> Activate a connection -> enp0s3 -> Deactivate ->
          Activate-> Back -> Ok
         

