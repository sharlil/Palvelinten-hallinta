# h4 Pizza Fantasia
 
# Sisältö
* [x) Artikkeli](#x-artikkeli)
* [a) Räpylä](#a-rapyla)
* [b) Automaatti](#b-automaatti)
* [c) Asetus](#c-asetus)
* [d) Paikka remonttiin](#d-paikka-remonttiin)
* [e) Idempotentti](#e-idempotentti)




### Koneen tekniset tiedot
* Prosessori: Intel Core i5-8265U CPU @ 1.60 GHz (1.80 GHz turbo, 8 ydintä)
* RAM: 16 GB (15,7 GB käytettävissä)
* Järjestelmä: Windows 11 Pro 64-bittinen (x64-suoritin)
* Näytönohjain: Intel UHD Graphics 620
* Tallennustila: 237 GB, josta 158 GB vapaana
* DirectX-versio: DirectX 12

# x) Artikkeli

### Karvinen 2023: Configuration Management of Distributed Systems over Unreliable and Hostile Networks

#### 4.12.1 Size and Complexity of Some DSLs (112. Ominaisuuksien määrä.)
- DSL-kielet voivat olla monimutkaisia ja laajoja. Karvinen analysoi johtavien konfiguraatiotyökalujen Saltin sekä Puppetin monimutkaisuutta.
- Analysoinnissa Karvinen käytti ja tutki automaattisesti generoituja manuaaleja lähdekoodista.
- Saltilla oli 510 toimintoa, 20 000 riviä dokumentaatiota ja yli 75 000 sanaa. Puppetilla on 113 toimintoa ja se käyttää omia erikoiskäsitteitä ja ohjausrakenteita, toisin kuin perinteisimmät ohjelmointikielet, kuten C, Go tai Python.
- Kysymys: Arvioiko juuri manuaalin koko monimutkaisuutta?

#### 4.12.2 Use of DSL Functions in Case Configuration (112-115. Mitä oikeasti käytetään.)
- Tutkimuksen laajuuden analysointia varten valikoitui United States Government Configuration baseline (NIST,2016) ja Mozilla Release Engineering Puppet Manifests (Mitchell et al., 2020).
- Pieni joukko komentoja kattaa 87% kaikista komennoista ja ohjausrakenteista. Yleisimpinä myös alla mainitut `file`, `package`, `exec` ja `service`.
- Kommentti: Miksi tehdä vaikeasti, kun voi tehdä yksinkertaisesti?


#### 4.12.3.1 Dependencies Between Main Functions (115-117. Tärkeimmät rakennuspalikat.)
- Muutama komento kattaa suurimman osan konfiguraatioista.
- Tärkeimmät perustoiminnot ovat kaiken perusta: `exec`, joka kutsuu muita ohjelmistoja, ja `file` tiedostojen hallintaan joiden pohjalle yllä olevat funktiot rakentuvat
- Kommentti: Miksi sitten muita komentoja on niin paljon, jos tarvitaan vain perustoiminnot?







# a) Räpylä

Lähdin tekemään raporttiosiota 19.4. kello 14.20. Meni hetki valita mieluinen demoni.

# Fail2Ban

Päätin asentaa fail2banin, eli tunkeutumisen estoon kehitetty järjestelmä, joka suojelee Linux-servereitä automaattisilta hyökkäyksiltä.  

Työkalu toimii _jailsien_, eli vankiloiden, avulla. Näitä kutsutaan valvontasäännöiksi tietyille palveluille. 

Sitä voidaan ajatella vartijana, joka tutkii ja tarkistelee palvelimen lokitietoja kellon ympäri ja tutkii merkkejä haitallisesta toiminnasta, kuten epäonnistuneet kirjautumisyritykset, toistuvat yritykset, skannausyritykset, salasanan arvaukset ja niin edelleen. 

Epäilyttävän toiminnan havaitessaan se estää automaattisesti hyökkääjän IP-osoitteen lisäämällä palomuuriin uusia sääntöjä estäen pääsyn tietyksi määräajaksi.

Löysin tähän erittäin selkeän ohjeen (James, J.) ja lähdin etenemään seuraavin askelin.


## Asennetaan Fail2Ban ja tarkistetaan asennuksen onnistuminen

Ennen asennusta tehdään tärkein asia: Päivitetään paketit.

* **`sudo apt-get update`** - päivitetään paketit eli pakettilistat

* **`sudo apt install fail2ban`** - asennetaan fail2ban

* **`fail2ban-client --version`** - katsotaan että asennus on onnistunut tarkistamalla versio

* **`apt-cache policy fail2ban`** - paketin lähde kertoo asennusversion ja repositorion (varasto) lähteen

````
fail2ban:
  Installed: 1.1.0-8
  Candidate: 1.1.0-8
  Version table:
 *** 1.1.0-8 500
        500 http://deb.debian.org/debian trixie/main amd64 Packages
        100 /var/lib/dpkg/status
````

![60](images/60.png)

_Asennus onnistunut ja versio 1.1.0_



## Tarkistetaan Fail2Banin status eli onko aktiivinen

* **`systemctl status fail2ban`** -tarkistetaan tila eli status

![61](images/61.png)

_enabled ja active eli käynnissä jo_

Fail2Ban käynnistyy automaattisesti eikä tässä kohtaa ollut ongelmia, eli se oli käynnissä. 

#### Jos ei olisi ollut käynnissä etenisit seuraavasti:

* **`sudo systemctl enable --now fail2ban`** - komento pakottaa käynnistämään ja pitää käynnissä boottauksen jälkeen.

### UFW asentaminen 

Tässä kohtaa pystyin skippaamaan, sillä minulla oli jo UFW asennettuna. 

Debian 13 käyttää oletuksena nftables-palomuuria, joka toimii fail2banin kanssa automaattisesti.

UFW on vaihtoehtoinen, ei pakollinen, sillä fail2ban itsessään tukee useita palomuuritaustajärjestelmiä.

#### Jos UFW:tä ei ole ja halutaan asentaa:

* **`sudo apt install ufw`** - asennetaan UFW eli Uncomplicated Firewall palomuuri

* **`ufw version tai sudo ufw version`** - tarkistetaan versio jälleen eli onnistuiko asentaminen 

![62](images/62.png)

_ufw versio eli onnistunut asennus_

## Enabloidaan eli otetaan käyttöön UFW

**Palomuuri aktivoitiin ja varmistettiin, että se alkaa automaattisesti, kun Debian-palvelin käynnistyy.**

On tärkeää muistaa, että jos olet kirjautunut SSH-yhteydellä, palomuurin tulee antaa oikeudet OpenSSH:lle. 

* **`sudo ufw allow OpenSSH`** - annetaan lupa SSH-yhteydelle - Olin suoraan konsoliyhteydellä, joten tämä oli itselleni tarpeeton.

* **`sudo ufw enable`** - laitetaan palomuuri päälle

Seuraavaksi tuli ilmoitus:

![63](images/63.png)

_Palomuuri oli päällä_

Fail2Ban oli nyt asennettu ja UFW-palomuuri otettu käyttöön.

## Varmuuskopio Fail2Banin konfiguraatiotiedostoille

**Lähdin luomaan Fail2Banin konfiguraatiotiedostoille varmuuskopiota, jotta tekemäni muutokset eivät katoa pakettipäivitysten aikana.**

**Fail2Ban lukee `.local` -tiedostoja oletuksena ennen `.conf` -tiedostoja, joten muutokset tehtiin sinne.**

Fail2Banin asennuksessa tulee kaksi oletuskonfiguraatiotiedostoa: 

* `/etc/fail2ban/jail.conf` ja `/etc/fail2ban/jail.d/defaults-debian.conf`.

Oli tärkeää ymmärtää, ettei `default.conf` -oletustiedostoja muuteta suoraan. 

Kopiot konfiguraatiotiedostoista luotiin `.local` -päätteellä, jotta omat muutokset säilyisivät. 



## Luodaan jail.local-tiedosto ja kopioidaan sinne konfiguraatiotiedoston sisältö

* **`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`** - luodaan tiedosto ja kopioidaan sudona `cp` komennolla konfiguraatiotiedoston sisältö jail.localiin

* **`ls /etc/fail2ban`** -  tarkistetaan fail2ban-hakemistosta, kopioituiko sisältö onnistuneesti.
  
* **`cat /etc/fail2ban/jail.local`** - hakemistosta katsotaan cat-komennolla tiedoston sisään

![64](images/64.png)

_Konfiguraatiotiedoston sisältö onnistuneesti jail.local -tiedostossa_

## Määritellään omat asetukset jail.local -tiedostoon

#### Asetetaan Fail2Banille bantime -asetukset

**Tällä asetusten muuttamisella pidennetään estoaikaa toistuvien hyökkäysten varalta.**

* **`sudo micro /etc/fail2ban/jail.local`** - muokataan microlla konfiguraatiotiedostoa

* Etsitään kohta `DEFAULT` konfiguraatiotiedostosta ja laitetaan ohjeistuksen mukaisesti siihen alla oleva:

````
## Ban time multipliers
bantime.increment = true
bantime.factor = 2
bantime.formula = ban.Time * (1<<(ban.Count if ban.Count<20 else 20)) * banFactor
````

![65](images/65.png)

_estoajan muutosten asettaminen_

Tässä kohdassa oli tärkeää olla tarkkana, ettei vain copy-pasteta sisältöä. Piti olla tarkkana, mitä muutoksia konfigurointitiedostoon teen.


## IP Whitelist konfiguraatio ja oletusestoaika

**Lisäsin `localhost` IP-osoitteen sallittujen listalle IP Whitelistiin samaan tiedostoon**

* Etsitään samasta konfiguraatiotiedostosta (jail.local eli oma konfiguraatiotiedoston kopio) kohta `ignoreip`

* **`#ignoreip = 127.0.0.1/8 ::1`** - kohta löytyi mutta se ei ollut käytössä

* Poistetaan `#` eli hashtag -merkki rivin alusta eli otetaan asetus käyttöön

Ohjeessa oli sopivasti valmiiksi localhostina käyttäminen esimerkkinä, jota tässä harjoituksessa toteutin.

* Tarkistetaan että `bantime`, `findtime` ja `maxretry` kohdassa on oikeat tiedot eli: 

````
# "bantime" is the number of seconds that a host is banned.
bantime = 10m

# A host is banned if it has generated "maxretry" during the last "findtime" seconds.
findtime = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 5
```` 

Ne näyttivät olevan kunnossa joten siirryin seuraavaan kohtaan


![66](images/66.png)

_IP Whitelistille localhost_

### SSH vankilan käyttöönottoa 

* **`ctrl + F`** - etsin `sshd` asetusten kohdan ja lisäsin ohjeistuksen mukaiset tiedot:

````
[sshd]
maxretry = 3 
bantime = 1h
findtime = 10m
```` 
Tällä konfiguraatiolla estetään IP-osoitteet yhdeksi (1) tunniksi kolmen epäonnistuneen yrityksen jälkeen, jotka tehdään kymmenen minuutin (10) sisällä.

* **`ctrl + S`** - tallennetaan määritetyt muutokset

![67](images/67.png)

_SSH vankilan käyttöönotto_

## Potkaistaan fail2ban käyttöön ja tarkistetaan toiminta

* **`sudo systemctl restart fail2ban`** - Potkaistaan fail2ban käyntiin

* **`sudo fail2ban-client status`** - tarkistetaan toimiiko eli kertoo kaikki aktiiviset vankilat

* **`sudo fail2ban-client status sshd`** -tarkistetaan tietyn vankilan status

* **`sudo fail2ban-client -t`** - tarkistetaan vielä konfiguraation syntaksi

![68](images/68.png)

_sshd vankila aktiivinen, konfiguraatiotesti onnistunut, ei estettyjä osoitteita_ 

# b) Automaatti

Tehtävän osioon siirryin 17:58.

Tutustuin ensin Dhandalan (2026) ohjeeseen, kunnes tajusin, että käytän sitä yhdellä koneella localhostina, enkä kahtena erillisenä. Sain sieltä kuitenkin hyviä viitteitä.

Palasin alkujuurille Ansible Docsiin ja Karvisen (2026) sekä omiin ohjeisiini ja sovittelin main.yml-sisällön.

Lähdin etenemään seuraavasti: 

## Luodaan tiedostolle rooli Ansibleen

* **`mkdir -p roles/fail2ban/tasks`** - luodaan tiedostolle rooli hakemistorakenteeseen Ansibleen
  
## Luodaan tehtävätiedosto ja sisältö

* **`micro roles/fail2ban/tasks/main.yml`** - Lähdetään luomaan tehtävätiedosto ja kirjoitetaan sisältö

**Laitan tähän kohtaan perusteellisen selityksen, jotta minun olisi helpompi ymmärtää, miten main.yml -sisältö rakentuu.**

````
--- #tämä tiedoston alkuun

- name: install fail2ban # tehtävän nimi eli esimerkiksi asentaa fail2ban
  apt: # apt-moduuli, tämä aina tyhjä
    name: fail2ban # mikä asennetaan eli fail2ban
    state: present # varmistaa että asennettu

- name: start fail2ban # toinen tehtävän nimi eli esimerkiksi käynnistää fail2ban
  service: # tähän ei koskaan kirjoteta mitään 
    name: fail2ban # palvelun nimi
    state: started # varmistaa että käynnissä
    enabled: yes # käynnistyy automaattisesti boottauksesta
````

Main.ymlssä ensin:

1.  **`apt`** eli **ensimmäinen tehtävä** joka **asentaa ohjelman**
  
2.  **`service`** - **toinen tehtävä** joka **käynnistää ohjelman, pitää käynnissä automaattisesti**

Tässä menikin pieni tovi hahmottaessa ja kirjoittaessa asiaa. 

Tärkeintä on sisäistää. Eiköhän tätä tule vielä harjoiteltua, mutta tämä perusteellisempi pohdinta ja ohje auttoi kyllä hieman.

Lopuksi vielä yllä oleva sisältö laitettiin `main.yml` YAML-tiedostoon ja tallennettiin `ctrl + S`.

![72](images/72.png)

_Main.yml YAML-tiedoston sisältö_ 

## Roolin lisäys site.yml -tiedostoon Ansiblelle

* **`micro site.yml`** - service rooli lisätään site.yml -tiedoston listalle

* **`ctrl + S ja ctrl + Q`*** - tallennetaan muutokset


![70](images/70.png)

_Onnistunut roolin lisäys_


## Ajetaan playbook eli potkaistaan käyntiin

* **`ansible-playbook site.yml -K`** - Ajetaan playbook

## Virhetilanne: Conflicting action statements: hosts, tasks

![71](images/71.png)

_Virheilmoitus_

Virheilmoitus näytti hyvin, missä virhe oli. 

**Olin virheellisesti laittanut `tasks/main.yml` tiedoston sisälle `hosts` -rivin ja `tasks`-otsikon.**

Sekoitin tässä playbookin ja roolin sisällöt keskenään. Ne kuuluivat `site.yml`:ään, eivät `tasks/main.yml`:ään.

![72](images/72.png)

_Korjattu YAML-tiedoston sisältö_

## Onnistuminen

Vihdoin – onnistuminen! Kello olikin jo tässä vaiheessa 20:01. 

Pessimisti ei pety. Olin tässä kohtaa jo varautunut henkisesti koko päivän opettelemiseen.

Opin tässä osiossa sen, että vaikka Ansible alkaa tuntumaan tutulta, itse `main.yml` YAML-tiedoston sisällön luominen ja muut ovat kuitenkin vielä kesken ja harjoituksia tulee jatkaa.

![73](images/73.png)

_failed 0 ja TASK install ja start fail2ban OK_


# c) Asetus

Lähdin jatkamaan 19:48 seuraavana päivänä. Aiemmin olin asentanut asennustiedoston Ansiblelle. 

Tarkoitukseni oli luoda Ansible-roolin asetustiedosto. Ansible vie tiedoston sisällön masterilta orjalle. 

Kaikki asetukset, joita muutan, vie Ansible muutokseni automaattisesti orjalle (eli toiselle koneelle), joka normaalissa tilanteissa olisi.

## Luodaan files-kansio

* **`mkdir -p roles/fail2ban/files/`** - luodaan kansiorakenne filesille

* **`sudo cp /etc/fail2ban/jail.local roles/fail2ban/files/jail.local`** - luodaan tiedosto `cp` komennolla ja kopioidaan asennustiedoston sisältö luotuun tiedostoon sudona

* **`cat roles/fail2ban/files/jail.local`** - katsoin tiedoston sisälle että sisältö kopioitui oikein

 ## Muutetaan asetustiedostosta jotain pientä

* **`micro roles/fail2ban/files/jail.local`** - avasin luodun asetustiedoston auki ja muutin bantime-arvon

![74](images/74.png)

_bantime muutettu 10 minuutista 5 minuuttiin_

* **`ctrl + S`** - tallennetaan tehty muutos


## Handlers kansio fail2banille ja sisältö eli tehtävä

Handlersilla luodaan tehtävä, joka ajetaan vain jonkin asian muuttuessa, eli uudelleenkäynnistetään fail2ban, kun jokin muutos on tässä tapauksessa tehty.

Katsoin Karvisen (2026) Ansible-ohjeistusta, josta sain suuntaa itse handlersin sisältöön. 

Löytyi onneksi Ansible Community Documentationin Handlers-ohjeesta apua.

* **`rm -r roles/fail2ban/handlers/main.yml`** - poistin virheellisen luodun `main.yml`-kansion (piti luoda handlersille kansio ei main.yml:lle).

Tässäkin kohtaa piti olla tarkkana, sillä `/`merkki main.yml -komennon perässä loi `main.yml` -kansion, eikä tiedoston. Jatkossa pitää olla tarkkana kauttaviivojen kanssa.

* **`mkdir -p roles/fail2ban/handlers`** - luodaan kansiorakenne handlersille
 
* **`micro roles/fail2ban/handlers/main.yml`** - laitetaan handlersin tiedostolle sisältö

![75](images/75.png)

_Handlersille sisältö_


## Main.yml-tiedostoon copy-tehtävä 

Tässä osiossa käytin Karvisen (2026) Apache installed with Ansible - quick notes -ohjetta.

* **`micro roles/fail2ban/tasks/main.yml`** - muokataan tiedostoon copy-tehtävä ja lopuksi tallennus `ctrl + S`
 
````
---
- name: copy jail.local  # tämä voi olla mikä vain eli nimi tehtävälle 
  copy: # copy eli moduulin nimi mitä Ansible käyttää
    dest: "/etc/fail2ban/jail.local" # kohde minne kopioidaan orjakoneella eli kohdekoneella
    src: "jail.local" # mistä otetaan tieto eli masterilta
    owner: "root" # kuka omistaa
    group: "root" # mikä ryhmä omistaa
    mode: "0644" # käyttöoikeudet, 6 = omistaja (root), 4= ryhmä (root), 4= muut käyttäjät
  notify: restart fail2ban
````

![76](images/76.png)

_Copy-tehtävän rakenne main.yml tiedostossa_ 

## Ajetaan playbook

## Virhetilanne: YAML parsing failed 

Virheilmoitus ilmoitti, että on löytynyt sisennysvirhe `handlers/main.yml` -tiedostosta riveiltä 1,2 ja 3 tarkistamalla.

* **`micro roles/fail2ban/handlers/main.yml/`** - mennään muokkaamaan tiedoston sisennystä

![81](images/81.png)

_Virheilmoitus YAML parsing failed_

### Korjaus

![77](images/77.png)

_Ylimääräiset välilyönnit pois, kolme viivaa alkuun ja service samalle riville kuin name_

Potkaisin uudestaan käyntiin playbookin 

* **`ansible-playbook site.yml -K`** - Ajetaan Ansible uudestaan

![78](images/78.png)

_Onnistunut lopputulos_

Kello alkoi olla jo jonkin verran ja sain onnekseni toisella yrityksellä käyntiin onnistuneesti fail2banin Ansiblella.


# d) Paikka remonttiin

Tehtävänä oli rikkoa asetuksia esimerkiksi poistamalla demoni. 

* **`sudo rm /etc/fail2ban/jail.local`** - poistin tiedoston

* **`ls -l /etc/fail2ban/jail.local`** - tarkistin että oli poissa

* **`ansible-playbook site.yml -K`** - Ajetaan Ansible uudestaan

![79](images/79.png)

_Poistin tiedoston, tarkistus ja Ansible ajo_

![80](images/80.png)

_Onnistunut Ansiblen korjaus_


# e) Idempotentti

* **`ansible-playbook site.yml -K`** - Ajetaan Ansible uudestaan

Tässä piti todistaa idempotenssi, joka oli onnistunut.

Saman tilan voi tehdä monta kertaa ja päätyä samaan tilanteeseen. Mitään ei muuttunut.

![83](images/83.png)

_Ei muutoksia. Idempotenssi on onnistunut_


## Pohdinta 

Kello oli tässä kohtaa 22:41 ja tällä kertaa oli hyvä tehdä raportti kahdessa osassa. 

Peruskomennot ja koko kokonaisuuden hahmottaminen vaativat edelleen opettelua kaikin puolin, sillä kaikki on hyvin uutta.

Tehtävä oli muutoin mieluisa ja hyvää harjoitusta.

## Lähteet 

Ansible Docs. Dokumentti. _Connection methods and details._ Luettavissa: https://docs.ansible.com/projects/ansible/latest/inventory_guide/connection_details.html/ Luettu: 19.04.2026.

Ansible Docs. Dokumentti. _Handlers: running operations on change._ Luettavissa: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html/ Luettu: 20.4.2026.

Ansible Docs. Dokumentti. _Getting started._ Luettavissa: https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_playbook.html/ Luettu: 19.04.2026.
 
Dhandala, N. OneUpTime. 2026. Verkkosivu. _How to Use Ansible local Connection Plugin._ Luettavissa: https://oneuptime.com/blog/post/2026-02-21-how-to-use-ansible-local-connection-plugin/view/ Luettu: 19.4.2026

Dhandala, N. OneUpTime. 2026. Verkkosivu. _Ansible configure fail2ban._ Luettavissa: https://oneuptime.com/blog/post/2026-02-21-ansible-configure-fail2ban/view/ Luettu: 19.04.2026.

James, J. Verkkosivu. _How to Install Fail2Ban on Debian (13, 12, 11)._ Luettavissa: https://linuxcapable.com/how-to-install-fail2ban-on-debian-linux/ Luettu: 19.4.2026.

Karvinen, T. 2023. Väitöskirja. _Configuration Management of Distributed Systems over Unreliable and Hostile Networks._ Luettavissa: https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks/ Luettu: 19.04.2026.

Karvinen, T. 2026. Verkkosivu. _Apache installed with Ansible - quick notes._ Luettavissa: https://terokarvinen.com/apache-ansible/ Luettu: 19.04.2026.

Karvinen, T. 2020. Verkkosivu. _Command Line Basics Revisited._ Luettavissa: https://terokarvinen.com/2020/command-line-basics-revisited/ Luettu: 19.04.2026.

Sharifi, L. 2026. Verkkosivu. _h2 Voileipä_. Luettavissa: https://github.com/LilJayyy/Palvelinten-hallinta/blob/main/h2%20Voileip%C3%A4.md/ Luettu: 19.04.2026.

Sharifi, L. 2026. Verkkosivu. _h3 Demoni_. Luettavissa: https://github.com/LilJayyy/Palvelinten-hallinta/blob/main/h3%20Demoni.md/ Luettu 19.04.2026.
