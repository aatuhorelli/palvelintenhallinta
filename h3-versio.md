# H3 - Versio

Tehtävänannot luettavissa osoitteessa https://terokarvinen.com/2023/configuration-management-2023-autumn/.

## a) Online

- Uuden repositorion luominen GitHubiin:
  - Kirjauduin sisälle olemassaolevilla tunnuksillani
  - Klikkasin 'New' vasemman laidan repositoriolistauksesta
    - Annoin repositoriolle nimen "winterrepo", lyhyen kuvauksen englanniksi, valitsin 'Add README file' ja valitsin lisenssiksi GNU General Public License 3. Tämän jälkeen klikkasin 'Create repository'

![Add file: repon tiedot](/img/repo_tiedot.png)
> Repositorion tiedot
  - Luotu repositorio aukesi ja oli valmis käytettäväksi


## b) Dolly
 Aloitin testaamalla, että git ei ole asennettu komennolla ``$ git``. Komentoa ei löytynyt, eli asennus oli vielä tekemättä. Suoritin asennuksen komennoilla ``$ sudo apt-get update # paketinhallinnan listojen päivitys`` ja ``sudo apt-get git -y # git asennus``. Tämän jälkeen loin kotihakemistoon kansion projektit (``$ mkdir projektit``), johon aioin kloonata edellisessä tehtävässä luomani repositorion.

 Kloonaamista varten pyöräytin ``$ ssh-keygen``-komennolla SSH-avaimen. Keygenin kysellessä lisätietoja painoin kaikkiin kohtiin vain enteriä. SSH-avain tallentui tiedostoon /home/aatu/.ssh/id_rsa.pub. Avasin tämän komennolla ``$ cat /home/aatu/.ssh/id_rsa.pub`` ja kopioin avaimen leikepöydälle. Syötin SSH-avaimen GitHubiin klikkaamalla ensin oikean yläkulman pyöreää painiketta -> settings -> SSH and GPG keys -> New SSH key. Annoin avaimelle kuvaavan nimen ja valitsin avaimen tyypiksi authentication key. GitHub kysyi tallentaessa salasanaa, joten syötin sen ja avain oli liitetty käyttäjätunnukseen.

![Add file: ssh key](/img/ssh_avain.png)
> SSH-avain lisätty. Merkkijono sanitoitu pois kuvasta.

Ajoin /home/aatu/projektit -hakemistossa komennon ``$ git clone https://github.com/aatuhorelli/winterrepo.git``. Winterrepo kopioitui onnistuneesti, koska tiedostot uusi hakemisto kopioitui sisältöineen. Siirryin hakemistoon komennolla ``$ cd winterrepo`` ja tarkistin vielä, että README.md:n sisältö oli oikea, ``$ cat README.md``.

![Add file: kopioitu repo](/img/git_clone.png)
> Repositorio kopioitu ja README.md-tiedoston sisältö on oikea.

Loin microlla uuden tiedoston helloworld.py ja kirjoitin sinne simppelin Hello world -ohjelman. Tämän jälkeen ajoin komennot ``$ git add . # lisää hakemiston kaikki tiedostot seurantaan`` ja ``$ git commit # tallentaa muutokset paikallisesti``. Commit-viestissä sähköpostiosoitteeni oli automaattisesti generoitu, joten muokkasin sen tehtävänannon vinkkiosion komennolla ``$ git config --global user.email [emailosoite]`` ja päivitin sen commit-rimpsuun Gitin neuvomalla komennolla ``$ git commit --amend --reset-author``. Jatkoin helloworldin puskemista komennoilla ``$ git pull # hakee muualla tehdyt muutokset repositoriosta`` ja ``$ git push # työntää uudet muutokset repositorioon``. Tämän jälkeen Git kysyi GitHubin käyttäjätunnusta ja salasanaa, joiden syöttämisen jälkeen ilmoitti, että salasanalla kirjautumisen tuki on päättynyt. 

![Add file: push_fail](/img/helloworld.png)
> Commit-viestin sähköpostiosoite väärä

Muistelin, ettei luennolla vastaavaa salasanan kyselyä ollut, joten otin toveri-[ChatGPT](https://chat.openai.com/):n apuun ja kysyin, miten kopioin Debianin terminaalista voimassaolevan repositorion. Selvisi, että olin unohtanut kopioidessa lisätä URLin eteen git@github.com-tunnuksen. Tämän lisääminen jälkeenpäin onnistuisi kuulemma komennolla ``$ git remote set-url origin git@github.com:käyttäjänimi/repositorio.git``. Muokkasin origin-urlia ja tein uuden pushin.

![Add file: push ok](/img/push_onnistui.png)
>Push onnistunut. Tiedosto näkyy weppiliittymässä.

## C) Doh!

Päätin muokata helloworldia: ``$ micro helloworld.py``. Maanantai-aamun paha mieli ja edellisen yön huonot yöunet ottivat minusta vallan ja muutin tekstin painokelvottomaksi. Päätin lähteä työntämään muutoksia repositorioon: ``$ git add .``. Tässä vaiheessa tulin järkiini ja vihelsin pelin poikki ja peruin tehdyt muokkaukset komennolla ``$ git reset --hard``, jolla tiedosto palautettiin takaisin edellisen commitin tilaan. 

![Add file: hard reset](/img/git_reset.png)
>Tiedoston sisältö ennen ja jälkeen resetin

Tiedoston sisältö oli myös weppiliittymästä tarkastellessa alkuperäisessä loistossaan.

## D) Tukki

Tutkin Gitin lokia komennolla ``$ git log --patch # Gitin commit-loki muutosten yhteenvedolla``. 


    commit e8d558c894c3761d47975acd118db7d6a2cbf1ca (HEAD -> main, origin/main, origin/HEAD) # Commitille generoitu tunniste
    Author: Aatu Horelli <aatu.horelli@myy.haaga-helia.fi> # Commitin tekijän nimi ja sähköposti
    Date:   Mon Nov 13 11:11:40 2023 +0200 # Aikaleima

        Add helloworld # Commit message

    diff --git a/helloworld.py b/helloworld.py # tiedoston erojen listaus tästä eteenpäin
    new file mode 100644 # uusi tiedosto
    index 0000000..f1a1813 # jotain indeksitietoja
    --- /dev/null            
    +++ b/helloworld.py 
    @@ -0,0 +1 @@ # Muutosten yhteenveto: 0 riviä poistettu, 1 rivi lisätty
    +print("Hello world!") # lisätty rivi

    commit c82fc97fdf0a864a107a08d7480b755134299e57 # Vastaavat tiedot lisenssitiedoston luomisesta
    [...]
      +                    GNU GENERAL PUBLIC LICENSE # Käytetty lisenssi
      +                       Version 3, 29 June 2007 # Lisenssin versionti ja päiväys
    [...]

## F & G) Yhteistyötä eri käyttöjärjestelmillä

Kone, jolla raporttia kirjoittelin tehtäviä tehdessä päätti Linuxille epäominaisesti kaatua, joten rapotti perustuu prosessista otettuihin screenshotteihin ja ulkomuistiin.

Alteregoni on Apple-fanaatikko, jolla on käytössään ARM-arkkitehtuurin M1 Pro -prosessorilla varustettu MacBook Pro. Käyttöjärjestelmä on macOS Sonoma 14.2. MacBookilla oli git jo asennettuna aiemmin, koska terminaali (cmd + space -> terminal -> enter) tunnisti komennon ``$ git``. 

Kutsuin winterrepon omistajan tunnuksella alteregoni repositorion collaboratoriksi (winterrepo -> Settings -> Collaborators -> Add people). Kutsutun käyttäjätunnuksen sähköpostiin lähti kutsulinkki, joka vei github.comin kirjautumissivun kautta kutsun tarkasteluun ja mahdolliseen hyväksyntään. Kirjauduin sisään toisella käyttäjätunnuksellani ja hyväksyin kutsun painamalla 'Accept invitation'. Tämän jälkeen winterrepo ilmestyi käyttäjän repositoriolistaukseen nimellä aatuhorelli/winterrepo

![Add file: github kutsu](/img/github_kutsu.png)
> Kutsulinkin sisältö


Kutsun hyväksyttyäni kloonasin repositorion MacBookille. Kirjautumisessa käytettävän ssh-avaimen luominen onnistui samalla tavalla kuin Linuxissa: ``$ ssh-keygen # luo avaimen tiedostoon /$HOME/$USER/.ssh/.id_rsa.pub``. Lisäsin alteregon GitHub-tunnukselle SSH-avaimen (Settings -> SSH and GPG Keys -> New SSH key) liittämällä .id_rsa.pub-tiedostoon generoidun avaimen. Tämän jälkeen loin käyttäjän kotihakemistoon hakemiston 'projektit' komennolla ``$ mkdir projektit``ja siirryin sinne komennolla ``$ cd projektit``. Kopioin repositorion komennolla ``$ git clone git@github.com:aatuhorelli/winterrepo.git`` ja SSH:n kysellessä sormenjäljestä vastasin yes. Siirryin kopioituun hakemistoon ja tarkastin sen sisällön: ``$ cd winterrepo`` & ``$ ls``. Tiedostot vaikuttivat olevan oikeita.

![Add file: alterego clone](/img/alterego_clone.png)
> Kansion luonti ja repositorion kopiointiprosessi

Apple-fanaatikon elkein alteregoni koki tarpeelliseksi muokata helloworld.py-tiedoston tulostetta lisäämällä sinne yhden rivin ``$ micro helloworld.py # lisätty tekstieditorilla rivi: print("Created with APPLE MACBOOK PRO!!!!")``. Tämän jälkeen muokkasin ennaltaehkäisevästi alteregon nimen ja sähköpostiosoitteen commitia varten ``$ git config --global user.email "aatu.horelli@gmail.com``& ``$ git config --global user.name "Illeroh Utaa"``. Tein commitin: ``$ git commit # Kuvaus: Add crucial lines to print``, varmistin tiedostojen olevan tuoreimmassa versiossa ``$ git pull # already up to date`` ja siirsin muutokset repositorioon ``$ git push``. 

![Add file: alterego push](/img/alterego_push.png)
> Käyttäjän tietojen syöttyö, commit, pull ja push

Uusi rivi ilmestyi näkyviin myös weppiliittymään. Järkytyin Apple-fanaatikon harkintakyvyn pettämisestä niin paljon, että kävin tiedoston normaaliksi palauttamisen yhteydessä poistamassa myös repositorion muokkausoikeudet. Repositorion omistajan toisen koneen terminaalista ajetut komennot: ``$ git pull # tiedostot ajan tasalle``, ``$ micro helloworld.py # rivin poisto``, ``$ git add . # lisätään tiedostot commitin piiriin``, ``$ git commit # kommentti: Reduce fanaticism``, ``$ git pull # tiedostot ajan tasalle siltä varalta, että muutoksia tehty samanaikaisesti muualla`` ja ``$ git push # korjattu tiedosto paikalleen``.

Käyttäjän muokkausoikeuksien poisto repositoriosta onnistui GitHubin polusta winterrepo -> settings -> collaborators -> remove. 

![Add file: fanaatikon poisto](/img/fanaatikon_poisto.png)
> Apple-käyttäjät pois collaboratoreista

Testasin vielä, mitä git ilmoittaa, jos koitan julkaista muutoksia MacBookin käyttäjällä. Hain tuoreimmat tiedostot komennolla ``$ git pull # 1 file changed, 1 deletion(-)``. Loin uuden tiedoston foo komennolla ``$ touch foo``, minkä jälkeen koitin työntää sen repositorioon ``$ git add .``, ``$ git commit # Add foo``, ``$ git pull`` ja ``$ git push # ERROR: Permission to aatuhorelli/winterrepo.git denied to horaat.``. Fanaatikko oli onnistuneesti laitettu jäähylle.
