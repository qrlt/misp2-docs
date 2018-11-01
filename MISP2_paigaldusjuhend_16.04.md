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
    * [3.3.1. Apache Tomcat + Apache HTTP Server + MISP2 baaspakett](#3.3.1.-apache-tomcat-+-apache-http-server-+-misp2-baaspakett)
    * [3.3.2. MISP2 veebirakendus](#3.3.2.-misp2-veebirakendus)
- [4. Seadistamine](#4.-seadistamine)
    * [4.1. MISP2 Apache veebiserveri HTTPS sertifikaadi seadistamine](#4.1.-misp2-apache-veebiserveri-https-sertifikaadi-seadistamine)
    * [4.2. MISP2 konfiguratsioonifail](#4.2.-misp2-konfiguratsioonifail)
    * [4.3. HTTPS ühenduse konfigureerimine MISP2 rakenduse ja X-tee turvaserveri vahel](#4.3.-https-ühenduse-konfigureerimine-misp2-rakenduse-ja-x-tee-turvaserveri-vahel)
    * [4.4. Mobiil-ID seadistamine](#4.4.-mobiil-id-seadistamine)
        * [4.4.1. Teenuse nimetus](#4.4.1.-teenuse-nimetus)
        * [4.4.2. Serveri sertifikaat](#4.4.2.-serveri-sertifikaat)
    * [4.5. Muud seadistused](#4.5.-muud-seadistused)
        * [4.5.1. Java VM seadistamine](#4.5.1.-java-vm-seadistamine)

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

```
sudo -i
```

### 3.1. MISP2 pakettide nimekirja uuendamine

Konfigureerida MISP2 pakettide repositooriumi asukoht failis */etc/apt/sources.list.d/misp.list*. Järgnev käsk lisab MISP2 repositooriumi aadressi faili: */etc/apt/sources.list.d/misp.list*

```
echo "deb [arch=amd64] http://www.x-tee.ee/packages/live/misp2/debs/packages xenial main">> /etc/apt/sources.list.d/misp.list
```

Et MISP2 paketid ei pärine ametlikust Ubuntu repositooriumist, tuleks pakettide allkirjastamiseks kasutatud avalik võti lisada lubatud võtmete hulka.

```
apt-key adv --keyserver keys.gnupg.net --recv "E775D097"
```

Siis tuleks pakettide nimekiri uuendada käsuga

```
apt-get update
```

### 3.2. MISP2 andmebaasipakett

MISP2 andmebaasipakett xtee-misp2-postgresql paigaldatakse käsuga:

```
apt-get install xtee-misp2-postgresql
```

---

Järgnevalt on ära toodud viimase käsu täitmisel esitatavad küsimused ja nende vastused.
 
Sisestada loodava andmebaasi nimi, vaikimisi jääb selleks „misp2db“:

```
Please provide database name: [misp2db]
```

Järgnevalt sisestada loodava andmebaasi kasutaja nimi, vaikimisi jääb selleks „misp2“:

```
Please provide username for accessing database: [misp2]
```

Paigaldusskript püüab seejärel kirjeldatud baasiga ühendust võtta. Kui see ei õnnestu, antakse kasutajale baasiloomise küsimus, muidu minnakse järgmise sammu juurde. Juhul kui soovitakse luua vastavanimelist baasi, võib anda vaikevastuse (enter), muidu vastata „n“, sellisel juhul lõpetab paigaldusskript veaga. 

```
Are you sure you want to create new database 'misp2db' (y/n)? [y]
```

Vastates eelmisele küsimusele jaatavalt, tuleb järgnevalt sisestada baasikasutaja parool (2 korda):

```
Adding new user misp2
Enter password for new role:
Enter it again:
```

### 3.3. MISP2 rakendus

Paigaldada pakett xtee-misp2-application.

```
apt-get install xtee-misp2-application
```

Sellel paketil on sõltuvused xtee-misp2-base ja xtee-misp2-orbeon pakettidele, millest esimesel on omakorda sõltuvused apache2, libapache2-mod-jk ja tomcat7 pakettidele. Kõik nimetatud paketid installeeritakse automaatselt. 
Installeerimisrakendus küsib kasutajalt rea küsimusi, mis seletatakse lahti järgnevates alapeatükkides.

#### 3.3.1. Apache Tomcat + Apache HTTP Server + MISP2 baaspakett

Järgnevale küsimusele vastamine: kui ID-kaarti hakatakse autentimiseks kasutama, siis: “yes”, sel puhul laetakse SK repositooriumist vajalikud sertifikaadid:

```
Do you want to update SK certificates (y/n)? [y]
```

Paigalduspaketi tegevuste ülevaade:

1. Seadistab Tomcati mälu failis */etc/default/tomcat7*

```
JAVA_OPTS="${JAVA_OPTS} –Xms512m –Xmx512m -XX:MaxPermSize=256m"
```

2. Avab Tomcat AJP connector pordil 8009: eemaldab kommentaarid realt `<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />` Tomcat konfiguratsioonifailis *server.xml*.

3. Keelab pöördumise Tomcati pordile 8080 server.xml konfiguratsioonifailis.

4. Loob mod_jk konfiguratsioonifail ja paigutab kataloogi */etc/apache2/mods-available*(vt kaasasaolev näidisfail: *jk.conf*), lisab vastava lingi kataloogi */etc/apache2/mods-enabled*(nt: a2enmod jk).

5. Aktiveerib järgnevad moodulid: *rewrite (a2enmod rewrite), ssl (a2enmod ssl), headers (a2enmod headers)* ja *proxy_http(a2enmod proxy_http* – automaatselt aktiveeritakse *proxy_http* jaoks vajalik proxy moodul.

6. Loob Apache konfiguratsioonifailis SSL ühendusega *virtualhost’i*.

7. Lubab ainult SSL ühendused: HTTP ühendused suunab HTTPS (443) porti (programmsete päringute korral 4443 porti)

8. Seadistab *mod_jk* mooduli Apache konfiguratsioonifailis

9. Paigaldab HTTPS serveri (genereeritud) sertifikaadid, Eesti ID-kaardi juursertifikaadid ja mobiil-ID turvasertifikaadi.

10. Paigaldab tühistusnimekirjade ja OCSP päringu sertifikaadid

11. Taaskäivitab Apache (apache2ctl restart).

Paigaldatavad konfiguratsiooni failid ja kataloogid:

```
/etc/apache2/sites-available/ssl.conf
/etc/apache2/ssl/
/etc/apache2/ssl/create_server_cert.sh
/etc/apache2/ssl/create_sslproxy_cert.sh
/etc/apache2/ssl/updatecrl.sh
/var/lib/tomcat7/conf/server.xml
```

#### 3.3.2. MISP2 veebirakendus

Järgmisele küsimusele vastata „y“ juhul, kui soovitakse konfigureerida rahvusvahelise (EU) versioonina või „n“ Eesti versiooni jaoks (vt allpoolt vastavate konfiguratsiooniparameetrite seadistatud väärtusi):

```
Do you want to configure as international version (if no, then will be configured as Estonian version)? [y/n] [default: n]:
```

Rahvusvahelise versiooni korral seadistatakse konfiguratsiooni¬parameetrid järgmiselt:

```
languages = en
countries = GB
auth.IDCard=false
auth.certificate=true
xrd.namespace=http://x-road.eu/xsd/x-road.xsd
```

Eesti versiooni korral seadistatakse konfiguratsiooniparameetrid järgmiselt:

```
languages = et
countries = EE
xrd.namespace=http://x-road.ee/xsd/x-road.xsd
```

Järgmistele küsimustele vastates tuleb sisestada MISP2 andmebaasi parameetrid: serveri IP, port, andmebaasi nimi, andmebaasi kasutajanimi ja parool. Üldjuhul sobivad kõigile vaikimisiväärtused, va parool. 

NB! Parameetrid peavad kooskõlas olema xtee-misp2-postgresql paketi paigaldamisel seadistatud parameetritega.

```
Please provide database host IP to be used [default: 127.0.0.1]:
Please provide database port to be used [default: 5432]:
Please provide database name to be used [default: misp2db]:
Please provide username to be communicating with database [default: misp2]:
Please enter password for database user 'misp2':
```

Järgmisele küsimusele vastata „y“ juhul, kui soovitakse seadistada rakenduses Eesti ID-kaardiga seotud tegevusi:

```
Do you want to configure signing and encrypting of Estonian ID-card certificates? [y/n]
```

Eelnevale küsimusele jaatavalt vastates antakse lisaküsimused ID-kaardiga seotud tegevuste kohta:

Sisestada PIN2 – serveripoolseks allkirjastamiseks (eeldab serveris digitempli olemasolu):

```
Please enter PIN2:
```

Vastata „y“ kui on soov soov sisse lülitada krüpteerimist ID-kaardiga:

```
Turn on using of encrypting: [y/n]:
```

Vastata „y“, kui on soov sisselülitada serveripoolset digiallkirjastamist:

```
Turn on using of digital signing: [y/n]:
```

Järgmisele küsimusele Vastata „y“, kui on soov sisse lülitada mobiil-ID autentimist serveris (eeldab ka vastava teenuselepingu olemasolu):

```
Do you want to enable authentication with Mobile-ID? [y/n]
```

Vastates eelmisele küsimusele jaatavalt, tuleb sisestada ka mobiil-ID teenuse nimetus:

```
Please provide Mobile-ID service name:
```

Järgnevalt küsitakse e-posti saatmise seadstamiseks vajalikke parameetreid (SMTP serveri aadress, MISP2 poolt kasutatav meiliaadress)

```
Please provide SMTP host address [default: smtp.domain.ee]: 
Please provide server email address: [default: info@domain.ee]:
```

Rahvusvahelise versiooni korral küsitakse kasutajalt veel X-tee versiooni 6 parameetreid: liikmeklasse ja X-tee instantse, mis mõlemad antakse komaga eraldatud loendina.

```
Please provide x-road v6 instances (comma separated list)? [default: eu-dev,eu-test,eu]
Please provide x-road v6 member classes (comma separated list)? [default: COM,NGO,GOV]
```

Pärast konfiguratsiooni failide loomist palutakse luua administraatori konto, soovitav on jätta vaikimisi vastus „jah“. Vastasel juhul ei looda administraatori kontot, millega saab administraatori liidese kaudu portaali seadistama hakata.

```
Do you want to add new administrator account? [y/n] [default: y]
```

Administraatori konto loomise järel küsitakse, millistelt IP-aadressitelt lubada juurdepääs administreerimisliidesele. Kui rakenduse administraator on sisse loginud üle SSH, pakutakse vaikimisi variandina vastava SSH sessiooni kliendipoolset IP-aadressi. Rakenduse administraator peaks tegema kindlaks, kas sellelt IP-aadressilt võib rakenduse administreerimisliidest kasutada. Juhul kui võib, saab valida vaikimisi vastuse. Vastasel juhul tuleks sisestada sobiv IP aadress. Sisestada võib ka tühikutega eraldatult mitu IP-aadressi. (X.X.X.X tähistab kasutaja SSH sessiooni kliendipoolset aadressi).

```
IP address from which administrator interface can be accessed  is currently '127.0.0.1' in /etc/apache2/sites-available/ssl.conf.
User remote IP is 'X.X.X.X'.
Please provide IP address(es) allowed to access administrator interface: [default: X.X.X.X]
```

Administreerimisliidesele ligi pääsevaid IP-aadresse saab hiljem muuta Apache 2 konfiguratsioonifailist (vt peatükk 5.1).

Järgmisele küsimusele vastata „y“ juhul, kui soovitakse paigaldamisel seadistada HTTPS ühendus MISP2 rakenduse ja X-tee turvaserveri vahel:

```
Do you want to enable HTTPS connection between MISP2 application and security server? [y/n] [default: n]
```

Kui eeltoodud küsimusele vastati „y“, siis järgmiseks sammuks HTTPS seadistamisel on vaja eksportida turvaserverist turvaserveri sertifikaadifail sertifikaadifail certs.tar.gz (vt X-tee 6 turvaserveri kasutusjuhendis peatükki: *„9 Communication with the Client Information Systems“*) ja ja kopeerida see MISP2 serverisse kausta */usr/xtee/apache2/*. Sertifikaadifaili nimi X-tee v6 turvaserveri puhul on *certs.tar.gz* ja X-tee v5 turvaserveri puhul *proxycert.tar.gz* – toetatud on mõlemad formaadid.

Paigaldusskript kontrollib eelnimetatud sertifikaadifaili olemasolu ning selle puudumisel palub selle kopeerida õigesse asukohta või katkestada HTTPS seadistamise all oleva küsimusega. Kui fail on kopeeritud MISP2 serverisse, tuleb vastata „y“ või katkestamise soovi korral vastata „n“. Viimasel juhul võib HTTPS seadistamise käivitada hiljem uuesti nagu kirjeldatud peatükis 4.3.

```
Please add Security Server certificate archive 'certs.tar.gz' to the MISP2 server directory '/usr/xtee/apache2/'

Proceed with HTTPS configuration? (Answering 'no' means that HTTPS configuration will not be done this time) [y/n] [default: n]
```

Seejärel tuleb sisestada sertifikaadihoidlate *truststore* ning *keystore* jaoks paroolid, mis on vähemalt 6 sümbolit pikad. Sissetrükitud paroole sisestamisel ei kuvata.

```
Enter truststore password:
Enter keystore password:
```

Turvaserveri sertifikaadi lisamisel truststore sertifikaadihoidlasse küsitakse kasutajalt täiendavalt, kas ta usaldab turvaserveri sertifkaati. Vastata tuleks yes.

```
Trust this certificate? [no]:  yes
```

Seejärel tekitab paigaldusskript MISP2 sertifikaadifaili ja kuvab selle asukoha kasutajale.

```
Get '/usr/xtee/app/cert.cer' and add it to your Security Server.
```

Sertifikaadifail sellest asukohast (antud näites /usr/xtee/app/cert.cer) tuleb laadida turvaserverisse.

Samuti tuleb määrata turvaserveris infosüsteemi serveri ühendusviisiks HTTPS (vt täpsemalt turvaserveri kasutusjuhend).

Peale veebirakenduse paigaldamist võib asuda MISP2 portaali seadistama läbi administraatori veebiliidese nagu kirjeldatud käesoleva juhendi punktis 5.1.

Toodangukeskkonnas peaks paigaldama Apache HTTP serverile ka asutuse sertifikaadi HTTPS ühenduse kasutamiseks nagu kirjeldatud juhendi punktis 4.1.

## 4. Seadistamine

### 4.1. MISP2 Apache veebiserveri HTTPS sertifikaadi seadistamine

Algsel paigaldusel luuakse Apache HTTP serverile iseallkirjastatud sertifikaat, mis on toodangukeskkonnas soovitatav asendada korrektse CA poolt valjastatud sertifikaadiga.

Vaikimisi on Apachel kasutusel järgmised sertifikaadifailid:

```
SSLCertificateFile /etc/apache2/ssl/httpsd.cert
SSLCertificateKeyFile /etc/apache2/ssl/httpsd.key
```

Seega oleks soovitatav oma sertifikaadifailid nimetada samade nimedega ja asendada nendega algsed failid (kuna MISP2 uuendamisel kopeeritakse Apache konfifail üle, pole seal soovitatav muudatusi teha, sest need võivad kaduma minna). Oma sertifikaadifailile(*httpsd.cert*) tuleks lisada lõppu ka paigaldusel genereeritud DH-parameetrite faili(*/etc/apache2/ssl/dhparams.pem*) sisu.

### 4.2. MISP2 konfiguratsioonifail

MISP2 paketi paigaldusskript seadistab andmebaasiühenduse ja ka muud parameetrid konfiguratsioonifailis *config.cfg*. Peale paigaldamist võib mõningaid parameetreid vajadusel muuta konfiguratsioonifailis, mille asukoht on vaikimisi:
`/var/lib/tomcat7/webapps/misp2/WEB-INF/classes/config.cfg`

Järgnevalt on välja toodud mõningad parameetrid, mis küll paigaldamisel seadistatakse automaatselt, kuid mille muutmise vajadus võib ka hiljem tekkida rakenduse ümberseadistamisel.

Peale konfiguratsioonifaili muutmist on vaja muudatuste jõustumiseks ka tomcat alati taaskäivitada näiteks käsuga:

```
service tomcat7 restart
```

Andmebaasiserveriga ühenduse loomise parameetrid:

```
#DB Info – andmebaasi serveri ja kasutaja parameetrid
jdbc.driver=org.postgresql.Driver
jdbc.url=jdbc:postgresql://IP/BAASINIMI
jdbc.username=USERNAME
jdbc.password=PASSWORD
jdbc.databasePlatform=org.hibernate.dialect.PostgreSQLDialect
```

Keelte parameeter ja riikide parameeter:

```
#Languages to which user is allowed to switch and in which can descriptions be set for different elements. Defined in http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes

#If no suitable languages are defined, then uses system default locale language
languages = et

#Countries which can be set for user's country. Defined in http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2

#If no suitable countries are defined, then uses system default locale country
countries = EE
```

Serveri ID-kaardi parameetrid (serveripoolseks digiallkirjastamiseks ning krüpteerimiseks):

```
# ID Card and its usage settings
digidoc.config_file=jar://JDigiDocID.cfg
digidoc.PIN2=01497
email.allow.sign_query=false
email.allow.encrypt_query=false
```

E-posti serveri parameetrid:

```
email.host = mailserver.domain.ee
email.sender.name = MISP2 Support
email.sender.email = info@asutus.ee
```

Mobiil-ID autentimise seadistamise parameetrid:

```
# Mobile ID and its usage settings
mobileID.digidocServiceURL = https://digidocservice.sk.ee/
mobileID.serviceName = Testimine
```

### 4.3. HTTPS ühenduse konfigureerimine MISP2 rakenduse ja X-tee turvaserveri vahel

HTTPS seadistamise sammud on järgmised:

1.	Eksportida turvaserverist turvaserveri sertifikaadifail certs.tar.gz (vt X-tee 6 turvaserveri kasutusjuhendis peatükki: *„9 Communication with the Client Information Systems“*) ja kopeerida see MISP2 serverisse kataloogi */usr/xtee/apache2/*.

Sertifikaadifaili nimi X-tee v6 turvaserveri puhul on *certs.tar.gz* ja X-tee v5 turvaserveri puhul *proxycert.tar.gz* – toetatud on mõlemad formaadid.

2.	Käivitada MISP2 serveris käsurealt seadistusskript:

```
/usr/xtee/app/create_https_certs_security_server.sh
```

Nimetatud skripti käivitamisel kasutajale esitatud küsimused ja vastusevariandid on täpsemalt kommenteeritud käesoleva juhendi veebirakenduse paigaldamise peatükis.
Skript kontrollib kas turvaserveri sertifikaati sisaldav *certs.tar.gz* või *proxycert.tar.gz* arhiiv on MISP2 serveri kaustas `/usr/xtee/apache2/`, loob võtmehoidla *misp2truststore.jks*, genereerib sertifikaadi turvaserveriga sidepidamiseks, paigaldab saadud privaatvõtme ja sertifikaadi võtmehoidlasse *misp2keystore.jks*, loob võtmehoidla ja impordib saadud PKCS12 faili, määrab MISP2 serveris vajalikud süsteemsed parameetrid Tomcati konfiguratsioonifailis `/etc/default/tomcat7` ning teeb Tomcatile restardi.

3.	Määrata turvaserveris infosüsteemi serveri ühendusviisiks HTTPS ja laadida punktis 2 genereeritud sertifikaat (X-tee v6 puhul `/usr/xtee/app/cert.cer`) turvaserverisse (vt täpsemaid juhiseid turvaserveri kasutusjuhendist). 

4.	MISP2 portaali administreerimise liideses muuta „Asutuse turvaserveri aadress“ ja „Päringute saatmise aadress“ väljades protokoll HTTP→HTTPS. 

Kui turvaserveri IP või domeeninimi on SEC_SERVER_IP, siis asendada

```
http://SEC_SERVER_IP -> https:// SEC_SERVER_IP

http:// SEC_SERVER_IP /cgi-bin/consumer_proxy -> https:// SEC_SERVER_IP /cgi-bin/consumer_proxy
```

## 4.4. Mobiil-ID seadistamine

### 4.4.1. Teenuse nimetus

Kindlasti tuleb seadistada konfiguratsioonifailis õige väärtusega parameeter *mobileID.serviceName*. Konkreetse teenusenime väärtuse väljastab igale asutusele Sertifitseerimiskeskus.

### 4.4.2. Serveri sertifikaat

Laadida alla [digidocservice serveri sertifikaat](https://www.sk.ee/upload/files/digidocservice.sk.ee_2016.pem.crt) ning nimetada see *digidocservice.cer*.

Kui HTTPS ühenduse konfigureerimine MISP2 veebirakenduse ja turvaserveri vahel on tehtud, siis võtmehoidla *misp2truststore.jks* on juba olemas. Antud sertifikaadi saab lisada võtmehoidlasse järgmise käsuga:

```
keytool -import -keystore misp2truststore.jks -file digidocservice.cer –alias digidocservice
```

Kui seda pole veel tehtud, siis seadistada *misp2truststore.jks* fail Tomcati konfiguratsioonifailis `/etc/default/tomcat7`

```
JAVA_OPTS="${JAVA_OPTS} -Djavax.net.ssl.trustStore=<loodud misp2truststore.jks faili asukoht> -Djavax.net.ssl.trustStorePassword=<misp2truststore.jks parool>
```

## 4.5 Muud seadistused

### 4.5.1. Java VM seadistamine

Vajadusel võib muuta java süsteemseid parameetreid failis `/etc/default/tomcat7`.

Mälukasutuse parameetrid seadistatakse paigaldusskripti poolt, kuid vajadusel peab antud väärtusi suurendama, näiteks:

```
JAVA_OPTS="${JAVA_OPTS} –Xms2048m –Xmx2048m-XX:MaxPermSize=256m"
```