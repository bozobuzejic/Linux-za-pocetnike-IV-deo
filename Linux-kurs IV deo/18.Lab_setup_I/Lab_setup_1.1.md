LAB SETUP 1.1 (postavljanje mreže od 4 OS (CentOS 7) u VirualBox-u)
     
  a. Konfigurisanje mreže
     
    1. Open VB -> File -> Host Network Manager -> Create -> vboxnet1
    2. ipv4 -> 192.168.60.0/24
    3. DHCP -> 192.168.60.100  
    4. Lower address bound -> 192.168.60.101
    5. Upper address bound -> 192.168.60.120
    
  b. Instaliranje virtuelne mašine - matrice: CentOS-7-x86_64-Minimal-1810.iso
    
    6. New ->  CentOS7 matrica -> Next ---> Memory size: 512MB -> Next ---> Create -> Next ---> Next ---> Create a virtual hard disk now -> Create -> VDI -> 
       Next ---> Dynaimically Allocated -> Next ---> 20GB -> Create 
    7. CentOS7 matrica -> Settings -> Network - Adapter 1 -> Attached to: Host-only Adpter -> Name: vboxnet1 -> OK
    8. Settings -> Storage -> Chose virtual optical disk file -> CentOS-7-x86_64-Minimal-1810.iso -> OK
    9. CentOS7 matrica -> Start -> Install CentOS 7 ---> English - English (United States) -> Continue ---> Date & time: Europe/Belgrade -> Done ---> 
       Network & hostname -> Ethernet (enp0s3) -> ON -> Done ---> Installation destination -> I will configure partitioning -> Done --->
       click here to create them automatically -> /boot 1024 MiB - swap 512 MiB - / (root) 17,99 GiB -> Done -> Accept -> Begin installation --->
       Root Password: password  (confirm)  password  -> Done  /  Done -> instalacija ------------> Rebbot -> root -> password -> poweroff ---->
       
  c. Kloniranje  virtuelne mašine - matrice (full clone) u: GW, Web, DB, Share
    
    10.  10.1.  CentOS7 matrica -> desni klik: Clone -> Name: GW -> Next -> Full clone -> Clone
         10.2.  CentOS7 matrica -> desni klik: Clone -> Name: Web -> Next -> Full clone -> Clone    
         10.3.  CentOS7 matrica -> desni klik: Clone -> Name: DB -> Next -> Full clone -> Clone           
         10.4.  CentOS7 matrica -> desni klik: Clone -> Name: Share -> Next -> Full clone -> Clone  
         
  d. Konfigurisanje mreža na klonovima:  GW, Web, DB, Share
  
    11.  11.1  GW -> Settings -> Network - >  Adapter 2 -> čekirati Enable Network Adapter -> Attached to: NAT -> OK
               GW -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.60.2 -> 
               čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK
    
         11.2  Web -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.60.10 ->                            
               Gateway: 192.168.60.2 -> Dns servers -> Add -> 192.168.60.2 -> čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK
   
         11.3  DB -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.60.11 ->                            
               Gateway: 192.168.60.2 -> Dns servers -> Add -> 192.168.60.2 -> čekirati Automatically connect -> OK ---> back -> Activate a connection -> back -> OK
         
         11.4  SHARE -> Start -> login -> nmtui -> Edit a connection -> enp0s3 -> edit -> Show -> ipv4 configuration ->  Manual -> Adresses -> Add -> 192.168.60.12 
               -> Gateway: 192.168.60.2 -> Dns servers -> Add -> 192.168.60.2 -> čekirati Automatically connect -> OK ---> back -> Activate a connection -> 
               back -> OK
               
               
         Provera:       Na GW:  ping 192.168.60.10         Na Web:  ping 192.168.60.2           Na DB:  ping 192.168.60.2         Na Share:  ping 192.168.60.2 
                                ping 192.168.60.11                  ping 192.168.60.11                  ping 192.168.60.10                   ping 192.168.60.10    
                                ping 192.168.60.12                  ping 192.168.60.12                  ping 192.168.60.12                   ping 192.168.60.11
                                
                                
               