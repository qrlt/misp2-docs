# MISP2 paigaldus- ja seadistusjuhend

Versioon 2.12



![sf-logo](img/sf.png)




## Sisukord

- [Sisukord](#sisukord)
- [1. Sissejuhatus](#1.-sissejuhatus)
- [2. Nõuded keskkonnale](#2.-nõuded-keskkonnale)
- [3. MISP2 paigaldamine](#3.-misp2-paigaldamine)
    * [3.1. MISP2 pakettide nimekirja uuendamine](#3.1.-misp2-pakettide-nimekirja-uuendamine)
    * [3.2. MISP2 andmebaasipakett](#3.2.-misp2-andmebaasipakett)
    * [3.3. MISP2 rakendus](#3.3.-misp2-rakendus)
    * [3.3.1. Apache Tomcat + Apache HTTP Server + MISP2 baaspakett](#3.3.1.-Apache-Tomcat-+-Apache-HTTP-Server-+-MISP2-baaspakett)
    * []()
    * []()

## 1. Sissejuhatus

Käesolev dokument kirjeldab MISP2 rakenduse paigaldamist ning seadistamist.

## 2. Nõuded keskkonnale

- Toetatud operatsioonisüsteem: Ubuntu Server 16.04 Long-Term Support (LTS), 64-bitine. Veel toetatakse ka  vanemat versiooni 14.04 LTS, millele paigaldamist käesolevas juhendis ei kaeta.
- Vajalik ühendus X-tee turvaserveriga (siseliidesega), milles on seadistatud X-tee asutus, millena MISP2 tegutseb X-teel
- Riistvara soovituslikud parameetrid: 64 bitine protsessor, 4GB RAM
- Vajalik tingimuslikult: 
    * OCSP kehtivuskinnituse teenuse kasutamise leping Sertifitseerimiskeskusega, kui on soov võimaldada päringutulemuste allkirjastamist MISP2 veebirakenduses ning ID-kaardiga autenditud kasutajate sertifikaatide kontrolli OCSP kaudu.
    * OCSP responder sertifikaat OCSP vastuse signatuuri kontrolliks.

## 3. MISP2 paigaldamine

Käesolevas peatükis kirjeldatakse MISP2 portaalikomponentide paigaldamist.
Paigaldamine eeldab, et kasutajal on süsteemis olemas *root* õigused, mille saamiseks tuleb käivitada käsurealt järgnev käsk:

```bash
sudo -i
```

### 3.1. MISP2 pakettide nimekirja uuendamine

Konfigureerida MISP2 pakettide repositooriumi asukoht failis */etc/apt/sources.list.d/misp.list*. Järgnev käsk lisab MISP2 repositooriumi aadressi faili: */etc/apt/sources.list.d/misp.list*

```bash
echo "deb [arch=amd64] http://www.x-tee.ee/packages/live/misp2/debs/packages xenial main">> /etc/apt/sources.list.d/misp.list
```

Et MISP2 paketid ei pärine ametlikust Ubuntu repositooriumist, tuleks pakettide allkirjastamiseks kasutatud avalik võti lisada lubatud võtmete hulka.

```bash
apt-key adv --keyserver keys.gnupg.net --recv "E775D097"
```

Siis tuleks pakettide nimekiri uuendada käsuga

```bash
apt-get update
```

### 3.2. MISP2 andmebaasipakett

MISP2 andmebaasipakett xtee-misp2-postgresql paigaldatakse käsuga:

```bash
apt-get install xtee-misp2-postgresql
```

---

Järgnevalt on ära toodud viimase käsu täitmisel esitatavad küsimused ja nende vastused.
 
Sisestada loodava andmebaasi nimi, vaikimisi jääb selleks „misp2db“:

```bash
Please provide database name: [misp2db]
```

Järgnevalt sisestada loodava andmebaasi kasutaja nimi, vaikimisi jääb selleks „misp2“:

```bash
Please provide username for accessing database: [misp2]
```

Paigaldusskript püüab seejärel kirjeldatud baasiga ühendust võtta. Kui see ei õnnestu, antakse kasutajale baasiloomise küsimus, muidu minnakse järgmise sammu juurde. Juhul kui soovitakse luua vastavanimelist baasi, võib anda vaikevastuse (enter), muidu vastata „n“, sellisel juhul lõpetab paigaldusskript veaga. 

```bash
Are you sure you want to create new database 'misp2db' (y/n)? [y]
```

Vastates eelmisele küsimusele jaatavalt, tuleb järgnevalt sisestada baasikasutaja parool (2 korda):

```bash
Adding new user misp2
Enter password for new role:
Enter it again:
```

### 3.3. MISP2 rakendus

Paigaldada pakett xtee-misp2-application.

```bash
apt-get install xtee-misp2-application
```

Sellel paketil on sõltuvused xtee-misp2-base ja xtee-misp2-orbeon pakettidele, millest esimesel on omakorda sõltuvused apache2, libapache2-mod-jk ja tomcat7 pakettidele. Kõik nimetatud paketid installeeritakse automaatselt. 
Installeerimisrakendus küsib kasutajalt rea küsimusi, mis seletatakse lahti järgnevates alapeatükkides.

#### 3.3.1. Apache Tomcat + Apache HTTP Server + MISP2 baaspakett