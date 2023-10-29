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


