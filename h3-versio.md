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
 Aloitin testaamalla, että git ei ole asennettu komennolla ``$ git``. Komentoa ei löytynyt, eli asennus oli vielä tekemättä. Suoritin asennuksen komennoilla ``$ sudo apt-get update # paketinhallinnan listojen päivitys`` ja ``sudo apt-get git -y # git asennus``. Tämän jälkeen loin kotihakemistoon kansion projektit, johon aioin kloonata edellisessä tehtävässä luomani repositorion.

 Kloonaamista varten pyöräytin ``$ ssh-keygen``-komennolla SSH-avaimen. Keygenin kysellessä lisätietoja painoin kaikkiin kohtiin vain enteriä. SSH-avain tallentui tiedostoon /home/aatu/.ssh/id_rsa.pub. Avasin tämän komennolla ``$ cat /home/aatu/.ssh/id_rsa.pub`` ja kopioin avaimen leikepöydälle. Syötin SSH-avaimen GitHubiin klikkaamalla ensin oikean yläkulman pyöreää painiketta -> settings -> SSH and PGP keys -> New SSH key. Annoin avaimelle kuvaavan nimen ja valitsin avaimen tyypiksi authentication key. GitHub kysyi tallentaessa salasanaa, joten syötin sen ja avain oli liitetty käyttäjätunnukseen.

![Add file: ssh key](/img/ssh_avain.png)
> SSH-avain lisätty. Merkkijono sanitoitu pois kuvasta.

Ajoin /home/aatu/projektit -hakemistossa komennon ``$ git clone https://github.com/aatuhorelli/winterrepo.git``. Winterrepo kopioitui onnistuneesti. Siirryin hakemistoon komennolla ``$ cd winterrepo`` ja tarkistin vielä, että README.md:n sisältö oli oikea, ``$ cat README.md``.

![Add file: kopioitu repo](/img/git_clone.png)
> Repositorio kopioitu ja README.md-tiedoston sisältö on oikea.

Loin microlla uuden tiedoston helloworld.py ja kirjoitin sinne simppelin Hello world -ohjelman. Tämän jälkeen ajoin komennot ``$ git add . # lisää hakemiston kaikki tiedostot seurantaan`` ja ``$ git commit # tallentaa muutokset paikallisesti``. Commit-viestissä sähköpostiosoitteeni oli automaattisesti generoitu, joten muokkasin sen tehtävänannon vinkkiosion komennolla ``$ git config --global user.email [emailosoite]`` ja päivitin sen commit-rimpsuun Gitin neuvomalla komennolla ``$ git commit --amend --reset-author``. Jatkoin helloworldin puskemista komennoilla ``$ git pull # hakee muualla tehdyt muutokset repositoriosta`` ja ``$ git push # työntää uudet muutokset repositorioon``. Tämän jälkeen Git kysyi GitHubin käyttäjätunnusta ja salasanaa, joiden syöttämisen jälkeen ilmoitti, että salasanalla kirjautumisen tuki on päättynyt. 

![Add file: push_fail](/img/helloworld.png)
> Commit-viestin sähköpostiosoite väärä

Muistelin, ettei luennolla vastaavaa salasanan kyselyä ollut, joten otin toveri-ChatGPT:n apuun ja kysyin, miten kopioin Debianin terminaalista voimassaolevan repositorion. Selvisi, että olin unohtanut kopioidessa lisätä URLin eteen git@github.com-tunnuksen. Tämän lisääminen jälkeenpäin onnistuisi kuulemma komennolla ``$ git remote set-url origin git@github.com:käyttäjänimi/repositorio.git``. Muokkasin origin-urlia ja tein uuden pushin.

![Add file: push ok](/img/push_onnistui.png)
>Push onnistunut. Tiedosto näkyy weppiliittymässä.
