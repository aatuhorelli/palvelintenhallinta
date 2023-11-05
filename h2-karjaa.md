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


    $ sudo apt install fasttrack-archive-keyring

Lisäsin Fast Trackin paketinhallintaan avaamalla sources.list-tiedoston komennolla ````$ sudoedit /etc/apt/sources.list```` ja lisäämällä sinne seuraavat rivit:

![Add file: fasttrack](/img/fasttrack.png)

Samassa yhteydessä huomasin, että backports löytyi jo sources.list-tiedostosta, joten poistin aiemmin luomani tiedoston komennolla ````$ sudo rm /etc/apt/sources.list.d/bookwork-backports.list````

Päivitin paketinhallinnan ````$ sudo apt update```` ja kokeilin asentaa VirtualBoxin komennolla ````$ sudo apt install virtualbox````, mutta asennus epäonnistui. Tutkin aiemmin lisäämiäni rivejä kirjoitusvirheiden varalta ja huomasin, että seuraamani ohje oli mahdollisesti tarkoitettu toiselle Debianin versiolle, Bullseyelle. Vaihdoin bullseyen tilalle bookworm, päivitin paketinhallinan ja yritin asennusta uudelleen.

![Add file: bookwormihan se olikin](img/bookworm-fasttrack.png)

Tämän jälkeen asennus onnistui komennolla ````$ sudo apt install virtualbox````. Lopuksi testasin, että Vagrant ja VirtualBox olivat asentuneet komennoilla ````$ vagrant```` ja ````$ virtualbox````. Vagrant palautti komennon käyttöohjeet ja VirtualBox aukesi, joten asennukset olivat onnistuneet.

## b) Yksi maantiekiertäjä

## c) Oma orjansa

## d) Herra-orja arkkitehtuuri verkon yli

## e) Idempotentit komennot verkon yli

## f) Grains

## g) Shell-komento verkon yli

## h) Hello IaC
