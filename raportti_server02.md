## Server02

### Serverin asennus

Virtuaalikoneen sijainti isäntäkoneella: V:\\VMBox\Server02

Virtuaalilevyn sijainti isäntäkoneella: V:\\VMBox\Server02\Server02.vmdk

Server02 konfigurointi tehdään muuten samoin kuin Server01, verkkokonfiguraatio on seuraavanlainen:

![Server02_verkko](images/ipv4_server02.png?raw=True)

Koneen nimeksi ja domainiksi asetetaan: Server02.projekti.local

Levyjä on aluksi vain yksi ja sen partitiointi voidaan tehdä asennusohjelman ehdottamalla tavalla.

Root-käyttäjälle asetetaan salasana ja aloitetaan asennus.

Päivitetään kone.

Ennen kuin tehdään mitään muuta, lisätään vielä toinen levy varmuuskopioita varten. Koska kopioitavan levyn koko on 20GB, varataan varmuuskopioille varmuuden vuoksi 30GB.

Avataan VMWaren _VM_-valikko ja valitaan sieltä _Settings_. Levyn lisääminen tapahtuu muuten samoin kuin Server01 kanssa tehtiin.

Levyn sijainti isäntäkoneella: V:\\VMBOX\Server02\Server02-0.vmdk

Käynnistetään kone uudelleen: `reboot`.

Listataan levyt ja partitiot komennolla: `parted -l`.

Uusi levy on `/dev/sdb`, joka on osioitava ennen kuin sen voi ottaa käyttöön.

Levylle luodaan koko levyn kokoinen partitio:
```
parted /dev/sdb
unit GB
mklabel gpt
mkpart primary 0 30
print
quit
```

Luodaan partitiolle tiedostojärjestelmä
```
mkfs.xfs /dev/sdb1
```

Luodaan hakemisto, johon partitio mountataan ja tehdään mountti:
```
mkdir backups
mount /dev/sdb1 backups
```

Lisätään vielä levyn mounttaus koneen käynnistyksessä tiedostoon `/etc/fstab`. Ensin täytyy selvittää partition `UUID`: `blkid /dev/sdb1`. Lisätään em. tiedostoon rivi:
```
UUID=4c23b535-a346-4db3-841e-c680ab2c1bdd /backups xfs defaults 0 0
```

### Varmuuskopiointi

Palvelimelle on jo valmiiksi asennettu `rsync` ja se riittää.


