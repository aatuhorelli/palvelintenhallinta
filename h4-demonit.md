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
