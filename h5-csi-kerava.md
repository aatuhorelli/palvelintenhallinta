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
   - sort järjestää tulosteen
     
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

## a) CSI Kerava

    Laitteen tiedot:
    OS: Debian 12
    CPU: Intel i7-6700k
    RAM: 32GB
    SSD: 100GB

Aloitin päivittämällä paketinhallinnan listat ja asennetut paketit komennoilla ``$ sudo apt-get update`` & ``$ sudo apt-get upgrade``. Tehtävänannon vinkkiosiota mukaillen ajoin /etc/-hakemiston tiedostojen etsimiseksi komennon ``$ sudo find /etc/ -printf '%T+ %p\n' | sort``.

Komennon tulkinta manpagesia (``$ man find``) hyödyntäen:
 - sudo: seuraavan komennon ajo pääkäyttäjäoikeuksilla. Olen aiemmin findiä käyttäessä huomannut, että kaikkia tiedostoja ei listata käytettäessä ilman sudoa.
 - find: ajettava komento. Find etsii tiedostoja hakemistohierarkiasta.
 - /etc/: hakemisto, josta haetaan
 - printf: findin tulosteen muotoilu
   - %T+: tulostaa tiedoston viimeisen muutoksen aikaleiman (T = aika, + = pvm+aika). Päivämäärä ja aika määräytyvät laitteen aikavyöhykkeen mukaan.
   - %p: tiedoston nimi
   - \n: rivinvaihto. printf ei tulosta rivinvaihtoja automaattisesti, vaan ne tulee lisätä itse.


/etc/-kansion viimeisimmät muokatut tiedostot:

    $ sudo find /etc/ -printf '%T+ %p\n' | sort
    ...
    2023-11-26+20:19:00.8039408450 /etc/default # Tiedostoa /etc/default muokattu 26.11.2023 kello 20:19:00. 
    2023-11-26+20:19:00.8679408450 /etc/kernel/preinst.d
    2023-11-26+20:19:00.9159408460 /etc/modprobe.d
    2023-11-26+20:19:02.1559408530 /etc/gimp/2.0
    2023-11-26+20:19:02.2159408540 /etc/firefox-esr
    2023-11-26+20:19:03.1279408590 /etc/ld.so.cache
    2023-11-26+20:19:04.3919408670 /etc/
    2023-11-26+20:19:04.3919408670 /etc/mailcap


Viimeisimmät muutokset näyttivät olevan hieman aiemmin suoritetun ``$ sudo apt-get upgrade``:n yhteydessä päivitettyjen ohjelmien tiedostoissa. 

Ajoin vielä kotihakemistossa vastaavan komennon ilman sudoa, koska oletin käyttäjän näkevän koko kotihakemistonsa sisällön ilman sudoa. Muistin myös kuulleeni, että sudon käyttö kotihakemistossa ei ole suotavaa. 

    $ pwd
    /home/aatu
    $ find -printf '%T+ %p\n' | sort
    ...
    2023-11-26+20:49:24.1279521710 ./.mozilla/firefox/kkkpuhhe.default-esr
    2023-11-26+20:49:24.1279521710 ./.mozilla/firefox/kkkpuhhe.default-esr/permissions.sqlite
    2023-11-26+20:50:36.6479526210 ./.mozilla/firefox/kkkpuhhe.default-esr/sessionstore-backups/recovery.baklz4
    2023-11-26+20:51:05.3879528000 ./.mozilla/firefox/kkkpuhhe.default-esr/sessionstore-backups
    2023-11-26+20:51:05.3879528000 ./.mozilla/firefox/kkkpuhhe.default-esr/sessionstore-backups/recovery.jsonlz4

Kotihakemiston tuoreimmat muokatut tiedostot liittyivät kaikki Firefoxiin, jolla selasin tehtävänantoa ja Githubia.


## b) Gui2fs

Ajattelin muokata jonkin sellaisen ohjelman asetuksia, joka ei raporttia kirjoitellessa ollut jo valmiiksi käytössä. Hetken Applications-listaa selattuani valitsin uhriksi GIMPin. (Applications -> Graphics -> Gnu Image Manipulation Program). GIMPin navigointipalkista valitsin Edit -> Preferences päästäkseni asetuksiin, josta kävin poistamassa Exif metadatan lisäämisen kuviin (Image import & Export -> [ ] Export Exif metadata by default when available).

![Add file: gimpin asetukset](img/GIMP.png)
> GIMPin asetukset ja tehty muutos

Seuraavaksi piti löytää tiedosto, johon muutos kirjoitettiin. Heti asetuksen muokkauksen jälkeen siirryin juurihakemistoon ``$ cd /``ja ajoin siellä komennon ``$ sudo find -printf '%T+ %p\n' | grep gimp | sort``, joka etsii pääkäyttäjäoikeuksilla tiedostojärjestelmän kaikkien tiedostojen muokkausajat ja sijainnit, listaa vain sanan 'gimp' sisältävät osumat ja järjestää listan. Tuorein muokattu tiedosto oli ``/home/aatu/.config/GIMP/2.10/gimprc``. Muutos oli ilmeisesti  vain käyttäjäkohtainen, koska asetus tallennettiin käyttäjän kotihakemistoon, eikä /etc/:n alle. 

Tarkistin vielä tiedoston sisällön varmistuakseni löytäneeni oikean tiedoston:

    /$ cat /home/aatu/.config/GIMP/2.10/gimprc
    # GIMP gimprc
    ...
    (export-metadata-exif no) # muokattu asetus
    ...
    # end of gimprc

## c) Komennus

Loin ensin uuden yksinkertaisen hello world -komennon omaan kotihakemistooni. Ensin paikallinen testaus, tämän jälkeen Saltin kimppuun. Päätin toteuttaa ohjelman bashilla, joten tarkistin myös bashin sijainnin ohjelman ensimmäistä riviä(shebang) varten. Shebangissa määritellään, millä ohjelmalla tiedosto tulee ajaa.

Komennot:

    $ cd /home/aatu
    $ mkdir komennot
    $ cd komennot
    $ which bash
    /usr/bin/bash
    $ micro helloworld
    $ cat helloworld
    #!/usr/bin/bash 

    echo Hello $USER

Tämän jälkeen testasin ohjelman ajamista. ``$ bash helloworld`` toimi, mutta ``$ ./helloworld`` tulosti virheen "bash: ./helloworld: Permission denied". Olin unohtanut määritellä chmodilla tiedoston käyttöoikeudet kuntoon. 

    $ ls -l helloworld  # tiedostojen listaus. -l näyttää oikeudet
    -rw-r--r-- 1 aatu aatu 34 26.11. 21:50 helloworld
    
Tiedosto oli yhden käyttäjän ryhmän 'aatu' omistama, ja oikeudet olivat -rw-r--r--. Ensimmäinen viiva tarkoittaa, että kyseessä on tiedosto. Kolme ensimmäistä merkkiä ovat käyttäjän oikeudet (r = read, w= write, - = suoritusoikeutta x ei ole), seuraavat kolme (r--) vastaavasti ryhmän oikeudet (pelkkä luku) ja viimeiset kolme (r--) muiden kuin omistajakäyttäjän tai -ryhmän oikeudet. Koska halusin komennon olevan kaikkien käytössä, tuli ajo-oikeus x määrittää kaikille. Lisäsin sen komennolla ``$ chmod ugo+x helloworld``. Tämä lisää User, Group ja Other -käyttäjille oikeuden suorittaa tiedosto. Tämän jälkeen tiedoston ajaminen onnistui myös komennolla ``$ ./helloworld``. 

![Add file: helloworld oikeudet](/img/helloworld_oikeudet.png)
> Tiedoston oikeudet, niiden muokkaus ja testi

Kopioin tiedoston hakemistoon /usr/bin/, jossa kaikkien käyttäjien yhteiset ohjelmat ovat. Kansio on rootin omistama, joten sinne kirjoittaminen vaatii sudon käyttöä. Käytetty komento: ``$ sudo cp helloworld /usr/bin/``. Tämän jälkeen testasin komennon toiminnan toisessa kansiossa.

    $ cd ..
    $ helloworld 
    Hello aatu

Ohjelma toimi, joten poistin sen /usr/bin-hakemistosta voidakseni varmistua siitä, että tiedosto on siirtynyt sinne myöhemmin saltin avulla. Tämän jälkeen helloworld-komento ei enää toiminut.

    ~$ sudo rm /usr/bin/helloworld
    ~$ helloworld
    bash: /usr/bin/helloworld: No such file or directory

Laitteella oli jo Salt minion ja master paikallisesti asennettuna, joten testasin niiden olevan toiminnassa. ``$ sudo salt '*' test.ping # kotiorja: True --> toiminnassa ja vastaa kutsuihin``. Loin asennettavaa ohjelmaa varten uuden kansion, johon kopioin tekemäni ohjelman ja loin init.sls-tiedoston.

    $ sudo mkdir /srv/salt/hello
    $ sudo cp /home/aatu/komennot/helloworld /srv/salt/hello
    $ cd /srv/salt/hello
    $ hello$ ./helloworld # ohjelman testaaminen
    Hello aatu
    $ sudoedit init.sls
    $ cat init.sls
    /usr/bin/helloworld:
      file.managed:
        - source: salt://hello/helloworld

Tila hello oli nyt luotu, joten testasin sen ajamista orjalle.

    $ sudo salt '*' state.apply hello # ajaa kaikille orjille tilan 'hello'
    kotiorja:
    ----------
          ID: /usr/bin/helloworld
    Function: file.managed
      Result: True # onnistunut suoritus
     Comment: File /usr/bin/helloworld updated # tiedosto päivitetty
     Started: 22:25:06.209155
    Duration: 50.259 ms
     Changes:   
              ----------
              diff:
                  New file # uusi tiedosto /usr/bin/helloworld luotu
              mode:
                  0644

    Summary for kotiorja
    ------------
    Succeeded: 1 (changed=1)
    Failed:    0
    ------------
    Total states run:     1
    Total run time:  50.259 ms
    $ helloworld # testisuoritus
    bash: /usr/bin/helloworld: Permission denied

Testisuorituksessa vastanotin valituksen käyttöoikeuksista, joten etsin salt-callin ohjeista niiden määrittämiseen liittyviä asetuksia. Käytin kotihakemistossani tehtävänannon vinkkiosion komentoa ``$ sudo salt-call --local sys.state_doc > statedoc``, jolla salt-callin ohjeet viedään omaan tiedostoonsa helpommin luettavaan muotoon. Tämän jälkeen avasin sen komennolla ``$ less statedoc`` ja etsin file.managediin liittyviä määrityksiä lessin hakutoiminnolla: ``/file.managed``.

![Add file: statedoc file.managed](/img/statedoc_file.managed.png)
> Oletettavasti oikeita asetuksia statedocista

Koska oikeudet tuli määrittää ilmeisesti numeerisesti, tarkistin vielä alkuperäisestä helloworld-tiedostosta oikeudet komennolla ``$ stat /home/aatu/komennot/helloworld # Access: (0755/-rwxr-xr-x)``. Muistelin edelliseltä luennolta, että vaikka statedocissa oikeudet eivät ole lainausmerkeissä, on niiden lisääminen toiminnan kannalta järkevää. Muokkasin aiemmin luomaani init.sls-tiedostoa oikeudet sinne lisäten:

    $ sudoedit /srv/salt/hello/init.sls
    $ cat /srv/salt/hello/init.sls
    /usr/bin/helloworld:
      file.managed:
        - source: salt://hello/helloworld
        - user: root
        - group: root
        - mode: "0755"

Ajoin tilan uudestaan saltilla ja testasin toiminnan: ``$ sudo salt '*' state.apply hello``, ``$ helloworld``. Tiedosto päivitettiin ja komento oli ajettavissa.

## d) Apassi

Testasin, onko Apache2 asennettu ``$ systemctl status apache2 # Unit apache2.service could not be found.``. Palvelua ei löytynyt, eli se olisi ensin asennettava. Olin jo aiemmin päivittänyt paketinhallinnan listat, joten ``$ sudo apt-get install apache2 -y`` riitti paketin asentamiseen. Tämän jälkeen käynnistin Apachen komennolla ``$ sudo systemctl start apache2`` ja testasin sen olevan päällä siirtymällä selaimella osoitteeseen localhost. 

![Add file: apassi päällä](/img/apassi.png)
> Apache testisivu päällä

Tämän jälkeen laitoin käyttäjien kotisivut käyttöön, käynnistin Apachen uudelleen ja tarkistin, mitä tiedostoa asetus muokkasi. 

    $ sudo a2enmod userdir
    $ sudo systemctl restart apache2
    $ sudo find /etc/ -printf '%T+ %p\n' | sort
    ...
    2023-11-26+22:51:47.6679977870 /etc/apache2/mods-enabled
    2023-11-26+22:51:47.6679977870 /etc/apache2/mods-enabled/userdir.conf
    2023-11-26+22:51:47.6679977870 /etc/apache2/mods-enabled/userdir.load

Tarkastin vielä mitä userdir pois päältä kytkeminen tekee komennolla ``$ sudo a2dismod userdir``. Tämä poisti mods-enabled kansiosta userdir-tiedostot, jotka olivat ``$ stat``-komennon mukaan symbolisia linkkejä vastaaviin tiedostoihin /etc/apache/mods-available/-hakemistossa. Kytkin userdirin takaisin käyttöön a2enmodilla. Tämän jälkeen tein vielä omalle käyttäjälleni testisivun ja testasin sitä curlilla.

    $ mkdir /home/aatu/public_html/
    $ echo TEST > /home/aatu/public_html/index.html
    $ curl localhost/~aatu/
    TEST # toimii!

Tarvittava manuaalityö oli nyt tehty. Seuraavaksi sama pitäisi toistaa saltilla. 

    $ sudo mkdir /srv/salt/apache
    $ sudo cp /home/aatu/public_html/index.html /srv/salt/apache/default-index.html
    $ ls /srv/salt/apache/
    default-index.html  userdir.conf  userdir.load # tiedostot paikallaan

Init.sls-tiedostoon olisi määritettävä seuraavat asiat:
 - Apache on oltava asennettu ja päällä
 - Apachen oletusetusivu korvattu  
 - Userdir.conf ja userdir.load tiedostoille luodaan symbolinen linkki /etc/apache2/mods-enabled/-kansioon
 - Apache2 on päällä, ja uudelleenkäynnistetään tehtäessä userdir-tiedostoihin muutoksia.
   

Käytin apuna aihetta käsittelevää [terokarvinen.com](https://terokarvinen.com/2018/04/03/apache-user-homepages-automatically-salt-package-file-service-example/) artikkelia. Kopioin init.sls:n sisällön kokonaisuudessaan.

    $ cat /srv/salt/apache/init.sls
    
    apache2:
     pkg.installed
    /var/www/html/index.html:
     file.managed:
       - source: salt://apache/default-index.html
    /etc/apache2/mods-enabled/userdir.conf:
     file.symlink:
       - target: ../mods-available/userdir.conf
    /etc/apache2/mods-enabled/userdir.load:
     file.symlink:
       - target: ../mods-available/userdir.load
    apache2service:
     service.running:
       - name: apache2
       - watch:
         - file: /etc/apache2/mods-enabled/userdir.conf
         - file: /etc/apache2/mods-enabled/userdir.load

Seuraavaksi poistin Apachen ja siihen liittyvät tiedostot kotihakemiston /public_html/-kansiota lukuunottamatta. Tuon kansion säilytin siksi, että voisin testata käyttäjien userdirin menneen onnistuneesti päälle. Tarkistin vielä, ettei localhost vastaa curliin ja että /etc/apache2/-hakemistossa ei ole tiedostoja, joita saltin muokkausten tulisi käsitellä.

    $ sudo apt-get purge apache2 # apachen ja siihen liittyvien tiedostojen poisto
    apache$ curl localhost
    curl: (7) Failed to connect to localhost port 80 after 0 ms: Couldn't connect to server # ei käytössä
    apache$ ls /etc/apache2/
    conf-available # Apachen kansiossa ei mods-enabled tai mods-available kansioita


Hyvältä näytti. Sitten testaamaan!

    $ sudo salt '*' state.apply apache
    ...
    Summary for kotiorja
    ------------
    Succeeded: 5 (changed=5) # kaikki vaiheet onnistuneet
    Failed:    0

Koska yhteenveto väitti kaikkien vaiheiden onnistuneen, testasin oman käyttäjäni kotihakemiston toimintoa curlilla.

    $ curl localhost/~aatu/
    TEST

Sivu vastasi, eli Apachen demoni ja käyttäjien kotihakemistot olivat käytössä. Toimii!

## e) Ämpärillinen

Loin ensin muutaman yksinkertaisen komennon kotihakemistoni komennot-kansioon. C-kohdassa kirjoiteltu helloworld löytyi jo, joten hyödynsin sitä osana tehtävää. Annoin kokeilun vuoksi myös [ChatGPT](https://chat.openai.com/):lle tehtäväksi luoda bash-pohjaisen simppelin arvauspelin, jossa koitetaan arvata satunnaisgeneroitua lukua yhden ja ohjelmaa käynnistäessä määritetyn maksimin väliltä. 

![Add file: ohjelmat koodeineen](/img/ohjelmat.png)

Tehtävä ei juurikaan poikkea c-kohdasta, määriteltyäni kaikille suoritusoikeudet ``$ chmod ugo+x *`` ja testattuani kutakin komentoa, kopioin ne kansioon /srv/salt/ampari/ komennoilla ``$ sudo mkdir /srv/salt/ampari/`` ja ``$ sudo cp * /srv/salt/ampari/``. Poistelin vielä c-kohdassa asentamani helloworld-komennon ``$ sudo rm /usr/bin/helloworld``, minkä jälkeen ``$ helloworld`` ei enää toiminut. 

C-kohdan init-sls oli myös käyttökelpoinen tähän tehtävään, joten siirryin hakemistoon /srv/salt/ampari/ ja kopioin hello-hakemistosta init.sls:n pohjaksi ``$ sudo cp /srv/salt/hello/init.sls .``. Muokkasin sudoeditillä lähdetiedostojen polut oikein ja lisäsin arvaa ja huomenta -ohjelmien määrittelyt. 

    $ cat init.sls 
    /usr/bin/helloworld:
      file.managed:
        - source: salt://ampari/helloworld
        - user: root
        - group: root
        - mode: "0755"
    /usr/bin/arvaa:
      file.managed:
        - source: salt://ampari/arvaa
        - user: root
        - group: root
        - mode: "0755"
    /usr/bin/huomenta:
      file.managed:
        - source: salt://ampari/huomenta
        - user: root
        - group: root
        - mode: "0755"

Tilan ajo orjalle:

    $ sudo salt '*' state.apply ampari
    ...
    Summary for kotiorja
    ------------
    Succeeded: 3 (changed=3) # Tiedostot luotu onnistuneesti
    Failed:    0
    ------------
    Total states run:     3

Saman komennon ajaminen uudestaan palautti onnistuneen suorituksen ja kertoi kaikkien tiedostojen olevan jo paikallaan ja oikeassa muodossa. Idempotenssi siis toteutui. Lopuksi luotujen komentojen testailua:

![Add file: testailua](/img/komennot_toimii.png)
> Komentojen testaus. Hyvä peli on.


# Lähteet

 - Karvinen, 2023. Infra as Code 2023. Luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/. Luettu 26.11.2023.
 - Karvinen, 2018. Apache User Homepages Automatically - Salt Package-File-Service Example. Luettavissa https://terokarvinen.com/2018/04/03/apache-user-homepages-automatically-salt-package-file-service-example/. Luettu 26.11.2023.
 - Linux manpages: find, sort








