POGLAVLJE 1


Instaliranje virtuelne mašine - base: ubuntu-16.04.1-server-amd64.iso 
    
    1. Open VB New ->  ubuntu-linux-base -> Next ---> Memory size: 512MB -> Next ---> Create -> Next ---> Next ---> Create a virtual hard disk now -> Create -> VDI 
       -> Next ---> Dynaimically Allocated -> Next ---> 20GB -> Create 
    2. ubuntu-linux-base -> Settings -> Network - Adapter 1 -> Attached to: Nat -> OK
    3. Settings -> Storage -> Chose virtual optical disk file -> ubuntu-16.04.1-server-amd64.iso -> OK
    4. ubuntu-linux-base -> Start -> ubuntu-linux-base    --->>>  propratiti instalaciju do kraja  ---->>>  reboot    


Kloniranje  virtuelne mašine - base (full clone) u: router (GW),client, server
    
    1. ubuntu-linux-base -> desni klik: Clone -> Name: ubuntu-linux-router -> Next -> Full clone -> Clone
    2. ubuntu-linux-base -> desni klik: Clone -> Name: ubuntu-linux-client -> Next -> Full clone -> Clone    
    3. ubuntu-linux-base -> desni klik: Clone -> Name: ubuntu-linux-server -> Next -> Full clone -> Clone   
    
    
Konfigurisanje mreža na klonovima (Adapter 1 je već uključen):
    
    1. ubuntu-linux-router -> Settings -> Network - >  Adapter 2 -> čekirati Enable Network Adapter -> Attached to: Internal network -> Name: neta -> 
       Adapter 3 -> čekirati Enable Network Adapter -> Attached to: Internal Network -> Name: netb ->
       Adapter 4 -> čekirati Enable Network Adapter -> Attached to: Bridged Adapter -> Name: eth0 (ili wlan0, zavisno kako je glavni računar povezan na mrežu) -> OK
         
    2. ubuntu-linux-client -> Settings -> Network - >  Adapter 2 -> čekirati Enable Network Adapter -> Attached to: Internal network -> Name: neta -> OK

    3. ubuntu-linux-server -> Settings -> Network - >  Adapter 2 -> čekirati Enable Network Adapter -> Attached to: Internal network -> Name: netb -> OK
       
    Napomena: Prilikom podešavanja routera (GW) umesto Bridged Adaptera (tačka 1, alineja 3) možemo podesiti Port Forwarding na NAT adapteru (Adapter 1) kako
              bi se omogućio protok sa realne mašine (MX) ka virtuelnoj mašini (router - ubuntu):
              
              ubuntu-linux-router -> Settings -> Network - >  Adapter 1 -> u padajućem meniju Atached to: izabrati opciju NAT -> Advanced -> Port Forwarding ->
              Adds new port forwarding rule -> upisati u polja:
                  Name: ssh    Protocol: TCP    Host IP:         Host port: 2200      Guest IP:         Guest Port: 22
                  Name: http   Protocol: TCP    Host IP:         Host port: 8000      Guest IP:         Guest Port: 80
              -> OK
    
              Pristup sa glavnog sistema na router:
    
              ssh -p 2200 bozoubuntu@localhost                                                       .....pristup reko porta 2200
              
---

Rekonfiguracija mreže na klonovima (alternativno, ako se kasnije uvodi dhcp i dns):
     
    1.   Open VB -> File -> Host Network Manager -> Create -> vboxnet2
         ipv4 -> 192.168.1.0/24
         DHCP -> 192.168.1.100  
         Lower address bound -> 192.168.1.101
         Upper address bound -> 192.168.1.120
    
    2.   Router -> Settings -> Network - >  Adapter 1 -> čekirati Enable Network Adapter -> Attached to: Host only adapter -> Name: vboxnet2 -> OK
         Network - >  Adapter 2 -> čekirati Enable Network Adapter -> Attached to: NAT -> OK
         Network - >  Adapter 3 -> čekirati Enable Network Adapter -> Attached to: Bridged adapter -> OK
         Router -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.1.2/24 -> 
         čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK
    3.   Client -> Settings -> Network - >  Adapter 1 -> čekirati Enable Network Adapter -> Attached to: Host only adapter -> Name: vboxnet2 -> OK      
         Client -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.1.11/24 ->  
         Gateway: 192.168.1.2 -> Dns servers -> Add -> 8.8.8.8 -> čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK
    4.   Server -> Settings -> Network - >  Adapter 1 -> čekirati Enable Network Adapter -> Attached to: Host only adapter -> Name: vboxnet2 -> OK      
         Server -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.1.12/24 ->  
         Gateway: 192.168.1.2 -> Dns servers -> Add -> 8.8.8.8 -> čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK     
    5.   sudo vim /etc/network/interfaces     
              sve zakomentarišemo osim: auto lo
                                        iface lo inet loopback
    6.   vim /etc/ufw/before.rules
             *nat
             :PREROUTING ACCEPT [0:0]
             -A PREROUTING -i enp0s8 -d 192.168.0.255  -p tcp --dport 80 -j  DNAT --to-destination 192.168.1.0:80
             -A PREROUTING -i enp0s8 -d 192.168.0.255  -p tcp --dport 443 -j  DNAT --to-destination 192.168.1.0:443
             -A POSTROUTING -s 192.168.1.0/24 ! -d 192.168.1.0/24 -j MASQUERADE
              COMMIT
     7.  sudo vim /etc/sysctl.conf
              net.ipv4.ip_forward = 1
     8.  sudo vim /etc/default/ufw                                                                              .....otvorimo fajl i izmenimo pravilo iz DROP u 
                                                                                                                     ACCEPT
              DEFAULT_FORWARD_POLICY="ACCEPT"                                                                   .....otvorimo fajl i izmenimo pravilo iz DROP u 
                                                                                                                     ACCEPT
    
     9.  Na svim mašinama:
     
         vim .ssh/config                                                      .....sa vim otvaramo novi fajl (koji ćemo posle editovanja snimiti u .ssh folder) i                         
                                                                                   upisujemo
            
                    Host router                                                    .....alias 
                         User bozoubuntu                                           .....navodimo ime naloga 
                         Hostname 192.168.1.2                                 .....navodimo IP adresu 
                         Port 22                                              .....navodimo broj porta 
                         #IdentityFile ~/.ssh/id_rsa                          .....putanju do ključa zakomentarišemo (u slučaju da zatreba)   
            
                    Host client                                   
                         User bozoubuntu                              
                         Hostname 192.168.1.11                 
                         Port 22                               
                         #IdentityFile ~/.ssh/id_rsa            
                         
                    Host server                                   
                         User bozoubuntu                              
                         Hostname 192.168.1.12                 
                         Port 22                               
                         #IdentityFile ~/.ssh/id_rsa      
                    ...................................     - >    kasnije možemo dodavati još mašina u mrežu                                             
                    Host guest                                   
                         User bozosus                              
                         Hostname 192.168.1.102                 
                         Port 22                               
                         #IdentityFile ~/.ssh/id_rsa  

                                        esc -> :wq            .....snimimo tekst i dokument  
                                        
  