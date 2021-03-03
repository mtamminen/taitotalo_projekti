## Server02

### Serverin asennus

Virtuaalikoneen sijainti: V:\\VMBox\Server02

Virtuaalilevyn sijainti: V:\\VMBox\Server02\Server02.vmdk

Server02 konfigurointi tehdään muuten samoin kuin Server01, verkkokonfiguraatio on seuraavanlainen:

![Server02_verkko](images/ipv4_server02.png?raw=True)

Koneen nimeksi ja domainiksi asetetaan: Server02.projekti.local

Root-käyttäjälle asetetaan salasana.


Ennen koneen käynnistämistä lisätään varmuuskopiointia varten toinen 25GB levy.

Levyn sijainti koneella: V:\\VMBOX\Server02\Server02-0.vmdk

Levylle luodaan koko levyn kokoinen partitio:
```
parted /dev/sdb
unit GB
mklabel gpt
mkpart / <Enter> / <Enter> / 0 / 25 
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

## Varmuuskopiointi

Asennetaan koneelle `rdiff-backup`. Kuten työasemalla, täytyy ensin asentaa `epel-release`, minkä jälkeen voidaan asentaa haluttu ohjelma.
```
dnf install -y epel-release
dnf install -y rdiff-backup
```

Tämän jälkeen valmistellan palvelin Server01, josta varmistukset tehdään.
