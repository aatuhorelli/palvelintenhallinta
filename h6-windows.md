# H6 - Windows

Tehtävänannot luettavissa https://terokarvinen.com/2023/configuration-management-2023-autumn/.

Virtualisoinnissa käytetyn isäntälaitteen tietoja:
    
    OS: Debian 12 (Bookworm)
    CPU: 12th Gen Intel(R) Core(TM) i7-1255U (x86_64), 10 ydintä 
    RAM: 16Gt 
    SSD: 512Gt NVME M.2

    Ohjelmien versiot:
    
    Virtualbox: 7.0.12_Debian
    Salt 3006.4 (Sulfur)
    
    

## x) Lue ja tiivistä

### Salt Windowsilla

 - Asennustiedoston [linkki](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/windows.html)
 - Asennus Powershellin kautta
   - Run as administrator
   - Siirry kansioon, johon tiedosto ladattu ``cd C:\~~\``
   - Tiedoston ajaminen ``./tiedoston_nimi.msi # esim ./Salt-Minion-3004.2-1-Py3-AMD64.msi``
   - Asennuksen jälkeen testaus esim. ``salt-call --local cmd.run "echo hello"``

 - Saltin komennot Powershellin kautta samat kuin Linuxilla.
   - Paikalliset komennot ``salt-call --local [komento] # esim. salt-call --local grains.items``
   

Lähde: https://github.com/sannnir/h5-Windows

### Halonen, Rajala ja Ollikainen 2023: Installing Windows 10 on a virtual machine

Artikkelissa on ohjeet Windows 10:n asennukselle Virtuaalikoneelle.

#### Valmistelu ja virtuaalikoneen määrittely:
 - Asennusmedian latauslinkki: https://www.microsoft.com/en-us/evalcenter/download-windows-10-enterprise
   - English (Great Britain) 64-bit, ei LTSC (Long-Term Servicing Channel, ellei tarkoitus käyttää virtuaalikonetta pitkään)
 - Virtualisointi VirtualBoxilla:
   - New -> Versio: Windows 10 -> RAM > 8Gt -> Create
   - Tarpeeksi tallennustilaa, esim. 50Gt
     - Hard disk file type: VDI
     - Dynamically allocated
   - CPU: 4 tai enemmän
   - Käynnistä virtuaalikone ja valitse asennusmediaksi aiemmin ladattu Windows 10 -tiedosto

#### Windows 10 asennus
 - Install now -> Custom: Install Windows only -> valitse kohteeksi virtuaaliasema -> Paikalliset asetukset
 - Mainonnan kohdistamiseen liittyviin kyselyihin ja tietojen jakamiseen vastaukset oman mielen mukaan
 - Sign in with Microsoft -kohdassa valitse Domain join instead oikeasta alalaidasta
 - Nimeä laite, valitse salasana ja aseta turvakysymykset

Lähde: https://github.com/therealhalonen/PhishSticks/blob/master/notes/ollikainen/windows.md

### SB Workgroup, The Linux Foundation 2015: Filesystem Hierarchy Standard

FHS:n tarkoitus on luoda ohjelmille ja käyttäjille looginen tapa järjestellä tiedostoja ja arkistoja. 

Poimintoja hierarkiasta:
 - /: Järjestelmän juurihakemisto. Sisältää vähintään järjestelmän käynnistämiseen, palauttamiseen ja korjaamiseen tarvittavat tiedot.
 - /bin/: Tärkeiden komentojen binäärit
 - /boot/: Bootloaderin staattiset tiedostot
 - /etc/: Järjestelmäasetukset
 - /media/: Irroitettavien medioiden mounttaus
 - /srv/: Järjestelmän palveluiden dataa
 - /tmp/: Väliaikaiset tiedot
 - /usr/: Paikallinen hierarkia
   - /usr/bin/: Useimmat komennot
   - /usr/local/: Paikallisesti asennetuja ohjelmia
 - /var/: Monenlaisia datatiedostoja, kuten lokeja ja väliaikaisia tiedostoja
   - /var/log/: lokitiedostoja

Lähde: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html

## A) Windowsin asennus virtuaalikoneeseen

Koneella oli jo valmiiksi asennettuna VirtualBox, joten sen asennusta en käy erikseen läpi tässä kohdassa.

Aloitin tehtävän lataamalla Halonen, Rajala & Ollikainen [ohjeessa](https://github.com/therealhalonen/PhishSticks/blob/master/notes/ollikainen/windows.md) mainitun Windows 10 -asennustiedoston. Lataus kesti joitain minuutteja, ja sen jälkeen tiedosto löytyi /home/aatuh/Downloads -hakemistosta. Latauksen jälkeen käynnistin VirtualBoxin (``$ virtualbox``) ja loin uuden virtuaalikoneen klikkaamalla "New". Annoin virtuaalikoneen nimeksi Windows-10-orja ja valitsin Version-listasta Windows 10 (64-bit). Aiemmin lukemassani ohjeessa ei tässä vaiheessa vielä mountattu asennusmediaa, joten jätin sen itsekin tekemättä.

![Add file: VirtualBox Windows 10](/img/win10-virtualbox.png)
> VirtualBoxin ensimmäisen sivun asetukset

Tämän jälkeen painoin Next, ja määritin koneen RAM-muistin määräksi 8192MB ja prosessorien määräksi 4 kappaletta. En kuvitellut tekeväni laitteella mitään kovinkaan raskasta, joten oletin pärjääväni neljällä ytimellä. Painoin jälleen Next, käytin seuraavalla sivulla valmiiksi valittua vaihtoehtoa "Create a Virtual Hard Disk Now" ja määritin virtuaalisen aseman kooksi 60Gt. Windows on Linuxia raskaampi käyttöjärjestelmä, enkä halunnut tilan loppuvan yllättäen kesken.

![Add file: Virtualbox cpu & ram](/img/win10-ram-cpu.png)
> RAM ja CPU -asetukset

![Add file: Virtualbox HDD](/img/win10-hdd.png)
> HDD-asetukset

Seuraavaksi painoin Next, tarkastin VirtualBoxin näyttämän yhteenvedon ja todettuani sen vastaavan sitä, mitä kuvittelinkin, painoin Finish. Kone ilmestyi VirtualBoxin etusivulle, josta käynnistin koneen painamalla Start. Käynnistyessään kone valitti, ettei löydä bootattavaa mediaa, ja antoi etsiä levykuvaa tietokoneelta. Valitsin ensimmäisessä vaiheessa lataamani Windows 10 -asennusmedian, minkä jälkeen painoin Mount and Retry Boot. Kone käynnistyi Windowsin asennustyökaluun.

![Add file: Ei boottimediaa](/img/win10-ei-boottaa.png)
> Virheilmoitus ja asennusmedian valinta

Asennusmediassa oli vain englanninkielinen Windows, jonka olisin joka tapauksessa valinnut mahdollisten virheilmoituksen selvittämiseksi. Aika- ja valuutta-, sekä näppäimistöasetuksiin valitsin kuitenkin Finnish. Next -> Install now. Hyväksyin käyttöehdot lukematta niitä, valitsin seuraavasta aukeavasta ikkunasta "Custom: Install Windows only (advanced)" ja valitsin ainoan näkyvän tallennusmedian asennuskohteeksi. 

![Add file: Tonne asennetaan](/img/win10-partitio.png)
> Helppo valinta, tänne asennetaan

Klikkasin Next ja Windows asentui parissa minuutissa ilman virheitä. Tämän jälkeen asennusohjelma käynnisti virtuaalikoneen uudelleen pariin kertaan ja asennus jatkui muutaman minuutin kuluttua Microsoftin kysymyksiin vastailemalla. Ohitan käyttäjään ja paikallisiin asetuksiin liittyvät kysymykset raportissa. Asennusohjelma käynnisti koneen vielä kerran uudelleen ja tarjosi kirjautumista Microsoft-tilille. Valitsin vasemmasta alakulmasta "Domain join instead". 

![Add file: domain join](/img/win10-domain-join.png)
> Valitaan 'Domain join instead'

Seuraavaksi salasanoja, salaisia kysymyksiä ja Microsoftin mainosahdistelua, minkä jälkeen Windows kehoitti olemaan sammuttamatta konetta sen valmistellessa käyttöjärjestelmää muutaman minuutin ajan. Tein työtä käskettyä ja parissa minuutissa Windowsin työpöydä tervehti minua iloisella kilahduksella. Avasin Task Managerin ja kurkistin paljonko resursseja tyhjä Windows kuluttaa, sekä suoritin nettiyhteyden testaamisen Windows Edgen tärkeimmällä tehtävällä: korvaavan selaimen latauksella.

![Add file: task manager](/img/win10-taskmanager.png)
> Käy ja kukkuu, resurssit riittävät toistaiseksi

Lopuksi poistin Windowsin kaikissa asennuksissa ärsyttävästi silmille hyppivät hyväksymisikkunat (Vasemman alakulman hakupalkki -> User account control settings -> Never notify -> OK). 

## B & I) Salt Windowsille & cmd.run

Latasin Saltin 3006.4 -version asennustiedoston ([linkki](https://repo.saltproject.io/salt/py3/windows/latest/Salt-Minion-3006.4-Py3-AMD64-Setup.exe)). Koska tarjolla oli vain .exe-tiedosto, päätin asentaa ohjelman perinteiseen tapaan avaamalla ohjelman graafisen käyttöliittymän kautta. Asennuksen kysellessä herran IP-osoitetta annoin aiempia orjianikin komennelleen palvelimen ip-osoitteen. Tälle orjalle valitsin nimeksi win10-orja ja painoin Install. Asennuksen alkuvaiheissa Salt kyseli, että asennellaanko puuttuva vcredist samassa yhteydessä, minkä hyväksyin. Asennus rullasi muutamassa sekunissa loppuun onnistuneesti.

![Add file: Onnistunut asennus](/img/win10-salt.png)
> Start salt-minion -> Finish

Testasin vielä paikallisesti Salt minionin asentuneen avaamalla Powershellin admin-oikeuksin (tehtäväpalkin haku -> Powershell -> Run as Administrator). Testit suoritin kokeilemalla Linuxista tuttuja komentoja ``salt-call --version`` sekä ``salt-call --local -l info state.single cmd.run "echo toimiiko tää tälleen?"``. Ensimmäinen komento tulosti version 3006.4 (Sulfur) ja jälkimmäinen väitti ainakin tehneensä työtä käskettyä (Succeeded: 1 (changed=1)). 

![Add file: salt-call testit](/img/win10-saltcall-testi.png)
> Salt-call testit. Toimii, mutta ääkkösistä ei näytä juurikaan pitävän.


## C & J) Grains.items

Koska paikalliset testit tuntuivat toimivan, otin ssh-yhteyden komentorivin kautta herrana toimivaan palvelimeen ja tarkastin, onko uudenkarhea orjani soitellut kotiin: ``$ sudo salt-key # Unaccepted Keys: win10-orja``. Tavoitellut oli, joten hyväksyin avaimen komennolla ``$ sudo salt-key -A # Proceed [n/Y] -> Y``. Tämän jälkeen kokeilin vielä, että yhteys on olemassa testaamalla vastaako win10-orja pingiin: ``$ sudo salt 'win10-orja' test.ping``.

![Add file: orja soittelee](/img/win10-orja-avain.png)
> Avain hyväksytty ja orja vastaa pingiin

Orja oli linjoilla, joten grains.items voitiin kätevästi käskyttää herran kautta. Tulosteen käsittelyn helpottamiseksi kirjauduin virtuaalikoneen komentorivissä pyörivästä ssh-yhteydestä ulos komennolla ``$ logout`` ja avasin ssh-yhteyden isäntälaitteen terminaalista. Näin copy-paste olisi helpompaa. Hain Windows-orjan tiedot komennolla ``$ sudo salt 'win10-orja' grains.items``. Rivejä tuli melko paljon. Alla tuloste karsittuna:

    win10-orja:
    ----------
    cpu_model:
        12th Gen Intel(R) Core(TM) i7-1255U    # Sama prosessori, kuin isäntälaitteessa: 12. sukupolven Intel i7
    cpuarch:
        AMD64    # Prosessorin arkkitehtuuri
    cwd:
        C:\Program Files\Salt Project\Salt    # current working directory: komento ajettu tässä sijainnissa
    id:
        win10-orja    # Orjan nimi
    init:
        Windows       # Windows. Menisiköhän Vagrantissa tällä init-nimellä?
    ipv4:
        - 10.0.2.15    # IP-osoite
        - 127.0.0.1    # localhostin IP
    master:
        172.232.xxx.xxx      # Herrapalvelimen osoite
    mem_total:
        8191                 # RAM määrä. Yksikkö megatavua (Mt/MB)
    num_cpus:
        4                    # Prosessoreiden määrä(kpl). Täsmää VirtualBoxissa asetettuihin.
    num_gpus:
        0                    # Ei erillistä näytönohjainta (lkm: 0kpl)
    osfinger:
        Windows-10           # Käyttöjärjestelmä
    osversion:
        10.0.19045           # Windowsin versio
    pending_reboot:
        True                 # Joku asetus ilmeisesti odottaa järjestelmän uudelleenkäynnistystä.
                             # [Lisäys: reboottasin koneen myöhemmin, ja tähän muuttui tilaksi False]
    pythonexecutable:
        C:\Program Files\Salt Project\Salt\Scripts\python.exe # Pythonin sijainti orjalla
    saltpath:
        C:\Program Files\Salt Project\Salt\Lib\site-packages\salt # Saltin sijainti orjalla
    saltversion:
        3006.4               # Saltin versio orjalla. OHerran version oltava sama tai uudempi. 
    shell:
        C:\Windows\system32\cmd.exe    # Orjan käyttämä komentorivi

## D) Saltin file -toiminto Windowsilla

Testasin file.managed-toimintoa herran kautta samaan tyyliin kuin Linux-orjia komentaessa, ja ajattelin antaa mahdollisten virheilmoitusten johdattaa minut oikeille raiteille. Suoritin komennon ``$ sudo salt 'win10-orja' state.single file.managed c:\testi\nakki.txt contents="Olen tekstitiedosto moi"``. Suoritus epäonnistui virheilmoituksella "Specified file c:testinakki.txt is not an absolute path". Tulosteen poluista puuttuivat \\-merkit, joten epäilin niiden toimivan "escapeina", ja käänsin ne toisin päin. Samalla virheilmoitus muuttui: 

    $ sudo salt 'win10-orja' state.single file.managed c:/testi/nakki.txt contents="Olen tekstitiedosto moi"
    win10-orja:
    ----------
              ID: c:/testi/nakki.txt
        Function: file.managed
          Result: False
         Comment: Parent directory not present # kansiota ei löydy
         Started: 00:21:53.482553
        Duration: 15.002 ms
         Changes:   

    Summary for win10-orja
    ------------
    Succeeded: 0
    Failed:    1
    ------------
    Total states run:     1
    Total run time:  15.002 ms
    ERROR: Minions returned with non-zero exit code

File.managed ei ilmeisesti suostu luomaan kansiota. Kävin luomassa sen virtuaalikoneella edelleen auki olevasta powershellistä: ``cd c:\`` & ``mkdir testi``. Siirryin kansioon komennolla ``cd testi`` ja tarkistin sen olevan tyhjä komennolla ``ls``.

![Add file: orjan testikansio](/img/win10-testikansio.png)
> Testikansion luominen powershellin kautta

Kansio oli nyt luotu, joten ajoin aiemman file.managed-komennon uudelleen. 

    $ sudo salt 'win10-orja' state.single file.managed c:/testi/nakki.txt contents="Olen tekstitiedosto moi"
    win10-orja:
    ----------
              ID: c:/testi/nakki.txt
        Function: file.managed
          Result: True
         Comment: File c:/testi/nakki.txt updated # tiedosto päivitetty
         Started: 00:30:51.525961
        Duration: 31.834 ms
         Changes:   
              ----------
                  diff:
                  New file

    Summary for win10-orja
    ------------
    Succeeded: 1 (changed=1)  # Onnistunut ajo, yhtä tiedostoa muutettu
    Failed:    0
    ------------
    Total states run:     1
    Total run time:  31.834 ms

Komennon suorittaminen onnistui, 'result' oli True ja yhteenveto-osiossa kaikki (1) komentoa oli ajettu onnistuneesti. Tarkistin vielä lopuksi orjan powershellistä, että tiedosto oli olemassa ja sisältö vastasi odotuksia. ``ls # nakki.txt löytyy`` & ``cat .\nakki.txt``.

![Add file: nakkia on](/img/win10-file-nakki.png)
> Tiedosto löytyy ja sisältö on täydellinen

## E & F) Uusi toiminto ja Ohjelman asennus Windowsille Saltilla

Koska F-kohdan tehtävänannossa mainittiin ohjelmien asennuksen vaativan alkuasetusten säätöjä, päätin sen olevan hyvä uusi testattava toiminto, jonka toiminnan testaamiseksi asentaisin uuden ohjelman. Etsin Googlesta ohjeita aiheesta ja päädyin [Saltprojectin sivulle](https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html), jonka ohjeiden mukaan lähdin tehtävää edistämään.

Ohjeet pitäisivät olla sinänsä simppelit. Uusien komentojen tehtävä piti tehdä Windowsilla, joten en käskyttänyt tällä kertaa orjaa herran kautta, vaan ajoin ensimmäiset komennot paikallisesti powershellistä. Ymmärsin ohjeista, että ohjelmat ladattaisiin GitHubista, joten [latasin](https://github.com/git-for-windows/git/releases/download/v2.43.0.windows.1/Git-2.43.0-64-bit.exe) ja asensin Gitin Windows-orjalle. Asennusprosessissa ainoa muutos jonka tein vakiovalintoihin oli tekstieditorin valinta, jossa vaihdoin Vimin Nanoon. Toivottavasti tämä riitti tässä vaiheessa.

Käynnistin Powershellin admin-oikeuksilla edellisissä kohdissa mainitulla tavalla ja kloonasin GitHubin winrepon komennolla ``salt-call --local winrepo.update_git_repos``. Sain versiointiin liittyviä varoituksia, mutta ehkä komennon suoritus onnistui siitä huolimatta. Ajoin ohjeiden seuraavan komennon, jonka ymmärsin päivittävän "paketinhallinnan" listat: ``salt-call --local pkg.refresh_db``. 

![Add file: winrepo asennus](/img/win10-winrepo.png)
> Varoituksia tuli, mutta refresh_db onnistui

Siirryin ohjeessa mainitusta linkistä [Saltstackin winrepon GitHub-repositorioon](https://github.com/saltstack/salt-winrepo-ng) silmäilläkseni, mitä asennettavaa sieltä löytyisi. Katselisin varmasti paljon videoita Windows-orjallani, joten testasin vlc:n asennusta komentelemalla orjaa herran kautta: ``$ sudo 'win10-orja' pkg.installed vlc``. Sain virheilmoituksen "'pkg.installed' is not available". Tarkastin vielä uudemman kerran ohjeen ja huomasin käytettävän komennon olevan pkg.install. Kokeilin sitä, vaikka idempotenssia tuolla lienee turha tavoitella. 

    $ sudo salt 'win10-orja' pkg.installed vlc # ensimmäinen epäonnistunut suoritus
    win10-orja:
        'pkg.installed' is not available.
    ERROR: Minions returned with non-zero exit code
    aatu@localhost:~$ sudo salt 'win10-orja' pkg.install vlc # Onnistunut suoritus
    win10-orja:
        ----------
        vlc:
            ----------
            new:
                3.0.18    # Uusi versio. Asennus onnistunut.
            old:

Windows-orjien asennuksissa on näemmä suppeammat tulosteet, mutta asennus onnistui. Tarkistin vielä orjalta, löytyykö VLC:tä. Kuvake oli ilmestynyt työpöydälle, ja ohjelma aukesi ilman ongelmia tuplaklikkaamalla. 

![Add file: VirtuaaliVLC](/img/win10-vlc.png)
> VLC toimii


## Lähteet

https://terokarvinen.com/2023/configuration-management-2023-autumn/
https://github.com/sannnir/h5-Windows
https://github.com/therealhalonen/PhishSticks/blob/master/notes/ollikainen/windows.md
https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html
https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html
