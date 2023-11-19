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



### Karvinen 2018: Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port
