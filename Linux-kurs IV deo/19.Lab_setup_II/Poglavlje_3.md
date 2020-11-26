POGLAVLJE 2
 

 *CONFIGURE CLIENT (konfiguracija ssh servisa)


     sudo hostnamectl set-hostname client                                                                 .....menjamo ime hosta
     password: password


     sudo vim /etc/hosts                                                                                  .....podešavanje hosts fajla
             127.0.1.1      client                                                                             .....umesto starog, upisujemo: client
                                                                                                               
             192.168.1.1   router                                                                               
             192.168.2.22  server                                                                              .....dopisujemo navedene hostove
             ...................................     - >    kasnije možemo dodavati još mašina u mrežu 
             192.168.1.102 guest
                                                     esc -> shift: -> wq    -  snimimo fajl


     sudo vim /etc/network/interfaces                                                                     .....konfigurišemo network interfaces 
             (# The primary network interfaces
             auto enp0s3
             iface enp0s3 inet dhcp                                                                            .....ispod ove linije dodati po potrebi (ako ne prepoznaje 
                                                                                                          google host):
             dns-nameservers 8.8.8.8 8.8.4.4                                                                        .....liniju dns sa google ip adresama (8.8.8.8 i 8.8.4.4))
   
             # The internal interface on neta                                                                      .....dodajemo ceo blok
             auto enp0s8
             iface enp0s8 inet static
             address 192.168.1.11
             netmask 255.255.255.0
             network 192.168.1.0
             broadcast 192.168.1.255
                                                    esc -> shift: -> wq    -  snimimo fajl

                                                    
Instaliranje sshd servisa:    

     sudo apt-get install openssh-server                                                                  .....instaliramo sshd servis
     sudo systemctl enable sshd.service                                                                   .....omogućimo servis
     sudo systemctl start sshd.service                                                                    .....startujemo sshd


     sudo vim /etc/network/interfaces                                                                     .....dodatno konfigurišemo network interfaces
             post-up route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1 dev enp0s8                  
             pre-down route del -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1 dev enp0s8                 .....dodajemo dve linije ispod poslednjeg reda u fajlu

                                                    esc -> shift: -> wq    -  snimimo fajl


Izrada ključeva:

     ssh-keygen                                                                                           .....na client mašini    generišemo ključ
     ssh-copy-id -i router                                                                                .....sa client mašine pošaljemo authorized_key na router mašinu
     ssh-copy-id -i server                                                                                .....sa client mašine pošaljemo authorized_key na server mašinu
     ...................................     - >    kasnije možemo dodavati još mašina u mrežu
     ssh-copy-id -i guest

Podešavnje UFW (firewall):

     sudo ufw enable
     sudo ufw allow ssh                                                                                .....omogućimo ssh saobraćaj  
     sudo ufw allow http                                                                               .....omogućimo http saobraćaj
     sudo ufw allow https                                                                              .....omogućimo https saobraćaj
     ili
     sudo ufw allow 'OpenSSH'
     sudo ufw allow 'Apache Full'


Instaliranje web browser-a:

     sudo apt install elinks
     (apt install lynx)

     sudo reboot                                                                                          .....restartujemo mašinu


