# H2 - Karjaa 
Tehtävänannot luettavissa https://terokarvinen.com/2023/configuration-management-2023-autumn/

## x) Lue ja tiivistä

## a) Vagrantin asennus
Aiemmin olen käyttänyt Linuxia vain virtuaalikoneissa. Edellisellä luennolla mainittiin, että virtuaalikoneiden pyörittäminen sisäkkäin voi aiheuttaa yllättäviä ongelmia. Näiltä välttyäkseni asensin Windowsin rinnalle Debian 12:n. 

Vagrantin asennuksessa seurattu Teron [artikkelin](https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/) ohjeita. 

Testasin ensin, että Vagrantia ja VirtualBoxia ei ole asennettu. Komentoja ei tunnistettu, eli kumpaakaan ei ollut asennettuna valmiiksi.
    
    ~$ vagrant
    bash: vagrant: command not found
    ~$ virtualbox
    bash: virtualbox: command not found

Vagrantin ja VirtualBoxin asennus:

    $ sudo apt-get update # paketinhallinnan listojen päivitys
    $ sudo apt-get install -y vagrant # vagrantin asennus
    $ sudo apt-get install -y virtualbox # virtualboxin asennus, epäonnistui
    E: Package 'virtualbox' has no installation candidate

VirtualBox ei ollut Debianin käyttämän paketinhallinnan piirissä. Etsin Googlesta asennusohjeita hakusanalla "install virtualbox on debian 12" ja koitin löytää mahdollisimman luotettavan oloisen lähteen, sillä asennus olisi tehtävä paketinhallinan ulkopuolisesta repositoriosta. [wiki.debian.org](https://wiki.debian.org/VirtualBox) vaikutti lupaavalta, joten testasin sieltä löytyvää ohjetta, joka mahdollistaa pakettien asennuksen Debian Fast Trackin kautta.  

### Fast Trackin asennus: 

Aloitin lisäämällä Debian Backportsin paketinhallintaan:

    sources.list.d$ pwd
    /etc/apt/sources.list.d
    sources.list.d$ sudo micro bookworm-backports.list # uusi tiedosto paketinhallinan ulkopuoliselle repositoriolle
    [sudo] password for aatu: 
    sources.list.d$ cat bookworm-backports.list 
    deb http://deb.debian.org/debian bookworm-backports main
    sources.list.d$ sudo apt-get update # paketinhallinnan listan päivitys
    ..
    Hit:4 http://deb.debian.org/debian bookworm-backports InRelease # Ilmeisesti nousi listoille, koska mainitaan updaten yhteydessä
    ..


    $ sudo apt install fasttrack-archive-keyring # fasttrack archiven keyringin asennus

Yritin selvittää, mikä keyringien tarkoitus on. Googlen avulla löysin oikean manpages-sivun: ````$ man apt-secure````, jonka mukaan keyringit ovat osa repositorioiden turvallisen julkaisemisen ketjua.

Lisäsin Fast Trackin paketinhallintaan avaamalla sources.list-tiedoston komennolla ````$ sudoedit /etc/apt/sources.list```` ja lisäämällä sinne seuraavat rivit:

![Add file: fasttrack](/img/fasttrack.png)

Samassa yhteydessä huomasin, että backports löytyi jo sources.list-tiedostosta, joten poistin aiemmin luomani tiedoston komennolla ````$ sudo rm /etc/apt/sources.list.d/bookwork-backports.list````

Päivitin paketinhallinnan ````$ sudo apt update```` ja kokeilin asentaa VirtualBoxin komennolla ````$ sudo apt install virtualbox````, mutta asennus epäonnistui. Tutkin aiemmin lisäämiäni rivejä kirjoitusvirheiden varalta ja huomasin, että seuraamani ohje oli mahdollisesti tarkoitettu toiselle Debianin versiolle, Bullseyelle. Vaihdoin bullseyen tilalle bookworm, päivitin paketinhallinan ja yritin asennusta uudelleen.

![Add file: bookwormihan se olikin](img/bookworm-fasttrack.png)

Tämän jälkeen asennus onnistui komennolla ````$ sudo apt install virtualbox````. Lopuksi testasin, että Vagrant ja VirtualBox olivat asentuneet komennoilla ````$ vagrant```` ja ````$ virtualbox````. Vagrant palautti komennon käyttöohjeet ja VirtualBox aukesi, joten asennukset olivat onnistuneet.

## b) Yksi maantiekiertäjä

Loin ensin Vagrantille oman kansion kotihakemistooni ````$ mkdir /home/aatu/Vagrant```` ja ajoin siellä komennon ````$ vagrant init debian/bullseye64````, joka virtuaalikoneen asennukseen käytettävän Vagrantfile-nimisen tiedoston. Pystytin virtuaalikoneen komennolla ````$ vagrant up````. Noin minuutin asennuksen jälkeen vagrant ilmoitti asennuksen onnistuneen ja koneeseen pystyi muodostamaan yhteyden komennolla ````$ vagrant ssh````.

![Add file: vagrant pystyssä](/img/vagrant_2.png)

Ajattelin testata nettiyhteyden toimintaa curlilla, mutta sitä ei tullut asennuksen mukana. Google.com vastasi komentoon ````$ ping google.com````, joten virtuaalikoneella oli pääsy internetiin. 

![Add file: vagrant ping](/img/vagrant_ping.png)

## c) Oma orjansa

Saltista ei ole vielä omaa versiota Debian 12:lle, joten suoritin asennuksen samaan tapaan kuin viime viikolla [terokarvinen.com](https://terokarvinen.com/2023/configuration-management-2023-autumn/)-sivuston H1-osan vinkkiosion ohjeiden mukaisesti. Loin tiedoston salt-asennus.sh, johon syötin seuraavat rivit, minkä jälkeen ajoin tiedoston komennolla ````$ bash salt-asennus.sh````. 

    sudo mkdir /etc/apt/keyrings # ei haittaa vaikka sanoisi etta hakemisto on jo
    sudo curl -fsSL -o /etc/apt/keyrings/salt-archive-keyring-2023.gpg https://repo.saltproject.io/salt/py3/debian/11/amd64/SALT-PROJECT-GPG-PUBKEY-2023.gpg
    echo "deb [signed-by=/etc/apt/keyrings/salt-archive-keyring-2023.gpg arch=amd64] https://repo.saltproject.io/salt/py3/debian/11/amd64/latest bullseye main" | sudo tee /etc/apt/sources.list.d/salt.list
    sudo apt-get update
    sudo apt-get install salt-minion

![Add file: minionin asennus)(/img/salt_bookwork_asennus.png)

Testasin orjan asennuksen onnistumisen:

    $ salt-minion --version
    salt-minion 3006.4 (Sulfur) # Versio 3006.4 asennettu

Salt-master piti vielä asentaa. Käytin asennuksessa hyväksi salt-minionin paikallisia komentoja. 

    $ salt-call --local -l info state.single pkg.installed salt-master
    [...]
    Local:
    ----------
          ID: salt-master 
    Function: pkg.installed
      Result: True # toiminnon jälkeen salt-master löytyy
     Comment: The following packages were installed/updated: salt-master # paketti asennettu
     Started: 22:05:19.273632
    Duration: 4399.297 ms
     Changes:   
              ----------
              salt-master:
                  ----------
                  new:
                      3006.4 # asennettu versio
                  old:
                            # ei vanhaa versiota, eli salt-masteria ei aiemmin ollut
    Summary for local
    ------------
    Succeeded: 1 (changed=1) # toiminto onnistunut, yksi muutos kokoonpanoon tehty 
    Failed:    0
    ------------
    Total states run:     1
    Total run time:   4.399 s


Testasin vielä, että salt-master on asentunut komennolla ````$ sudo salt-keys````, joka tulostaa listan orjien avaimista. Lista tulostui ymmärrettävästä syystä vielä tyhjänä, mutta komento kuitenkin toimi. Asennus oli onnistunut. Seuraavaksi jatkoin [Salt Quickstart - Tero Karvinen](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20master) ohjeen mukaisesti kertomaan orjalle, mistä osoitteesta herraa tulee tavoitella muokkaamalla /etc/salt/minion-tiedostoa. 

    $ hostname -I
    192.168.50.100 # koneen ip-osoite
    $ sudoedit /etc/salt/minion 
    master: 192.168.50.100 # tavoiteltava ip-osoite
    id: kotiorja # orjan nimi 
    $ sudo systemctl restart salt-minion

Orjan tulisi uudelleenkäynnistyksen jälkeen tavoitella herraa, mutta näin ei käynyt. ````$ sudo salt-key```` -listaan ei tulostunut kotiorjaa. Päätin kokeilla vaihtaa herran ip-osoitteeksi 127.0.0.1, jos se toimisi paremmin. Ei vaikutusta. Käynnistin vielä molemmat demonit uudelleen, minkä jälkeen kotiorja lähti soittelemaan herralle. 

    $ sudo systemctl restart salt-master
    $ sudo systemctl restart salt-minion
    $ sudo salt-key
    Accepted Keys:
    Denied Keys:
    Unaccepted Keys:
    kotiorja # kotiorja yrittänyt yhteyttä, mutta avain hyväksymättä
    Rejected Keys:
    $ sudo salt-key -A # kotiorjan avaimen hyväksyntä
    
    $ sudo salt '*' test.ping # testataan, vastaako orja
    kotiorja:
        True # vastaa, eli toimii



## d) Herra-orja arkkitehtuuri verkon yli

## e) Idempotentit komennot verkon yli

## f) Grains

## g) Shell-komento verkon yli

## h) Hello IaC
