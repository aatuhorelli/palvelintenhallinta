# H1 - Viisikko

Viikkotehtävien tehtävänannot löytyvät osoitteesta https://terokarvinen.com/2023/configuration-management-2023-autumn/

## x) Lue ja tiivistä

### Create a Web Page Using Github
 - Github on nopea tapa julkaista verkkoon
 - Vaiheet:
   - kirjaudu github.comiin
   - Luo julkinen repositorio
     - Yhden lauseen kuvaus
     - Luomalla README-tiedoston repositorion luomisen yhteydessä vältyt tulevaisuuden ongelmilta
     - Valitse lisenssi. GNU GPL3 suositeltu.
   - Uusi tiedosto repositorioon
     - Add file: create a new file
     - tiedoston_nimi.md (.md-pääteen käyttäminen muuntaa tiedoston automaattisesti webbisivuksi)
   - Kirjoita sisältöä tiedostoon ja julkaise se
  
Lähde: Tero Karvinen, 2023. Create a Web Page Using Github. Luettavissa: https://terokarvinen.com/2023/create-a-web-page-using-github/

### Run Salt Command Locally
 - Saltia käytetään usein tietokoneiden(orja) hallinnointiin verkon yli
 - Samat komennot toimivat Linuxissa ja Windowsissa
 - Komennot luotava idempotenteiksi. Muutoksia kokoonpanoon tehdään vain tarvittaessa.
 - Salt-komentojen ajaminen paikallisesti on hyvä tapa harjoitella, testata ja suorittaa nopeita asennuksia tietokoneelle. Paikallisesti testatessa tulokset näkyvät välittömästi.

#### Salt Slaven asennus
    $ sudo apt-get update
    $ sudo apt-get -y install salt-minion
    $ sudo salt-call --version # testaa asennuksen onnistuminen

#### Tärkeimmät komennot:

Ohjeet:

    $ sudo salt-call --local -l sys.state_doc

pkg.installed:

    $ sudo salt-call --local -l info state.single pkg.installed esimerkki-paketti # paketin tulee olla asennettu
    $ sudo salt-call --local -l info state.single pkg.removed esimerkki-paketti # pakettia ei pidä olla asennettuna

file.managed:

    $ sudo salt-call --local -l info state.single file.managed /tmp/hellotero # tarkistaa, että tiedosto löytyy
    $ sudo salt-call --local -l info state.single file.managed /tmp/hellotero contents="foo" # tiedoston sisällön määrittäminen
    $ sudo salt-call --local -l info state.single file.absent /tmp/hellotero # tiedostoa ei saa löytyä

service.running (käytetään myös demonien uudelleenkäynnistämiseen asetusten muuttumisen yhteydessä):

    $ sudo salt-call --local -l info state.single service.running apache2 enable=True # demoni käynnissä
    $ sudo salt-call --local -l info state.single service.dead apache2 enable=False # demoni ei käynnissä

user.present:

    $ sudo salt-call --local -l info state.single user.present testi01 # käyttäjä testi01 löytyy
    $ sudo salt-call --local -l info state.single user.absent testi01 # käyttäjää testi01 ei ole

cmd.run (Idempotenssin määrittely tärkeää cmd.runin yhteydessä / creates, onlyif, unless):

     $ sudo salt-call --local -l info state.single cmd.run 'touch /tmp/foo' creates="/tmp/foo" # luo tiedoston /tmp/foo touchilla

Lähde: Tero Karvinen, 2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/

## A) Saltin-minionin asennus koneelle

Viikkotehtäviä varten asensin tuoreen kopion Debian12:sta (12.2.0-amd64-xcfe) VirtualBoxiin. 
    
    Isäntälaitteen tiedot:
    OS: Windows 10
    Prosessori: Intel I7-6700K @ 4.5Ghz (64bit)
    RAM: 32GB
    SSD: 1TB M.2 NVMe

Muistin luennolta, että Saltin asentaminen Debian 12:een paketinhallinnan kautta ei toistaiseksi onnistu. Seurasin [Teron ohjeita tehtävänannon vinkkiosiosta](https://terokarvinen.com/2023/configuration-management-2023-autumn/)[1]. Ohjeen komennot lisäävät uuden lähteen paketinhallintaan, päivittävät paketinhallinnan listat ja asentavat salt-minionin. Netistä löytyvien komentojen ajamisessa on syytä noudattaa harkintaa. Pidin Teroa kuitenkin luotettavana lähteenä, joten uskalsin ajaa komennot.

Loin uuden shell-skriptin salt-asennus.sh (````$ nano salt-asennus.sh````), johon lisäsin seuraavat rivit. Näin kaikki komennot on mahdollista suorittaa yhdellä komennolla ````$ bash salt-asennus.sh````. 

    sudo mkdir /etc/apt/keyrings # ei haittaa vaikka sanoisi etta hakemisto on jo
    sudo curl -fsSL -o /etc/apt/keyrings/salt-archive-keyring-2023.gpg https://repo.saltproject.io/salt/py3/debian/11/amd64/SALT-PROJECT-GPG-PUBKEY- 
    2023.gpg
    echo "deb [signed-by=/etc/apt/keyrings/salt-archive-keyring-2023.gpg arch=amd64] https://repo.saltproject.io/salt/py3/debian/11/amd64/latest bullseye 
    main" | sudo tee /etc/apt/sources.list.d/salt.list
    sudo apt-get update
    sudo apt-get install salt-minion

Tarkastin tiedoston sisällön komennolla ````$ cat salt-asennus.sh````, ja todettuani sen oikeaksi ajoin tiedoston: ````$ bash salt-asennus.sh````. Tämän jälkeen syötin sudo-oikeuksille salasanan ja asennus käynnistyi. Asennuksen varmistaessa haluanko asentaa salt-minionin (Y/N)?, valitsin Y. 

![Add file: salt-shell](/img/salt-shelli.png)

Varmistin, että asennus onnistui tarkistamalla saltin version komennolla ````$ sudo salt-call --version-````. Versio tulostui, joten asennus oli onnistunut.

![Add file: salt-asennettu](/img/salt-asennettu.png)


## B & C) Viisi tärkeintä + idempotenssi

Teron [artikkelin](https://terokarvinen.com/2021/salt-run-command-locally/)[2] mukaan viisi tärkeintä idempotentin tilan funktiota Saltissa ovat pkg, file, service, user ja cmd.run. Idempotenssilla tarkoitetaan sitä, että määrittelemme järjestelmän tavoitetilan, johon sen tulee pyrkiä. Jos tavoitetila on jo saavutettu, kokoonpanoon ei tehdä muutoksia. 

Testasin kaikkia viittä paikallisesti. 

### pkg

Virtuaalikoneessa oli puhdas asennus Debianista, joten päätin asentaa siihen Apache2:n Saltin avulla. Varmistin ensin, että pakettia ei oltu asennettu potkaisemalla sitä: ````$ sudo systemctl restart apache2````. "Failed to restart apache2.service: Unit apache2.service not found", eli Apachea ei löydy.

    $ sudo salt-call --local -l info state.single pkg.installed apache2
    
Asennus kesti noin 10 sekuntia, minkä jälkeen Salt antoi seuraavanlaisen yhteenvedon:

![Add file: apache2-asennus](/img/apache2-asennus.png)

Raportti vaikutti melko selkeältä. Käytin funktiota pkg.installed paketille apache2 kellonajassa 21:18:07. "Result: True" viittaa siihen, että suorituksen jälkeen paketti riippuvuuksineen oli asennettu. Kommenttikenttä kertoo, että apache2-paketti asennettiin. Alareunan yhteenvedossa Succeeded: 1 tarkoittaa, että haluttu paketti löytyy laitteelta ja changed=1, että se on asennettu suorituksen aikana.

Ajoin komennon vielä toisen kerran. Tällöin idempotenssin hengessä tavoitetila oli jo saavutettu, ja muutoksia ei pitäisi tulla. 

![Add file: apache2-toinen yritys](/img/apache2-toka.png)

".. packages are already installed" / Succeeded: 1 (ilman muutoksia). ````$ curl localhost```` palautti myös Apachen index-sivun. Apache2 siis löytyi!

Testasin myös paketin poistoa komennolla ````$ sudo salt-call --local -l info state.singel pkg.removed apache2````, minkä jälkeen koitin uudelleenkäynnistää apachen ````$ sudo systemctl restart apache2````. 

![Add file: apache2 pois](/img/apache2-poistettu.png)

Sinne meni! Asensin vielä saltilla takaisin, koska ajattelin hyödyntää Apachea myöhemmässä vaiheessa.

### service.running/dead

Apache2 oli siis asennettu, joten määrittelin Saltilla, että haluan sen myös olevan käynnissä.

    $ sudo salt-call --local -l info state.single service.running apache2 enable=True

![Add file: apache käynnissä](/img/apache2-käynnissä.png)

Apache2 oli käynnistynyt itsestään asennuksen jälkeen. "The service apache2 is already running". ````$ curl localhost```` palautti Apachen esimerkkisivun.

Päätin sammuttaa demonin Saltin avulla. Ajoin komennon kahdesti peräkkäin. Idempotenssin mukaisesti ensimmäisellä suorituskerralla pitäisi tulla Succeeded: 1, changed=1, eli prosessi on tapettu onnistuneesti. Toisella suorituskerralla prosessi ei ole enää päällä, joten toimenpiteitä ei tehdä.

    $ sudo salt-call --local -l info state.single service.dead apache2 enable=False

Odotettu lopputulos syntyi.

![Add file: Idempotenssia, beibi](/img/idempotenssi-dead.png)

### user

user-funktiolla voidaan varmistaa, että tietty käyttäjä löytyy tai on poistettu. Varmistin ensin user.absentia hyödyntäen, että käyttäjää kikki ei löydy. 

    $ sudo salt-call --local -l info state.single user.absent kikki

![Add file: kikki puuttuu](/img/kikki-absent.png)

Käyttäjää ei löytynyt, ei muutostarpeita. Onnistunut suoritus.

Seuraavaksi loin käyttäjän user.presentiä hyödyntäen. Tarkistin lopuksi, että kikin kotihakemisto oli luotu. 
    
    $ sudo salt-call --local -l info state.single user.present kikki
    $ ls -l /home/
    
![Add file: kikki löytyy](/img/kikki-present.png)

Käyttäjä luotu (changed=1), kotihakemisto löydettävissä.

### file.managed

File.managed-funktiolla voidaan varmistaa, että tiedosto löytyy, ja halutessaan määritellä sille sisältöä. File.absent varmistaa, että tiedosto on poistettu. Aloitin tarkistamalla, että testitiedostoa /tmp/moro ei löydy:

    $ sudo salt-call --local -l info state.single file.absent /tmp/moro

![Add file: ei moro](/img/file-absent.png)

Not present, succeeded: 1. Tiedosto onnistuneesti todettu poissaolevaksi ilman muita toimenpiteitä. 

Seuraavaksi loin /tmp/moro-tiedoston Saltilla:

    $ sudo salt-call --local -l info state.single file.managed /tmp/moro # Tiedoston olemassaolon tarkistus ja luominen
    $ cat /tmp/moro # tyhjä tiedosto, ei tulostettavaa
    $ ls /tmp/ # moro löytyy

![Add file: moro moro](/img/file-managed.png)

File created, succeeded 1, changed 1. Tiedosto luotu ja toiminto onnistunut. Tyhjä tiedosto on tosin tylsä, joten lisäsin sinne sisältöä.

    $ sudo salt-call --local -l info state.single file.managed /tmp/moro contents="No moro moro!"

![Add file: moro päivitetty](/img/file-updated.png)

Tiedosto /tmp/moro löytyi ja sen sisältö päivitettiin onnistuneesti. 

### cmd.run

Cmd.runilla voidaan ajaa tietty komento. Cmd.run on määriteltävä idempotentiksi creates, unless tai onlyif avulla. Loin tiedoston echolla ja määrittelin, että komento luo tiedoston. Sen ei siis pitäisi tehdä mitään, jos tiedosto jo löytyy. Ajoin seuraavan komennon kahdesti:

    $ sudo salt-call --local -l info state.single cmd.run 'echo olen seeämdee run >> /tmp/runi' creates="/tmp/runi"

![Add file: seeämdee](/img/cmd-run.png)

Ensimmäisellä suorituskerralla tiedosto luotiin sisällöllä "olen seeämdee run" (changed=1). Toisella suorituskerralla tiedosto löytyi jo, eikä muutoksia tehty. 


## D) Tietoa koneesta

Komento ````$ sudo salt-call --local grains.items```` tulosti listan virtuaalikoneen tiedoista. Listattuna oli tietoja käyttöjärjestelmästä, laitteen raudasta, saltista ja monesta muusta asiasta. 

VirtualBox nousi useampaan kenttään(biosversion, boardname, productname, virtual). Näissä ei ilmeisesti edes yritetä virtualisoida isäntälaitteen tietoja, vaan käytetään sen sijaan VirtualBoxia. 

Pythonista oli kerätty versiotietojen lisäksi muutamia eri polkuja. Oletan tämän pohjalta, että Pythonia hyödynnetään jotenkin saltin käytössä. 

Virtualisoinnissa annetut resurssit näyttivät täsmäävän hyvin grains.items:llä listattuihin tietoihin:
 - cpu_model: i7-6700k
 - mem_total: 7940
 - num_cpus: 4


## Lähteet

[1]Tero Karvinen, 2023. Infra as Code 2023. Luettavissa https://terokarvinen.com/2023/configuration-management-2023-autumn/

[2]Tero Karvinen, 2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/
