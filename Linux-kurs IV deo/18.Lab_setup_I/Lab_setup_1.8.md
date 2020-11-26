LAB SETUP 1.8 - Podešavanje OpenVPN na GW, WEB,DB i SHARE mašini 

                
 *OpenVPN  
 
    Na OpenVPN serveru = GW mašina
 
       yum install openvpn easy-rsa -y                                                .....instaliramo openvpn i easy-rsa (EasyRSA je alat za kreiranje 
                                                                                           infrastrukture javnih ključeva (PKI) za OpenVPN)
                
       mkdir /etc/openvpn/easy-rsa                                                    .....pravimo direktorijum za ključeve
       cp /usr/share/easy-rsa/3.0.3/* /etc/openvpn/easy-rsa/ -r                       .....u kreirani folder kopiramo sve fajlove iz navedenog foldera (/3.0.3/)
       
       /etc/openvpn/easy-rsa/openssl-1.0.cnf                                          .....konfiguraconi fajl za ključeve
       
       cd /etc/openvpn/easy-rsa/                                                      .....premestimo se u kreirani direktorijum /easy-rsa/ da bi smo pokrenuli 
                                                                                           inicijalizaciju
       
    Inicijalizujemo CA:
      
       ./easyrsa init-pki                                                             .....inicijalizuje okruženje i gradi strukturu fajla za klijenta PKI
       
            init-pki complete; you may now create a CA or requests. 
            Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki                       .....kreira direktorijum gde će biti smešteni ključevi

       ./easyrsa build-ca nopass                                                      .....generiše privatni ključ klijenta i njegov zahtev za potpisivanje 
                                                                                           sertifikata CSR  
            .....
            writing new private key to '/etc/openvpn/easy-rsa/pki/private/ca.key.AyvDPzLbBa'
            .....
            Common Name (eg: your user, host, or server name) [Easy-RSA CA]:Moj_server     .....klijentu dajemo ime: Moj_server 
            .....
            Your new CA certificate file for publishing is at:
            /etc/openvpn/easy-rsa/pki/ca.crt                                               .....ključevi se smeštaju u odgovarajući dirktorijum /pki/
            
        ./easyrsa gen-dh                                                              .....generiršemo Diffie-Hellman ključ koji omogućuje sigurnu razmenu          
                                                                                           kriptografskih ključeva između servera i klijenata
                                                                                           
        ./easyrsa build-server-full Moj_server nopass                                 .....generišemo sertifikat za sam server Moj_server
        
                                                                                           
        ln -s /etc/openvpn/easy-rsa/pki/ /etc/openvpn/server/keys                     .....kreiramo link ka sadržaju foldera /pki/ sa ključevima i smeštamo ga u 
                                                                                           folder /etc/openvpn/server/    
                                                                                           
        mkdir /etc/openvpn/server/ccd                                                 .....kreiramo folder ccd u folderu /etc/openvpn/server/ (u ccd folder stavljamo
                                                                                           fajlove koje kreiramo za svakog klijenta kome hoćemo da dodelimo statičku
                                                                                           ip adresu) - client-config-dir
        
    Pravljenje sertifikata za klijente (na GW mašini)
    
        cd /etc/openvpn/easy-rsa/                                                     .....nalazimo se u folderu iz koga je moguće generisati ključeve
    
    WEB
        ./easyrsa build-client-full webclient                                         .....kreiramo sertifikacioni ključ za klijenta webclient
            
            .....
            The Subject's Distinguished Name is as follows
            commonName            :ASN.1 12:'webclient'
            Certificate is to be certified until Jun 18 20:58:40 2029 GMT (3650 days)
            .....

    DB        
       ./easyrsa build-client-full dbclient                                           .....kreiramo sertifikacioni ključ za klijenta dbclient
            
            .....
            The Subject's Distinguished Name is as follows
            commonName            :ASN.1 12:'dbclient'
            Certificate is to be certified until Jun 19 16:56:09 2029 GMT (3650 days)
            .....
            
    SHARE        
       ./easyrsa build-client-full shareclient                                         .....kreiramo sertifikacioni ključ za klijenta shareclient
            
            .....
            The Subject's Distinguished Name is as follows
            commonName            :ASN.1 12:'shareclient'
            Certificate is to be certified until Jun 19 16:59:00 2029 GMT (3650 days)
            .....        
     
    GW
        ./easyrsa build-client-full gwclient                                           .....kreiramo sertifikacioni ključ za klijenta gwclient (klijent kreiran na                
                                                                                            samom serveru) 
            .....
            The Subject's Distinguished Name is as follows
            commonName            :ASN.1 12:'gwclient'
            Certificate is to be certified until Jun 20 07:09:13 2029 GMT (3650 days)
            ..... 
     
     
    Podesavanje servera:
     
        /usr/share/doc/openvpn-2.4.7/sample/sample-config-files/server.conf           .....primer konfiguracionog fajla za openvpn server (fajl možemo iskopirati
        (/usr/share/doc/openvpn/examples/sample-config-files/.....)                        u /etc/openvpn/server/ folder i editovati ga kao svoj)
        
        ili:
        
        vim /etc/openvpn/server/Moj_server.conf                                       .....kreiramo (novi) konfiguracioni fajl za Moj_server              
            port 1194
            proto tcp
            dev tun
            ca keys/ca.crt
            cert keys/issued/Moj_server.crt
            key keys/private/Moj_server.key  # This file should be kept secret
            dh keys/dh.pem
            server 172.31.1.0 255.255.255.0
            ifconfig-pool-persist ipp.txt
            #push "route 192.168.60.0 255.255.255.0"
            push "register-dns"
            topology subnet
            push "topology subnet"
            client-config-dir ccd
            keepalive 10 120
            comp-lzo
            user nobody
            group nobody
            persist-key
            persist-tun
            status /var/log/openvpn-status.log
            log-append  /var/log/openvpn.log
            verb 4
             # sledece linije idu ako pravimo za unix korisnike na sistemu:
             # plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
             # client-cert-not-required
             # username-as-common-name
             # za njih pravimo i fajl za openvpn u pam sistemu:
             #%PAM-1.0
              #auth            include         system-auth
              #account         include         system-auth
              #password        include         system-auth
              #session         include         system-auth
              
                                                  esc -> :wq                          .....snimimo tekst i dokument
                                                  
    Alternativno (dodela statičke ip adrese klijentu):
     
        vim /etc/openvpn/server/ccd/imeklijenta                                       .....u direktorijumu /ccd/ kreiramo fajl po nazivu klijenta (imeklijenta)
            ifconfig-push 172.31.1.2 255.255.255.0                                         .....i klijentu imeklijenta dodeljujemo statičku ip adresu
                                                  esc -> :wq                          .....snimimo tekst i dokument   
                                                  
    Upravljanje servisom:
     
        systemctl enable --now openvpn-server@Moj_server.service                      .....omogućimo Moj_server openvpn server
        systemctl status openvpn-server@Moj_server.service                            .....proverimo status Moj_server openvpn servera
     
        iptables -vnL --line-numbers -t nat
        iptables -t nat -A POSTROUTING -s 172.31.1.0/24 -o tun0 -j MASQUERADE
        iptables -vnL --line-numbers
        iptables -A FORWARD -i tun0 -d 172.31.1.1/24 -j ACCEPT
        iptables-save > /etc/sysconfig/iptables
        systemctl restart iptables.service
        
     Podešavanje klijenta (na WEB, DB, SHARE i GW  mašini)
     
        yum install openvpn
   
        /usr/share/doc/openvpn-2.4.7/sample/sample-config-files/client.conf           .....primer konfiguracionog fajla za klijenta (možemo ga iskopirati u 
                                                                                           /etc/openvpn/ i editovati kao svoj)
                                                                                                  
     WEB     vim /etc/openvpn/client/webclient.conf                                               .....kreiramo konfiguracioni fajl klijenta webclient i upisujemo:
     DB      vim /etc/openvpn/client/dbclient.conf                                                .....kreiramo konfiguracioni fajl klijenta dbclient i upisujemo:
     SHARE   vim /etc/openvpn/client/shareclient.conf                                             .....kreiramo konfiguracioni fajl klijenta shareclient i upisujemo:
     GW      vim /etc/openvpn/client/gwclient.conf                                                .....kreiramo konfiguracioni fajl klijenta gwclient i upisujemo:

           client
           dev tun
           proto tcp
           remote 192.168.60.2 1194
           resolv-retry infinite
           nobind
           persist-key
           persist-tun
           ca /etc/openvpn/keys/ca.crt
           # sledece 2 linije za auth preko kljuceva
           cert /etc/openvpn/keys/webclient.crt        ...      /dbclient.crt   ...      /shareclient.crt   ...     /gwclient.crt
           key /etc/openvpn/keys/webclient.key         ...      /dbclient.key   ...      /shareclient.key   ...     /gwclient.key 
           # ako se pak radi sa unix nalozima
           # auth-user-pass
           comp-lzo
           verb 2

        
                                                    esc -> :wq                          .....snimimo tekst i dokument

       systemctl enable openvpn-client@webclient.service                                .....omogucimo na svim mašinama


                                      
WEB mašina                                         
                                                    
        mkdir /etc/openvpn/keys/                                                        .....kreiramo direktorijum za ključeve 
        
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/keys/       .....kopiramo ca.cert fajl sa openvpn servera (GW mašine) u direktorijum 
                                                                                             /keys/
          root@192.168.60.2's password: password                                             
          ca.crt                                                                                       .....fajl iskopiran
          
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/private/webclient.key /etc/openvpn/keys/  .....kopiramo webclient.key fajl sa openvpn servera u direktorijum 
          root@192.168.60.2's password: password                                                       /keys/
          webclient.key                                                                                 .....fajl iskopiran
                    
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/issued/webclient.crt /etc/openvpn/keys/   .....kopiramo webclient.crt fajl sa openvpn servera u direktorijum 
          root@192.168.60.2's password: password                                                       /keys/
          webclient.crt                                                                                .....fajl iskopiran
          
          
DB mašina                                         
                                                    
        mkdir /etc/openvpn/keys/                                                        .....kreiramo direktorijum za ključeve 
        
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/keys/       .....kopiramo ca.cert fajl sa openvpn servera (GW mašine) u direktorijum 
                                                                                             /keys/
          root@192.168.60.2's password: password                                             
          ca.crt                                                                                      .....fajl iskopiran
          
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/private/dbclient.key /etc/openvpn/keys/  .....kopiramo dbclient.key fajl sa openvpn servera u direktorijum 
          root@192.168.60.2's password: password                                                      /keys/
          dbclient.key                                                                                ....fajl iskopiran
                    
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/issued/dbclient.crt /etc/openvpn/keys/   .....kopiramo dbclient.crt fajl sa openvpn servera u direktorijum 
          root@192.168.60.2's password: password                                                      /keys/
          dbclient.crt                                                                                .....fajl iskopiran
          
  
SHARE mašina                                         
                                                    
        mkdir /etc/openvpn/keys/                                                        .....kreiramo direktorijum za ključeve 
        
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/keys/       .....kopiramo ca.cert fajl sa openvpn servera (GW mašine) u direktorijum 
                                                                                             /keys/
          root@192.168.60.2's password: password                                             
          ca.crt                                                                                           .....fajl iskopiran
          
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/private/shareclient.key /etc/openvpn/keys/    .....kopiramo shareclient.key fajl sa openvpn servera u 
          root@192.168.60.2's password: password                                                           direktorijum /keys/
          shareclient.key                                                                                  ....fajl iskopiran
                    
        scp root@192.168.60.2:/etc/openvpn/easy-rsa/pki/issued/shareclient.crt /etc/openvpn/keys/     .....kopiramo dbclient.crt fajl sa openvpn servera u            
                                                                                                           direktorijum /keys/
          root@192.168.60.2's password: password                                                           
          shareclient.crt                                                                                  .....fajl iskopiran
       
       
GW mašina                                         
                                                    
        mkdir /etc/openvpn/keys/                                                .....kreiramo direktorijum za ključeve 
        
        cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/keys/                  .....kopiramo ca.cert fajl sa openvpn servera (GW mašine) u direktorijum /keys/
          
        cp /etc/openvpn/easy-rsa/pki/private/gwclient.key /etc/openvpn/keys/    .....kopiramo gwclient.key fajl sa openvpn servera u direktorijum /keys/ klijenta
                    
        cp /etc/openvpn/easy-rsa/pki/issued/gwclient.crt /etc/openvpn/keys/     .....kopiramo gwclient.crt fajl sa openvpn servera u direktorijum /keys/ klijenta
         
       
Iptables na svim mašinama:  
  
        iptables -vnL --line-numbers
        iptables -I INPUT 12 -m state --state NEW -p tcp --dport 1194 -j ACCEPT
        iptables-save > /etc/sysconfig/iptables
        systemctl restart iptables.service  


Pokretanje I

       systemctl start openvpn-client@webclient                                         .....pokrecemo vpn klijente na svim masinama
       systemctl start openvpn-client@dbclient
       systemctl start openvpn-client@shareclient    
       systemctl start openvpn-client@gwclient  

        
Pokretanje II  
    
        openvpn --config /etc/openvpn/client/webclient.conf                                       .....starujemo klijenta na terminalu same mašine
        openvpn --config /etc/openvpn/client/dbclient.conf                                        .....starujemo klijenta na terminalu same mašine
        openvpn --config /etc/openvpn/client/shareclient.conf                                     .....starujemo klijenta na terminalu same mašine 
        openvpn --config /etc/openvpn/client/gwclient.conf                                        .....starujemo klijenta na terminalu same mašine  
        
        Primer na MX linux-u
        openvpn --config /etc/openvpn/client/mxclient.conf                                        .....startujemo klijenta na mx mašini
        ctrl c                                                                                    .....zauzstavimo klijenta na mx mašini
        
        TEST:
       -Otvoriti 4 terminala na računaru-domaćinu i povezati svaki sa odgovarajućom mašinom iz VirtualBoxa (ulogovati se na mašine GW, WEB, DB i SHARE)
        Sa komandom ip a pročitati dodeljene ip adrese na adapterima tun0 (172.31.1.x)  WEB, DB i SHARE mašina a na GW pročitati tun1 
        (tun0 je rezervisan za sam server na GW)
        Pingovati medjusobno sve mašine (svaku sa svakom).

       -ssh root@172.31.1.1                                                                 .....uz pomoc ssh se prebacujemo na vpn server
           root@localhost ~]# ip a                                                               .....proverimo na gw masini u koju smo se prebacili

Zaustavljanje:
       
        systemctl stop openvpn-client@webclient.service
        systemctl stop openvpn-client@dbclient.service
        systemctl stop openvpn-client@sharelient.service
        systemctl stop openvpn-client@gwclient.service
