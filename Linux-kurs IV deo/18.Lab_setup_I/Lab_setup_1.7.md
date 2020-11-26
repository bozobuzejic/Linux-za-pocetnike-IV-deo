LAB SETUP 1.7 - Podešavanje SELinux-a na GW, WEB,DB i Share mašini

      1. Podešavanje SELinux-a na GW mašini
      
                 vim .ssh/config                                   .....sa vim otvaramo fajl i upisujemo
            
                    Host gw                                     .....alias 
                         User root                              .....navodimo ime naloga 
                         Hostname 192.168.60.2                  .....navodimo IP adresu 
                         Port 22                                .....navodimo broj porta 
                         #IdentityFile ~/.ssh/id_rsa            .....putanju do ključa zakomentarišemo (u slučaju da zatreba)   
            
                    Host web                                   
                         User root                              
                         Hostname 192.168.60.10                 
                         Port 2233                              .....upisano da radi na portu 2233   
                         #IdentityFile ~/.ssh/id_rsa            
                         
                    Host db                                   
                         User root                              
                         Hostname 192.168.60.11                 
                         Port 2233                              .....upisano da radi na portu 2233 
                         #IdentityFile ~/.ssh/id_rsa      
                                                                
                    Host share                                   
                         User root                              
                         Hostname 192.168.60.12                 
                         Port 2233                              .....upisano da radi na portu 2233   
                         #IdentityFile ~/.ssh/id_rsa  

                                           esc -> :wq                             .....snimimo tekst i dokument 
                                           
                 systemctl reload sshd.service                          
                                           
                                           
                 semanage port -l                                    .....prikazuje sve portove u sistemu (proverimo koje sve portove selinux fabrički dopušta)
                 semanage port -l | grep ssh                         .....prikazuje samo port koji selinux dozvoljava za ssh
                     ssh_port_t                     tcp      22           .....(ssh sluša na tcp portu 22)
                     
                 semanage port -a -t ssh_port_t -p tcp 2233          .....komanda kojom selinux prihvata sa ssh port bude i 2233 
                     ssh_port_t                     tcp      2233, 22     .....selinux omogućuje i slušanje ssh na portu 2233 pored porta 22
                                   
                 reboot                                                        .....restartovati mašinu                            
                                           
                                           
      2. Podešavanje SELinux-a na WEB, DB i SHARE mašina (dati postupak se realizuje na svakoj mašini) 
      
                vim /etc/selinux/config                              .....sa vim editorom otvorimi konfiguracioni fajli (da bi onemogućili selinux) upisujemo:
                          SELINUX=enforcing                               .....otkomentarišemo i aktiviramo selinux 
                          #SELINUX=permissing                             .....zakomentarišemo (stavimo znak ispred #) 
                                
                                        esc -> :wq                             .....snimimo i izađemo iz fajla
                                           
                                           
                iptables -vnL --line-numbers                                                .....proveriti rene brojeve već postavljenih pravila
                iptables -I INPUT 10 -m state --state NEW -p tcp --dport 2233 -j ACCEPT     .....lanac koji dopušta rad na portu 2233
                iptables-save > /etc/sysconfig/iptables                                      .....snimimo promenu
                systemctl restart iptables.service                                           .....restartujemo servis
                
                vim /etc/ssh/sshd_config                            .....otvorimo sshd_config fajl i upišemo ispod linije #Port 22 Port 2233:
                          #Port 22
                          Port 2233                                     
                                
                                        esc -> :wq                                .....snimimo i izađemo iz fajla   
                                       
                                       
                vim .ssh/config                                    .....sa vim otvaramo fajl i upisujemo
            
                    Host gw                                             .....alias 
                         User root                                      .....navodimo ime naloga 
                         Hostname 192.168.60.2                          .....navodimo IP adresu 
                         Port 22                                        .....navodimo broj porta 
                         #IdentityFile ~/.ssh/id_rsa                    .....putanju do ključa zakomentarišemo (u slučaju da zatreba)   
            
                    Host web                                   
                         User root                              
                         Hostname 192.168.60.10                 
                         Port 2233                                           .....upisano da radi na portu 2233
                         #IdentityFile ~/.ssh/id_rsa            
                         
                    Host db                                   
                         User root                              
                         Hostname 192.168.60.11                 
                         Port 2233                                           .....upisano da radi na portu 2233
                         #IdentityFile ~/.ssh/id_rsa      
                                                                
                    Host share                                   
                         User root                              
                         Hostname 192.168.60.12                 
                         Port 2233                                           .....upisano da radi na portu 2233
                         #IdentityFile ~/.ssh/id_rsa  

                                        esc -> :wq                                .....snimimo tekst i dokument    
                                                                
                                    
                yum provides /usr/sbin/semanage                                     .....proverimo gde možemo pronaći semanage alatku              
                yum install policycoreutils-python-2.5-29.el7_6.1.x86_64            .....instaliramo samanage alatku za podešavanje portova u selinux-u  
                
                semanage port -a -t ssh_port_t -p tcp 2233                          .....komanda kojom selinux prihvata sa ssh port bude i 2233 
                     ssh_port_t                     tcp      2233, 22                    .....selinux omogućuje i slušanje ssh na portu 2233 pored porta 22
                          
   
                reboot                                                        .....restartovati mašinu     
                     
 
 
 