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

