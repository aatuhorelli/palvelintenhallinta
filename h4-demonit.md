# H4 - Demonit

Tehtävänannot luettavissa https://terokarvinen.com/2023/configuration-management-2023-autumn/

## X) Tiivistelmät

### Karvinen 2023: Salt Vagrant - automatically provision one master and two slaves

 - Modulit kirjoitetaan tiedostoon /srv/salt/modulin_nimi/init.sls
   - Syntaksi: YAML. Sisennys 2 välilyöntiä.
   - ``$ sudo salt '*' state.appy modulin_nimi``
 - /srv/salt/top.sls -tiedostoon voidaan määritellä, mitä moduleita kullekin orjalle ajetaan automaattisesti komennolla ``$ sudo salt '*' state.appy``

   ````
   $ cat /srv/salt/top.sls
   base:
     '*':
       - hello
   ````
Lähde: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file

### Salt contributors: Salt overview

 - Salt käyttää YAML-renderoijaa YAML-muotoisten tekstitiedostojen kääntämiseksi Pythonille sopivaan muotoon
 - YAMLin perussääntöjä:
   - Tiedot järjestetään avain: arvo -pareiksi
     - avain, kaksoispiste, väliyönti ja arvo. Esim: vegetables: peas
   - Kirjankoko on merkitsevä
   - Sisennys välilyönneillä, TAB käyttö ei sallittua
     - Sisennykset määrittelevät kontekstin
   - Kommentointi risuaidalla (#)

 - YAMLin kolme peruselementtiä:
   - Scalars (avain: arvo). Arvo voi olla numero, merkkijono tai boolean.
   - Lists:
     - ````   
       avain:
         - arvo1 # sisennys (kaksi välilyöntiä), viiva, välilyönti, arvo
         - arvo2
       ````
    - Sanakirja / avain-arvotaulukko:
      - ````
        #esimerkki

        dinner:
          appetizer: shrimp soup
          drink: sparkling water
          entree:
            - steak
            - mashed potatoes
            - dinner roll
          dessert:
            - chocolate cake
        ````

Lähde: https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml

### Salt contributors: Salt states

 - Tilojen luominen module.function kautta
 - Tilan tarkistus kutsumalla state.sls 

````
# Tila-SLS:n datan rakenne

# identifier: # tunnisteen määrittäminen
#   module.function: # tila ja funktio, esim pkg.installed
#     - name: name_value # tilan kutsun nimi, usein sama kuin muokattavan tiedoston tai asennettan paketin nimi
#     - function_arg: arg_value # hyväksytyt argumentit

# Esimerkki

install_tree_now:
  pkg.installed:
    - pkgs:
    - tree
````

 - Tilat tulee kirjoittaa helposti ymmärrettävään muotoon
 - Luettavuuden ja mahdollisten ongelmien rajaamiseksi tiloja ei kannata niputtaa liikaa yhteen

````
# Esimerkki rakenteesta

/srv/salt
├── core.sls
├── httpd
│ ├── files
│ │ ├── apache2.conf
│ │ └── httpd.conf
│ └── init.sls
├── dns
│ ├── files
│ │ ├── bind.conf
│ │ └── named.conf
│ └── init.sls
├── ntp
│ ├── files
│ │ └── ntp.conf
│ ├── init.sls
│ ├── ntp-client.sls
│ └── ntp-server.sls
├── redis
│ ├── files
│ │ └── redis.conf
│ ├── init.sls
│ └── map.jinja
├── ssh
│ ├── files
│ │ ├── ssh_config
│ │ └── sshd_config
│ ├── init.sls
│ └── map.jinja
└── top.sls
````

 - Satojen tai tuhansien orjien määrittely käsin on työlästä
 - top.sls-tiedostolla voidaan osoittaa tietyt tilat suoritettavaksi eri muuttujien perusteella

````
# Esimerkki

base: 
'*': # suoritetaan kaikille
  - core
'^(app|web).(qa|prod).loc$':
  - match: pcre
  - httpd
  - nagios.web
'os:Ubuntu': # suoritetaan, jos käyttöjärjestelmä Ubuntu
  - match: grain
  - repos.ubuntu
'os_family:RedHat':
  - match: grain
  - repos.epel
'nagios* or G@role:monitoring':
  - match: compound
  - nagios.server

````

SSH-tilan määrittely:

````
install_openssh:
  pkg.installed:
    - name: openssh

push_ssh_conf:
  file.managed:
    - name: /etc/ssh/ssh_config
    - source: salt://ssh/ssh_config

push_sshd_conf:
  file.managed:
    - name: /etc/ssh/sshd_config
    - source: salt://ssh/sshd_config

start_sshd:
  service.running:
    - name: sshd
    - enable: True
````

Lähde: https://docs.saltproject.io/salt/user-guide/en/latest/topics/states.html#state-modules

### Karvinen 2018: Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port

SSH:n portin vaihtamisen vaiheet:

 - Luo SSH-moduli

 - ````
   $ cat /srv/salt/sshd.sls
   openssh-server:
     pkg.installed
     /etc/ssh/sshd_config:
    file.managed:
      - source: salt://sshd_config
   sshd:
    service.running:
      - watch:
        - file: /etc/ssh/sshd_config
   ````

- Luo sshd_config-tiedosto /srv/salt/sshd_config (käytä oman käyttöjärjestelmän sshd_configia pohjana, poista kommentit ja vaihda portin numero)
- Aja tila orjille ``$ sudo salt '*' state.apply sshd``
- Testaa toiminta ``$ nc -vz tero.example.com 8888`` tai `` $ ssh -p 8888 tero@tero.example.com``

Lähde: https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh

## A) Hello SLS!

Tehtävien orjat pyörivät paikallisella koneella Debian 12 -käyttöjärjestelmässä VirtualBoxilla. Herra on vuokratulla palvelimella (Linode Nanode).

Aloitin potkaisemalla aiemmin [määrittelemäni](https://github.com/aatuhorelli/palvelintenhallinta/blob/main/h2-karjaa.md) Debian 11 -orjat ajamalla komennon ``$ vagrant up`` hakemistossa /home/aatu/Vagrant/nettiorja. Otin toisessa terminaalissa samalla ssh-yhteyden herraa pyörittävään palvelimeen komennolla ``$ ssh [ip-osoite]``. Päästyäni herrakoneelle loin hakemiston ``$ mkdir /srv/salt/hello``-komennolla hakemiston, johon tilan init.sls oli tarkoitus luoda.

Siirryin luomaani hakemistoon ``$ cd /srv/salt/hello`` ja tarkistin orjien olevan linjoilla ``$ sudo salt '*' test.ping``. Molemmat orjat vastasivat True, eli tilan ajaminen pitäisi sen perusteella myöhemmin onnistua. Testasin toiselle orjista hello-tiedoston luomista state.singlen avulla.

![Add file: t002 hello](/img/ping_t002_hello.png)
> Ajettu komento: Orjalta t002 tulee löytyä tiedosto /tmp/hello, jonka sisältö on "Hello world!"  

Tarkistin vielä tiedoston sisällön:
    
    aatu@localhost:/srv/salt/hello$ sudo salt 't002' state.single cmd.run "cat /tmp/hello"
    t002:
    ----------
    ...
        stdout:
                  Hello world! # catilla luettu sisältö. Näyttäisi olevan oikein.
    ...
    

Loin microlla tiedoston init.sls, johon käänsin aiemmin orjalle ajetun komennon YAML-muotoon. Tallensin(ctrl + s) tiedoston ja suljin sen(ctrl + q). 
    
    aatu@localhost:/srv/salt/hello$ micro init.sls 
    aatu@localhost:/srv/salt/hello$ cat init.sls 
    /tmp/hello: # Tiedoston polku ja nimi
      file.managed: # tila, tiedoston tulee löytyä
        - contents: Hello world! # tiedoston sisältö

Seuraavaksi ajoin tilan orjille. Odotusarvo on, että molemmat onnistuvat, mutta t002:lta tiedosto jo löytyy oikealla sisällöllä, joten siihen ei tehdä muutoksia (Succeeded: 1). T001:llä yhteenvedossa tulisi lukea Succeeded: 1, (changed=1), sillä tiedostoa ei vielä ole olemassa.

![Add file: state hello](/img/state_hello.png)
> Ajettu komento ja orjien vastaukset

Vastaukset vastasivat odotuksia. Varmistin vielä tiedoston sisällön suoraan virtuaalikoneelle ssh:lla kirjautumalla:

````
$ vagrant ssh t001
...
vagrant@t001:~$ cat /tmp/hello
Hello world!
````

## B) Top

Kirjoitin herran /srv/salt/top.sls-tiedostoon seuraavanlaisen sisällön:

````
base:
  '*': # koskee kaikkia orjia
     - hello # ajettavan tilan nimi
````

Poistin aiemman hello-tiedoston orjilta komennolla ``$ sudo salt '*' state.single file.absent /tmp/hello # Comment: File /tmp/hello is not present``. Olin käynnistänyt tietokoneeni välissä uudestaan, joten tmp-hakemisto oli tyhjennetty, mutta tulipa ainakin varmistettua. 

Tämän jälkeen ajoin top.sls:n mukaiset tilat orjille komennolla ``$ sudo salt '*' state.apply``. Hello-modulia ei tässä yhteydessä tarvinnut mainita, koska top.sls-tiedostoon on määritelty mitä tiloja kullekin orjalle tulee ajaa. 

````
t002:
----------
          ID: /tmp/hello # tiedoston nimi
    Function: file.managed # tilafunktio
      Result: True 
     Comment: File /tmp/hello updated # Tehty muutoksia -> tiedosto päivitetty
     Started: 11:00:13.846069 
    Duration: 13.236 ms
     Changes:   
              ----------
              diff:
                  New file # /tmp/hello -tietosto luotu

Summary for t002
------------
Succeeded: 1 (changed=1) # Onnistunut suoritus. Tiedosto luotu.
Failed:    0
------------

[...]

Summary for t001
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
````
Molemmat orjat vastasivat onnistuneensa tehtävässä (Succeeded: 1) luomalla uuden tiedoston (changed=1). Ajamalla sama komento uudestaan, molempien orjien vastauksessa oli kommenttina rivi "File /tmp/hello is in the correct state", eli tiedosto löytyi ja sisältö oli oikea.

## C) Apache

Varmistin ensin, että orjilla ei ole apache2 asennettuna komennolla ``$ sudo '*' state.single pkg.removed apache2``. Molemmat orjat vastasivat "Comment: All targeted packages were removed", eli Apache2 oli valmiiksi asennettuna, mutta poistettiin komennon ajamisen yhteydessä. Varmistin myös, että curl on asennettu, sillä ajattelin testata index.html:n sisällön lopuksi sillä: ``$ sudo '*' state.single pkg.installed curl``. 

Apachen asennuksessa ja testisivun korvaamisen vaiheet:
 - Apachen asennus (pkg.installed apache2)
 - Testisivun korvaus (file.managed /var/www/html/index.html contents="Testisivu")
 - Tarkistetaan, että demoni on käynnissä (service.running apache2-service)
 - Tarkistetaan index.html:n sisältö (cmd.run "curl localhost")

Testasin näitä komentoja t001:lle. 

    $ sudo salt 't001' state.single pkg.installed apache2 # Succeeded 1, changed 1. Paketti asennettu
    $ sudo salt 't001' state.single file.managed /var/www/html/index.html contents="Testisivu" # tiedosto muokattu. Olin näemmä määritellyt jo aiemmin index.html:lle normaalista poikkeavan sisällön, joka nyt muutettiin.
    $ sudo salt 't001' state.single service.running apache2 # Comment: The service apache2 is already running. Demoni käynnissä.
    $ sudo salt 't001' state.single cmd.run "curl localhost" # stdout: Testisivu

    # t002 testaus. Apache2 ei asennettu, joten curl localhost pitäisi palauttaa virheilmoitus
    
    $ sudo salt 't002' state.single cmd.run "curl localhost" # stderr: curl: (7) Failed to connect to localhost port 80: Connection refused

Komentojen ajo onnistui ja curl localhost palautti oikeanlaisen sisällön, joten päättelin voivani kirjoitella näistä oman modulin /srv/salt/apache/init.sls. Tiedoston sisältö ja tilan ajaminen t001:lle.

![Add file: feil](/img/apache2_fail.png)
> /srv/salt/apache2/init.sls -tiedoston sisältö ja epäonnistunut ajo

Tilan ajaminen epäonnistui, joten tarkistin vielä init.sls:n kirjoitusasun. Löysin ylimääräisen välilyönnin riviltä 4, jonka poistamisen jälkeen ``$ sudo salt 't001' state.apply apache2`` onnistui.

````
t001:
----------
          ID: apache2
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed # valmiiksi asennettu
     Started: 11:29:00.050822
    Duration: 64.463 ms
     Changes:   
----------
          ID: /var/www/html/index.html
    Function: file.managed
      Result: True
     Comment: File /var/www/html/index.html is in the correct state # tiedoston sisältö oikea
     Started: 11:29:00.117498
    Duration: 10.45 ms
     Changes:   
----------
          ID: apache2-service
    Function: service.running
        Name: apache2
      Result: True
     Comment: The service apache2 is already running # palvelu käynnissä
     Started: 11:29:00.128453
    Duration: 25.976 ms
     Changes:   
----------
          ID: curlaus
    Function: cmd.run
        Name: curl localhost
      Result: True
     Comment: Command "curl localhost" run
     Started: 11:29:00.155917
    Duration: 12.673 ms
     Changes:   
              ----------
              pid:
                  2163
              retcode:
                  0
              stderr:
                    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                                   Dload  Upload   Total   Spent    Left  Speed
                  
                    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
                  100    10  100    10    0     0   5000      0 --:--:-- --:--:-- --:--:--  5000
              stdout:
                  Testisivu # palvelin palauttaa curlilla oikeanlaisen sivun

Summary for t001
------------
Succeeded: 4 (changed=1)
Failed:    0
------------
Total states run:     4
Total run time: 113.562 ms

````

Kaikki vaiheet onnistuivat, eikä muutoksia tehty, koska olin asentanut kaikki jo käsin t001:lle. Ajoin vielä saman komennon t002:lle, jolla Apachea ei vielä ollut.

````
t002:
...
     Comment: The following packages were installed/updated: apache2
...
     Comment: File /var/www/html/index.html updated
...
     Comment: The service apache2 is already running
...
              stdout:
                  Testisivu
...
Succeeded: 4 (changed=3) # Apache2 oli jo käynnissä, joten muutoksia ei tehty. 3 muuta muutosta.

````

## d) SSHouto

Seurasin tehtävässä [terokarvinen.com](https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh) ohjetta. 

Aloitin kopioimalla /etc/ssh/sshd_config -tiedoston hakemistoon /srv/salt/ssh/ komennolla ``$ sudo cp /etc/ssh/sshd_config /srv/salt/``. Tämän jälkeen avasin kopioimani tiedoston sudoeditillä ``$ sudoedit /srv/salt/sshd_config`` ja muokkasin asetusta, jolla määritellään kuunneltava portti. Herran palvelimella oli myös tietoturvasyistä "PermitRootLogin no", jonka kommentoin pois lisäämällä rivin eteen risuaidan (#), sillä kirjautumista yritettäisiin vakiona root-tunnuksella, enkä tiedä mitään muita käyttäjiä vagrantin ylläpitämiltä koneilta.


    aatu@localhost:/srv/salt/$ cat sshd_config | grep 8888
    Port 8888
  

Ajanpuutteen vuoksi en ehtinyt viritellä uutta virtuaalikonetta, joten poistin toiselta orjalta ssh:n komennolla ``$ sudo salt 't001' state.single pkg.removed openssh-server``. Tämän jälkeen ajoin orjia pyörittävällä isäntäkoneella komennon ``$ ssh 192.168.56.2 # t001 ip-osoite``, joka vastasi "Connection refused", eli ssh ei vastaa.

[---- palautus tässä välissä keskeneräisenä ----]

Tein ohjeen mukaisen sshd.sls-nimisen tiedoston tällaisella sisällöllä:

    openssh-server:
      pkg.installed

    /etc/ssh/sshd_config:
      file.managed:
        - source: salt://sshd_config

    sshd:
      service.running:
        - watch:
          - file /etc/ssh/sshd_config # kaksoispiste jäi puuttumaan filen perästä, johti virheeseen.

Tämän jälkeen koitin ajaa modulin t001:lle komennolla ``$ sudo salt 't001' state.apply sshd``. Ensimmäinen ajokerta asensi openssh-serverin ja muokkasi /etc/ssh/sshd_config-tiedoston onnistuneesti, mutta epäonnistui viimeisessä vaiheessa virheilmoituksella.

````
...
Comment: The following requisites were not found:
                                 watch:
                                     id: file /etc/ssh/sshd_config
...
````

Huomasin filen perästä puuttuvan kaksoispisteen, jonka lisäämisen jälkeen viimeinenkin vaihe onnistui. Tämän jälkeen testasin orjia pyörittävällä isäntäkoneella, toimiiko ssh-yhteys. Vakioportilla ssh kieltäytyi yhteyden muodostamisesta, mutta portilla 8888 kyseli salasanaa, eli käyttäjätunnusten ollessa tiedossa yhteys olisi todennäköisesti muodostunut.

![Add file: ssh testi](/img/sshd_testi.png)
> Vakioportti ei toimi, portti 8888 toimii

[--- loppuosuus palautettu lisätty 14:20 ---]
