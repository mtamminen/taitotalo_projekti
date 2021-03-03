# Asennusraportti

## Server01

### VMWaren verkkoasetukset.

Avataan VMWare Virtual Network Editor:

Painetaan Change Settings, jotta päästään editoimaan VMnet8 asetuksia.
Otetaan pois päältä DHCP.
Alareunan tekstikenttiin laitetaan Subnet IP: 172.19.20.0
Subnet mask: 255.255.255.0

![VMWare Virtual Network Editor](images/virtual_network_editor.png?raw=True)

NAT Settings: 
Gateway IP: 172.19.20.2

![NAT Settins](images/nat_settings.png?raw=True)

### Serverin asennus

Aloitetaan uuden virtuaalikoneen asennus:

Koneen nimeksi asetetaan Server01.

Virtuaalikoneen sijainti: V:\\VMBox\Server01

Virtuaalilevyn sijainti: V:\\VMBox\Server01\Server01.vmdk

Valitaan käyttöjärjestelmän kieleksi englanti, näppäimistön asetteluksi Finnish, aikavyöhykkeeksi Helsinki.

Verkkoasetukset tehdään seuraavalla tavalla. Nimipalveluna käytetään aluksi Googlen nimipalvelimia (8.8.8.8, 8.8.4.4). Toimiva nimipalvelu on tarpeen, jotta pystytään asentamaan järjestelmään päivityksiä ja ohjelmia ennen kuin saadaan koneen oma nimipalvelu toimintaan.

![Verkkoasetukset](images/ipv4_settings.png?raw=True)

Levyn partitiointi tehdään automaattisen ehdotuksen mukaan.

Software Selectionista valitaan asennettavaksi Server, ilman lisäohjelmia.

Asetetaan root-käyttäjälle salasana. (Salasanoja ei liitetä julkiseen dokumentaatioon.)

Muita käyttäjiä ei luoda vielä tässä vaiheessa.

### Serverin käyttöönotto

Kun asennus on valmis, bootataan kone ja kirjaudutaan sisään.

Testataan verkon toiminta: `ping google.com` vastaa, joten verkko toimii.

Päivitetään järjestelmä: `dnf upgrade`

Asennetaan dnsmasq: `dnf install dnsmasq`

Kopioidaan talteen dnsmasqin alkuperäinen konfiguraatiotiedosto: `cp /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`

Konfiguraatiotiedosto on varsin pitkä, joten käydään läpi vain ne kohdat, joihin tehdään muutoksia.

```
# Never forward plain names (without a dot or domain part)
domain-needed

# Never forward addresses in the non-routed address spaces.
bogus-priv

# If you don't want dnsmasq to read /etc/resolv.conf or any other
# file, getting its servers from this file instead (see below), then
# uncomment this.
no-resolv

# Add other name servers here, with domain specs if they are for
# non-public domains.
server=8.8.8.8
server=8.8.4.4

# Add local-only domains here, queries in these domains are answered
# from /etc/hosts or DHCP only.
local=/projekti.local/

# Or which to listen on by address (remember to include 127.0.0.1 if
# you use this.)
listen-address=::1,127.0.0.1,172.19.20.10 

# Set this (and domain: see below) if you want to have a domain
# automatically added to simple names in a hosts-file.
expand-hosts

# Set the domain for dnsmasq. this is optional, but if it is set, it
# does the following things.
# 1) Allows DHCP hosts to have fully qualified domain names, as long
#     as the domain part matches this setting.
# 2) Sets the "domain" DHCP option thereby potentially setting the
#    domain of all systems configured by DHCP
# 3) Provides the domain part for "expand-hosts"
domain=projekti.local

# Uncomment this to enable the integrated DHCP server, you need
# to supply the range of addresses available for lease and optionally
# a lease time. If you have more than one network, you will need to
# repeat this for each network on which you want to supply DHCP
# service.
dhcp-range=172.19.20.12,172.19.20.100,12h

# Do the same thing, but using the option name
dhcp-option=option:router,172.19.20.2

# Set the DHCP server to authoritative mode. In this mode it will barge in
# and take over the lease for any client which broadcasts on the network,
# whether it has a record of the lease or not. This avoids long timeouts
# when a machine wakes up on a new network. DO NOT enable this if there's
# the slightest chance that you might end up accidentally configuring a DHCP
# server for your campus/company accidentally. The ISC server uses
# the same option, and this URL provides more information:
# http://www.isc.org/files/auth.html
dhcp-authoritative

# The DHCP server needs somewhere on disk to keep its lease database.
# This defaults to a sane location, but if you want to change it, use
# the line below.
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
```

Konfiguraation syntaksin voi tarkastaa komennolla: `dnsmasq --test`

Lisätään tiedoston `/etc/hosts` loppuun rivit:
```
172.19.20.2     router
172.19.20.10    Server01
172.19.20.11    Server02
```

Editoidaan tiedostosta `/etc/sysconfig/network-scripts/ifcfg-ens33` pois DNS1 ja DNS2 asetukset ja käynnistetään interface uudelleen:
```
ip link set ens33 down
ip link set ens33 up
```

Uudellenkäynnistetään dnsmasq: `systemctl restart dnsmasq`

Lisätään dns ja dhcp palomuurin sallittuihin palveluihin`:
```
firewall-cmd --add-service=dns
firewall-cmd --add-service=dhcp
firewall-cmd --runtime-to-permanent
```

### Jaetut kansiot

Jaettuja kansioita varten lisätään Server01:n toinen levy. Valitaan VMWaressa Edit Virtual Machine Settings. Valitaan Hard Disk ja Add. Seurataan VMWaren ehdotuksia levyn ominaisuuksista, paitsi että tallennetaan levy yhteen tiedostoon.

Virtuaalilevyn sijainti: V:\\VMBox\Server01\Server01\Server01-0.vmdk

Tarkastetaan, että lisätty levy näkyy järjestelmässä: `parted -l` löytää järjestelmästä kaksi levyä `/dev/sda` ja `/dev/sdb`, joista jälkimmäistä ei ole partitioitu. Partitioidaan levy: `parted /dev/sdb`. Varataan koko levy jaetuille kansioille ja tehdään vain yksi partitio. 

Vaihdetaan käytettäväksi yksiköksi GB: `unit GB`.

Luodaan partitiotaulu: `mktable gpt`

Luodaan partitio: `mkpart`, vastaillaan esitettyihin kysymyksiin, partition nimi voidaan ohittaa painamalla enteriä, tiedostojärjestelmän tyypiksi voidaan valita xfs, partition alkukohta 0 ja loppu 20. `print` tulostaa partition tiedot, ohjelmasta poistuminen `quit`. Poistuessa parted muistuttaa, että voi olla tarpeen päivittää `/etc/fstab`.

Luodaan partitiolla xfs-tiedostojärjestelmä: `mkfs.xfs /dev/sdb1`.

Luodaan mount point: `mkdir /shared`

Mountataan partitio: `mount /dev/sdb1`

Koska mounttaus olisi hyvä tehdä, kun järjestelmä käynnistetään, täytyy editoida tiedostoa `/etc/fstab`

Tarkastetaan ensin partition UUID: `blkid /dev/sdb1`

Lisätään tiedostoon `/etc/fstab` rivi:
```
UUID=a8bba54d-1383-48e1-be29-c01c6607a48d /shared    xfs     defaults        0 0
```   

### Käyttäjien ja käyttäjäryhmien luonti

Luodaan osastoille käyttäjäryhmät, ryhmälle annetaan tietty GID, jota käytetään myös työaseman vastaaville käyttäjäryhmille.
```
groupadd -g 1111 Hallinto
groupadd -g 2222 Myynti
```

Ryhmien pitäisi nyt näkyä `/etc/group` tiedoston viimesillä riveillä.

Jotta hakemistojen jakaminen toimisi ongelmitta, luodaan serverille samat käyttäjät kuin työasemalla. Käyttäjillä täytyy olla sama käyttäjätunnus ja `uid` sekä työasemalla että serverillä.
```
useradd -c "<käyttäjän nimi>" -G <osasto> -s /sbin/nologin <käyttäjätunnus>
```

### Kansioiden jakaminen `nfs`llä

Asennetaan `nfs`
```
dnf install nfs-utils -y
```

Käynnistetään `nfs`
```
systemctl enable --now nfs-server
```

Luodaan osastojen jaetut hakemistot:
```
mkdir /shared/Hallinto
mkdir /shared/Myynti
```

Vaihdetaan jaettujen kansioiden omistajaksi nobody ja vastaava osasto
```
chown -R nobody:Hallinto /shared/Hallinto/
chown -R nobody:Myynti /shared/Myynti
```

Muutetaan hakemistojen suojausta. Omistajalla ja ryhmällä on kaikki oikeudet, alihakemistoille tulee sama omistaja kuin hakemistolla.
```
chmod -R 2770 /shared/*
```

Jotta muutokset tulevat voimaan, käynnistetään uudelleen `nfs-utils`
```
systemctl restart nfs-utils
```

Lisätään tiedostoon `/etc/exports` jaettujen kansioiden tiedot, jotta ne näkyvät myös työasemalle.
```
/shared/Hallinto	wks1.projekti.local(rw,sync,no_all_squash,root_squash)
/shared/Myynti  	wks1.projekti.local(rw,sync,no_all_squash,root_squash)
```
Ensimmäisenä on polku jaettavaan kansioon, seuraavana asiakaskoneen nimi ja sen jälkeen optiot jaolle.

* `rw`: asiakkaalla luku- ja kirjoitusoikeudet jaettuihin kansioihin
*  `sync`: muutokset kirjoitetaan levylle ennen kuin voidaan suorittaa muita operaatioita
* `no_all_squash`: mäppää NFS serverin `uid` ja `gid` tiedot työaseman vastaaviin
* `root_squash`: työaseman `root`-käyttäjän `uid` ja `gid` mäpätään serverillä tuntemattomaksi
 
Hakemistot eksportoidaan komennolla:
```
exportfs -arv
```
optio `-a` eksportoi kaikki kansiot, `-r` tekee kaikkien kansioiden uudelleneksportoinnin ja `-v` näyttää mitä tehdään.

Lopuksi lisätään palomuurin sallittuihin palveluihin `nfs`, `rpc-bind`ja `mountd`:
```
firewall-cmd --add-service={nfs,rpc-bind,mountd}
firewall-cmd --runtime-to-permanent
```

Tämän jälkeen on vielä valmisteltava jako työasemalla.
  
### Jaettujen kansioiden varmuuskopiointi

Asennetaan palvelimelle `rdiff-backup`. Sitä varten tarvitaan `epel-release`, joka on asennettava ensin.
```
dnf install -y epel-release
dnf install -y rdiff-backup
```

Varmuuskopioinnissa käyttäjän tunnistus tapahtuu ssh-avaimella. Luodaan root-käyttäjälle ssh-avain ja kopioidaan se Server02:lle.
```
ssh-keygen <painetaan Enter kaikkiin kysymyksiin>
ssh-copy-id root@Server02 <vastataan: yes, syötetään Server02 rootin salasana>
```

Testataan avaimen toimivuus: `ssh root@Server02`

Varmuuskopiointi tapahtuu komennolla:
```
rdiff /shared Server02::/backups
```

Ajastetaan varmuuskopiointi tehtäväksi päivittäin klo 23.

Tehdään hakemisto ajastettaville skripteille:`mkdir scripts`. Lisätään sinne skripti `backup_shared`
```
#!/bin/bash

# Backup files from directory /shared to Server02 directory /backups
rdiff /shared Server02::/backups
```

Lisätään tiedostolle suoritusoikeudet: `chmod u+x backup_shared`.

Ajastetaan varmistus: `crontab -e`
```
00 23 * * * /root/scripts/backup_shared
```


