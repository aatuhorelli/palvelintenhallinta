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

cmd.run (suosi file, service tai useria cmd.run-komennon sijaan):

     $ sudo salt-call --local -l info state.single cmd.run 'touch /tmp/foo' creates="/tmp/foo" # ajaa komennon touch /tmp/foo

Lähde: Tero Karvinen, 2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/

## A) Saltin-minionin asennus koneelle

Viikkotehtäviä varten asensin tuoreen kopion Debian12:sta (12.2.0-amd64-xcfe) VirtualBoxiin. 
    
    Isäntälaitteen tiedot:
    OS: Windows 10
    Prosessori: Intel I7-6700K @ 4.5Ghz (64bit)
    RAM: 32GB
    SSD: 1TB M.2 NVMe

Muistin luennolta, että Saltin asentaminen Debian 12:een paketinhallinnan kautta ei toistaiseksi onnistu. Seurasin [Teron ohjeita tehtävänannon vinkkiosiosta](https://terokarvinen.com/2023/configuration-management-2023-autumn/). Ohjeen komennot lisäävät uuden lähteen paketinhallintaan, päivittävät paketinhallinnan listat ja asentavat salt-minionin. Netistä löytyvien komentojen ajamisessa on syytä noudattaa harkintaa. Pidin Teroa kuitenkin luotettavana lähteenä, joten uskalsin ajaa komennot.

Loin uuden tiedoston salt-asennus.sh (````$ nano salt-asennus.sh````), johon lisäsin seuraavat rivit. Näin kaikki komennot on mahdollista suorittaa komennolla ````$ bash salt-asennus.sh````. 

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
