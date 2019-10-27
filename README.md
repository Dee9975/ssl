# Darbības ar sertifikātiem, atslēgām un sertifikācijas autoritāti
Autors: `Matīss Kalniņš P2-4` 10/26/2019

Openssl sertifikāta izveidošanas dokumentācija.
Katra izmantotā komanda tiks detalizēti aprakstīta, lai arī iesācēji varētu sekot līdzi.

Lai sāktu darbu nepieciešams:
  - Debian distributīvs (debian/ubuntu)
  - Pieeja internetam
  - "Root" pieeja
  
## 1) Atslēgu ģenerēšana un openssl uzstādīšana
###### Sāksim ar openssl uzinstalēšanu

```
apt-get install openssl
```
`apt` : (Advanced Package Tool) debian iebūvētais pakotņu pārvaldieks (package manager)

`openssl` : Pakotne, kas paredzēta ssl sertifikātu ģenerēšanai.

###### Kad openssl ir uzinstalēts, jāizveido datne, kurā turēt nepieciešamos datus.

```
cd /
mkdir ca
cd ca
mkdir newcerts certs crl private requests
touch /ca/index.txt
touch /ca/serial
echo ‘01’ > /ca/serial
```

`cd <directory>` : (Change directory) šī komanda paredzēta, lai pārvietotos pa datnēm

`/` : root datne

`mkdir <directory name>` : Šī komanda izveido jaunu datni, lai ietaupītu laiku, vienā rindā var nodefinēt vairākas datnes, atdalot tās ar atstarpēm.

`touch <file name or location>` : Šī komanda izveido jaunu failu, ja norāda tikai nosaukumu, tad to izveido vietā, kur lietotājs pašreiz atrodas, ja norāda atrašanās vietu, tad fails tiks ievietots noradītajā datnē.

`echo <value> <write to file[optional]> <file[optional]>` : Echo komanda ir paredzēta, lai komandrindā varētu izvadīt tekstu vai lai no tās ierakstīt tekstu failā(ja tas tiek norādīts, `>` nozīmē pārrakstīt failu ar doto vērtību, ja ir vēlme pievienot vērtibu nepārrakstot failu jāizmanto: `>>`).

###### Kad datnes ir izveidotas, laiks pievienot noklusējuma konfigurācijas failu ca datnē:

```
cp /etc/ssl/openssl.cnf /ca/
```

`cp <file to copy> <location to copy to>` : `cp` izmanto lai kopētu failu no vienas vietas uz citu.

###### Nākošais solis ir rediģēt tikko pārkopēto konfigurāciju

```
nano /ca/openssl.cnf
```

`nano <file to edit>` : nano ir debian iebūvētais teksta redaktors (text editor)

Pēc komandas ievadīšanas, ap 45. rindu atradīsies rinda:

```
dir		= ./demoCA		# Where everything is kept
```

Šo vērtību nomainīt uz:

```
dir = /ca		# Where everything is kept
```

###### Nākošais solis būs ģenerēt pašu atslēgu: 
```
openssl genrsa -out /ca/private/cakey.pem 4096
```

`openssl` : Pakotne ko uzinstalējām pašā sākumā

`genrsa` : Parametrs, ko padod, ja vēlās ģenerēt rsa atslēgu

`-out filename` : Parametrs, ko padod, lai izvadītu ģenerēto atslēgu uz norādīto failu

`4096` : Atslēgas izmērs bitos (sākuma izmērs ir 2048, atslēga nedrīkst būt mazāka par 512 bitiem)

###### Izveidojam pašparakstītu CA sertifikātu: 

```
openssl req -new -x509 –config /ca/openssl.cnf -key /ca/private/cakey.pem -out ../cacert.pem
```

`req` : kommanda, kas izveido un apstrādā sertifikāta pieprasījumus `PKCS#10` formātā, vai mūsu gadījumā izveidot pašparakstītu sertifikātu.

`-new` : jauns sertifikāts

`-x509` : parametrs, kas norāda, ka jāveido pašparakstīts sertifikāts

`-config <filename>` : norāda citu konfigurācijas failu

`-key <filename>` : norāda failu, no kura lasīt atslēgu

`-out <filename>` : norāda vietu, kur saglabāt uzģenerēto sertifikātu

Ja komanda ievadīta korekti, parādīsies ievades lauki, kurus jāizpilda (šos laukus aizpildīt pēc saviem datiem):

```
Country Name (2 letter code) [AU]: HU
State or Province Name (Full name) [Some-State]: HU
Locality Name (eg, city) []: Budapest
Organization Name (eg, company) [Internet Widgits Pty Ltd]: euro.org
Organizational Unit Name (eg, section) []: HQ
Common Name (e.g. server FQDN or YOUR name) []: hqfw.euro.org
Email Address []: ca@euro.org
```

## 2) WEB serveris – atslēgas un sertifikāta parakstīšanas pieprasījuma ģenerēšana
Tālākās darbības notiks web servera

###### Izveidosim nepieciešamās datnes

```
cd /
mkdir ssl
cd ssl
mkdir key cert csr
```

###### Uzģenerējam rsa atslēgu ar openssl

```
cd key
openssl genrsa -out intra.euro.org 4096
```

###### Uzģenerējam jaunu sertifikāta pieprasījumu

```
cd ..
cd csr
openssl req -new -key /ssl/key/intra.euro.org –out ../csr/intra.euro.org.csr
```

###### Aizpildam ievades laukus

```
Country Name (2 letter code) [AU]: HU
State or Province Name (Full name) [Some-State]: HU
Locality Name (eg, city) []: Budapest
Organization Name (eg, company) [Internet Widgits Pty Ltd]: euro.org
Organizational Unit Name (eg, section) []: HQ
Common Name (e.g. server FQDN or YOUR name) []: intra.euro.org
Email Address []: admin@euro.org
A challenge password []: Passw0rd
An optional company name []: euro.org
```

## 3) Sertifikāta parakstīšana
Šajā sadaļā mēs atgriežamies pie sākotnējā servera.

###### Parakstīsim sertifikātu

```
openssl ca –config /ca/openssl.cnf –extfile /ca/openssl.cnf –extensions v3_req –infiles /ssl/csr/intra.euro.org.csr
```

`ca` : komanda, kas paredzēta sertifikātu parakstīšanai, piešķirto sertifikātu un to statusu datubāzes uzturēšanai

`-extfile <filename>` : nolasa alternatīvo konfigurāciju

`-extensions <extension>` : norāda sertifikāta paplašinājumu

`-infiles <filename>` : norāda sertifikāta pieprasījumu sarakstu

###### Pārsauksim ģenerēto sertifikātu lasāmākā un saprotamākā vārdā

```
mv /ca/newcerts/xx.pem intra.euro.org.pem
```

`mv <filename> <new_filename>` : komanda, kas paredzēta, lai pārvietotu, vai pārdefinētu faila nosaukumu

###### Kopēt sertifikātu uz tīmekļa serverim paredzēto datni

```
cp /ca/newcerts/intra.euro.org.pem /ssl/cert/
```

Ja sekojāt līdzi bez kļūdām, tagad jums ir pašparakstīts ssl sertifikāts!
