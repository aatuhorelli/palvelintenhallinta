# H5 - CSI Kerava

Tehtävänannot luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/

## x) terokarvinen.com - Apache User Homepages Automatically – Salt Package-File-Service Example

Artikkeli sisältää esimerkkejä demonien konfiguraatiotiedostoista Saltille. Package-file-service on yleisin tapa konfiguroida demoneita.

### Konfiguraatiotiedostojen paikannus

 - Konfiguraatiot tehdään ensin käsin, sitten vasta automatisoidaan.
 - Konfiguraatiotiedostojen paikannus (jos muokkaus tehty esim. graafisesta käyttöliittymästä sudoeditin sijaan): ``$ sudo find -printf '%T+ %p\n | sort``
   - ``$ find`` etsii tiedostoja
   - %T+ tulostaa muokkausajan
   - %p tiedostonimi ja polku
   - \n rivinvaihto
     
### Komentojen käyttö Saltin tilan määrittelyssä
 - cmd määriteltävä idempotentiksi konfiguraatiotiedostoissa käytettäessä
   - esim.
     ````
     $ cat /srv/salt/apache/init.sls

     apache2:
       pkg.installed
     /var/www/html/index.html:
       file.managed:
         - source: salt://apache/default-index.html
     a2enmod userdir:
       cmd.run:
         - creates: /etc/apache2/mods-enabled/userdir.conf # Idempotenssi: komento ajetaan, jos tiedostoa ei löydy
     apache2service:
       service.running:
         - name: apache2
         - watch:
           - cmd: 'a2enmod userdir'
     ````
### Tiedostojen hyödyntäminen
 - Esimerkkitiedostojen käyttö on komentojen ajamista luotettavampi tapa määritellä tiloja
 - Edellisen esimerkin toteutus tiedostojen avulla. Kommentit eivät ole artikkelista, vaan omiani.
   ````
   $ cat /srv/salt/apache/default-index.html
   See you at TeroKarvinen.com
   $ cat /srv/salt/apache/init.sls

   apache2:
     pkg.installed # apache2 asennettu
   /var/www/html/index.html: 
     file.managed: # tiedosto /var/www/html/index.html löytyy
       - source: salt://apache/default-index.html # tiedoston sisältö (salt:// = /srv/salt/)
   /etc/apache2/mods-enabled/userdir.conf:
     file.symlink:
       - target: ../mods-available/userdir.conf # symbolinen linkki: userdir.conf -> userdir.load
   /etc/apache2/mods-enabled/userdir.load:
     file.symlink:
       - target: ../mods-available/userdir.load
   apache2service:
     service.running:
       - name: apache2 # apachen demoni päällä
       - watch: # demonin uudelleenkäynnistys, jos listattuja tiedostoja muokataan
         - file: /etc/apache2/mods-enabled/userdir.conf
         - file: /etc/apache2/mods-enabled/userdir.load
   ````

 - Tilojen ajaminen herralta orjille: ``$ sudo salt '*' state.apply apache``
 - Käyttäjien kotisivut ovat kotihakemistojen kansiossa public_html (/$HOME/public_html/)
   - Testaus selaimella: localhost/~käyttäjän_nimi/ (esim. localhost/~aatu/)
   - Hakemiston käyttöoikeudet kuntoon: ``$ chmod ugo+x $HOME/public_html/; chmod ug+r $HOME/public_html/index.html``

Lähde: https://terokarvinen.com/2018/04/03/apache-user-homepages-automatically-salt-package-file-service-example/

