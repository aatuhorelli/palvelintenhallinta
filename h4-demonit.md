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

### Salt contributors: Salt overview



### Salt contributors: Salt states



### Karvinen 2018: Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port
