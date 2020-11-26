LAB SETUP 1.6 - Instaliranje i podešavanje mail servera 

  
  1   Instaliranje i podešavanje mail servera na GW mašini (podešavanje za ETF server)
  
      systemctl status postfix.service                                                  .....proverimo status postfix servisa
      yum install postfix                                                               .....instaliramo postfix (ako nije instaliran)
       
      vim /etc/postfix/main.cf                                                          .....otvorimo konfiguracioni fajl i otkomentarišemo ili upišemo sledeće:
        myhostname = gw.lab                                                                  .....dodamo liniju
        mydomain = lab                                                                       .....dodamo liniju
        myorigin = $mydomain                                                                 .....otkomentarišemo liniju                                                     
        inet_interfaces = all                                                                .....otkomentarišemo liniju
       #inet_interfaces = localhost                                                               .....zakomentarišemo liniju
        inet_protocols = ipv4                                                                .....na kraj linije umesto all dodajemo ipv4
       #mydestination = $myhostname, localhost.$mydomain, localhost                               .....zakomentarišemo liniju
        mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain               .....otkomentarišemo liniju
        mynetworks = 192.168.60.0/24, 127.0.0.0/8                                            .....otkomentarišemo i upišemo umesto fabrički date mreže
        relayhost = [smtp.etf.bg.ac.rs]:25                                                   .....dodamo liniju kad idemo preko servera na ETF
    
                                                 esc -> shift: -> wq                         .....snimimo izmene u fajlu
                                                 
      systemctl restart postfix.service                                                 .....restartujemo servis
      systemctl status postfix.service                                                  .....proverimo status servisa
   
      iptables -vnL --line-numbers                                                      .....proverimo raspored lanca u iptables
      iptables -I INPUT 10 -p tcp --dport 25 -j ACCEPT                                  .....dodamo dozvolu za port 25 na ulazu
      iptables -I OUTPUT 1 -p tcp --dport 25 -j ACCEPT                                  .....dodamo dozvolu za port 25 na izlazu
      iptables-save > /etc/sysconfig/iptables                                           .....snimimo promene
      systemctl reload iptables.service                                                 .....restartujemo iptables
 
      yum install mailx                                                                 .....instalirati mailx pomoćnu alatku za kreiranje mailova
        
      mail -s "Test poruka" -r bilosta@etf.bg.ac.rs bbuzejic@gmail.com   /enter/        .....napisati mail sa naslovom (-s: subject) "Test poruka" preko ETF servera 
                                                                                             za bbuzejic@gmail.com (staviti neki svoj nalog)
                 Neki tekst poruke                                       /enter/             .....poruka                                                                           
                 .                                                       /enter/             .....kraj poruke i izlaz
                 
      postqueue -p                                                                      .....proverimo šta je sa porukom na serveru (GW mašina) a nakon toga se               
                                                                                             ulogujemo na bbuzejic@gmail.com (neki svoj nalog)
      postqueue -f                                                                      .....komanda kojom obrišemo poruke                                                                                      
                                                                                             
  2   Instaliranje i podešavanje mail servera na WEB, DB i SHARE mašinama
                 
      systemctl status postfix.service                                                  .....proverimo status postfix servisa
      yum install postfix                                                               .....instaliramo postfix (ako nije instaliran)           
                                                                                             
      vim /etc/postfix/main.cf                                                          .....otvorimo konfiguracioni fajl i otkomentarišemo ili upišemo sledeće:
        relayhost = [gw.lab]                                                                 .....dodamo liniju kojom definišemo da pošta ide preko gw.lab
                                            esc -> shift: -> wq                         .....snimimo izmene u fajlu     
                                            
      systemctl restart postfix.service                                                 .....restartujemo servis
      systemctl status postfix.service                                                  .....proverimo status servisa  
      
      mail -s "Test poruka" -r bilosta@etf.bg.ac.rs nekinalog@gmail.com   /enter/       .....napisati mail sa naslovom (-s: subject) "Test poruka" preko ETF servera 
                                                                                             za nekinalog@gmail.com (staviti neki svoj nalog)
                 Neki tekst poruke                                        /enter/            .....poruka                                                                           
                 .                                                        /enter/            .....kraj poruke i izlaz
                 
      postqueue -p                                                                      .....proverimo šta je sa porukom na serveru (GW mašina) a nakon toga se               
                                                                                             ulogujemo na nekinalog@gmail.com (neki svoj nalog) 
                                                                                                                            
                                                                                                                  
                                                                                                                               
  GMAIL SERVER                                                                                                                               
                                                                                                                               
  1.Postfix kao relay host za gmail server (slanje preko autorizovanog relay hosta)
 
    systemctl status postfix.service                                                  .....proverimo status postfix servisa
    yum install postfix                                                               .....instaliramo postfix (ako nije instaliran)
    yum install postfix mailx cyrus-sasl cyrus-sasl-plain                             .....instalirati dodatne pomoćne programe (mailx, cyrus)
 
    vim /etc/postfix/main.cf
       myhostname = gw.lab                                                                  .....dodamo liniju
       mydomain = lab                                                                       .....dodamo liniju
       myorigin = $mydomain                                                                 .....otkomentarišemo liniju                                                     
       inet_interfaces = all                                                                .....otkomentarišemo liniju
      #inet_interfaces = localhost                                                               .....zakomentarišemo liniju
       inet_protocols = ipv4                                                                .....na kraj linije umesto all dodajemo ipv4
      #mydestination = $myhostname, localhost.$mydomain, localhost                               .....zakomentarišemo liniju
       mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain               .....otkomentarišemo liniju
       mynetworks = 192.168.60.0/24, 127.0.0.0/8                                            .....otkomentarišemo i upišemo umesto fabrički date mreže
       relayhost = [smtp.gmail.com]:587                                                     .....podešavanje kad idemo preko gmail servisa
       smtp_use_tls = yes
       smtp_sasl_auth_enable = yes
       smtp_sasl_security_options =
       smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
       smtp_tls_CAfile = /etc/pki/tls/certs/ca-bundle.crt                                        .....na ubuntu: /etc/ssl/certs/ca-certificates.crt
       smtpd_tls_security_level = may                                                       .....dodajemo na kraj dokumenta
       
                                                esc -> shift: -> wq                         .....snimimo izmene u fajlu
 
    *Gmail podržava "App Password": koristiti Google’s 2-Step verification umesto gmail passwod-a. Generisati "App Password": https://myaccount.google.com/
     apppasswords -> kliknuti “Select App” → “Other”, upisati “postfix” i pritisnuti “Generate” -> dobija se password od 16 karaktera koji se upiše u konfiguracioni 
     fajl.  
     
     vim /etc/postfix/sasl_passwd   
        [smtp.gmail.com]:587    nekiaccount@gmail.com:dfrnijgzwejqqwlo                  .....podesimo gmail server da radi sa našim nalogom i dobijenim passwordom od 
                                                                                             16 karaktera
                                                 esc -> shift: -> wq                         .....snimimo izmene u fajlu   
                                                 
     postmap /etc/postfix/sasl_passwd                                                    .....heširamo fajl sa passwordom: ovo kreira /etc/postfix/sasl_passwd.db 
                                                                                             (binarni fajl)
     chmod 600 /etc/postfix/sasl_passwd                                                  .....promenimo permisije sasl_passwd fajlu da mu se ograniči pristup
 
     systemctl restart postfix.service                                                   .....restarujemo postfix servis
    
    
     Firewall za email servis

     iptables -vnL --line-numbers                                                       .....proverimo raspored lanca u iptables
     iptables -I INPUT 10 -p tcp --dport 25 -j ACCEPT                                   .....dodamo dozvolu za port 25 na ulazu
    iptables -I OUTPUT 1 -p tcp --dport 25 -j ACCEPT                                   .....dodamo dozvolu za port 25 na izlazu
    
    iptables-save > /etc/sysconfig/iptables                                            .....snimimo promene
    systemctl reload iptables.service                                                  .....restartujemo iptables
    
    
    PROBA:   mail -s "Test subject" nekinalog@hotmail.com                                .....sa gmail naloga šaljemo mail nekom sa hotmail nalogom
           TEKST PORUKE
           ctrl D                                                                           .....poruka poslata
    
    
    
    
  2.Instaliranje i podešavanje mail servisa na WEB, DB i SHARE mašinama
                 
    systemctl status postfix.service                                                   .....proverimo status postfix servisa
    yum install postfix                                                                .....instaliramo postfix (ako nije instaliran)           
    yum install mailutils                                                                                         
    vim /etc/postfix/main.cf                                                           .....otvorimo konfiguracioni fajl i otkomentarišemo ili upišemo sledeće:
        relayhost = [gw.lab]                                                                .....dodamo liniju kojom definišemo da pošta ide preko gw.lab
                                                 esc -> shift: -> wq                        .....snimimo izmene u fajlu     
                                            
    systemctl restart postfix.service                                              .....restartujemo servis
    systemctl status postfix.service                                               .....proverimo status servisa  
                                                                                         
    mail -s "Test subject" bbuzejic@hotmail.com       /enter/                      .....napisati mail sa naslovom (-s: subject) "Test poruka" preko gmail servera 
                                                                                        za nekinalog@hotmail.com (staviti neki svoj nalog)
        Neki tekst poruke                             /enter/                           .....poruka                                                                           
        ctrl D                                        /enter/                                .....kraj poruke i izlaz
                 
      postqueue -p                                                                   .....proverimo šta je sa porukom na serveru (GW mašina) a nakon toga se               
                                                                                          ulogujemo na nekinalog@gmail.com (neki svoj nalog) 
                                                                                                                                
    
    
  