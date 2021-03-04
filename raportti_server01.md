# Asennusraportti

## Server01

### Virtuaalikoneen luonti

![VM luonti aloitusruutu](images/vm_luonti1.png?raw=True)

Koska Centos haluaa tehdä ns. easy installin, valitaan käyttöjärjestelmän asennus myöhemmin.

![Asenna myöhemmin](images/vm_luonti2.png?raw=True)

Valitaan käyttöjärjestelmä ja oikea versio

![Käyttis ja distro](images/vm_luonti3.png?raw=True)

Koneelle annetaan nimeksi Server01. 

Virtuaalikoneen sijainti isäntäkonella: V:\\VMBox\Server01

![Nimi ja sijainti](images/vm_luonti4.png?raw=True)

VMWare ehdottaa levytilaksi 20GB, mikä on ihan hyvä. Valitaan levyn tallennus yhteen tiedostoon (oletus on levyn jakaminen eri tiedostoihin).

![Levyn koko ja tallennustapa](images/vm_luonti5.png?raw=True)

Virtuaalilevyn sijainti isäntäkoneella: V:\\VMBox\Server01\Server01.vmdk

Ennen kuin päätetään koneen luonti, valitaan vielä _Customize Hardware..._

![Customize Hardware](images/vm_luonti6.png?raw=True)

Valitaan vasemmalta puolelta _New CD/DVD_ ja oikealta _Use ISO image file:_ ja lisätään asennustiedosto ja lopuksi suljetaan ikkuna.

![Asennusmedia](images/vm_luonti7.png?raw=True)

Palataan takaisin ikkunaan, jonka alalaidasta voidaan nyt valita _Finish_.
 
Ennen kuin aloitetaan käyttöjärjestelmän asennus, lisätään vielä toinen kovalevy, levyn lisäämiseen pääsee painamalla VMWaren virtuaalikone näkymässä _Edit virtual machine settings_.

Valitaan alareunasta _Add..._

![Kovalevyn lisääminen](images/toinen_levy.png?raw=True)

Avautuvasta ikkunasta valitaan _Hard Disk_

![Kovalevy](images/toinen_levy2.png?raw=True)

Käytetään VMWaren ehdottamia arvoja seuraavissa valinnoissa (SCSI, uuden virtuaalilevyn luominen), toinenkin levy saa olla 20GB ja se tallennetaan yhteen tiedostoon (Server01-0.vmdk).

![Disk file](images/toinen_levy6.png?raw=True)

Painetaan _Finish_ ja nyt koneen asetuksissa näkyy kaksi 20G kovalevyä.

![Ennen asennusta](images/ennen_asennusta.png?raw=True)

Virtuaalikone voidaan käynnistää ja aloittaa käyttöjärjestelmän asentaminen. Valitaan ensimmäisestä näkymästä vaihtoehto _Install_, valikossa liikutaan nuolinäppäimillä ja valinta tehdään painamalla enteriä.
 
Valitaan asennuskieleksi englanti, näppäimistön asetteluksi Finnish, aikavyöhykkeeksi Helsinki.

Verkkoasetukset tehdään kuvan osoittamalla tavalla. Nimipalveluna käytetään aluksi Googlen nimipalvelimia (8.8.8.8, 8.8.4.4). Toimiva nimipalvelu on tarpeen, jotta pystytään asentamaan järjestelmään päivityksiä ja ohjelmia ennen kuin saadaan koneen oma nimipalvelu toimintaan. Tallennetaan asetukset. Laitetaan koneen nimeksi Server01.localdomain ja napsautetaan verkkokytkin asentoon **On**

![Verkkoasetukset](images/ipv4_settings.png?raw=True)

Seuraavaksi tehdään partitioidaan levyt. Valitaan järjestelmän molemmat levyt ja vaihdetaan _Custom_ osiointiin ja painetaan *Done*. 

Valitaan valikosta *Standard Partition* ja painetaan alareunasta `+`. Ensimmäisenä lisätään `/boot` -osio.

![Partitiointi aloitus](images/centos_asennus_partitiointi.png?raw=True)

![boot](images/centos_asennus_partitiointi2.png?raw=True)

Lisätään muutkin partitiot (swap, / ja /shared). `sda` on järjestelmälle ja `sdb` datalle.  

![Partitiot](images/centos_asennus_partitiointi3.png?raw=True) 

*Installation Source* saa olla kuten on.

*Software Selectionista* valitaan asennettavaksi Server, ei muita paketteja.

![Software Selection](images/centos_asennus_sws.png?raw=True)

Asetetaan root-käyttäjälle salasana.

Muita käyttäjiä ei luoda vielä tässä vaiheessa.

Kun kaikki tarvittavat valinnat on tehty, voidaan aloittaa asennus. Asennus kestää 10-15 minuuttia.

![Valinnat tehty](images/centos_asennus_lopuksi.png?raw=True)

### Serverin käyttöönotto

Kun asennus on valmis ja kone on bootannut, kirjaudutaan sisään.

Ensimmäiseksi päivitetään järjestelmä: `dnf upgrade`

Asennetaan dnsmasq: `dnf install dnsmasq`

Kopioidaan talteen dnsmasqin alkuperäinen konfiguraatiotiedosto: `cp /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`

Konfiguraatiotiedosto on varsin pitkä, joten käydään läpi vain ne kohdat, joihin tehdään muutoksia. Muutokset on esitetty siinä järjestyksessä kuin ne konfiguraatiotiedostossa esiintyvät, mukana on myös konfiguraatiotiedostosta löytyvä selitys kullekin konfiguraatiolle. Muut käytettävät nimipalvelut ovat `8.8.8.8`ja `8.8.4.4` eli Googlen nimipalvelimet, mutta voitaisiin käyttää myös muita vastaavia palveluita (esim. OpenDNS). `dnsmasq` toimii `projekti.local` domainin nimipalveluna ja verkon ulkopuolelle menevä liikenne välitetään eteenpäin muille palvelimille.  

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

# The DHCP server needs somewhere on disk to keep its lease database.
# This defaults to a sane location, but if you want to change it, use
# the line below.
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases

# Set the DHCP server to authoritative mode. In this mode it will barge in
# and take over the lease for any client which broadcasts on the network,
# whether it has a record of the lease or not. This avoids long timeouts
# when a machine wakes up on a new network. DO NOT enable this if there's
# the slightest chance that you might end up accidentally configuring a DHCP
# server for your campus/company accidentally. The ISC server uses
# the same option, and this URL provides more information:
# http://www.isc.org/files/auth.html
dhcp-authoritative
```

Konfiguraation syntaksin voi tarkastaa komennolla: `dnsmasq --test`

Lisätään tiedoston `/etc/hosts` loppuun rivit:
```
172.19.20.2     router
172.19.20.10    Server01
172.19.20.11    Server02
```

Editoidaan tiedostosta /etc/sysconfig/network-scripts/ifcfg-ens33 pois DNS1 ja DNS2 asetukset ja käynnistetään interface uudelleen:
```
ip link set ens33 down
ip link set ens33 up
```

Tiedoston `/etc/resolv.conf` sisältö pitäisi näyttää suunnilleen tältä, jos tiedostossa on muita `nameserver` asetuksia, ne voi poistaa. `dnsmasq` ei käytä tätä tiedostoa, mutta muut ohjelmat hakevat tiedon käytössä olevasta nimipalvelusta tästä tiedostosta.
```
search localdomain
nameserver 127.0.0.1
```

Käynnistetään `dnsmasq` ja määritellään sen käynnistys koneen käynnistyksen yhteydessä: `systemctl enable --now  dnsmasq`

Lisätään dns ja dhcp palomuurin sallittuihin palveluihin`:
```
firewall-cmd --add-service=dns
firewall-cmd --add-service=dhcp
firewall-cmd --runtime-to-permanent
```

### Käyttäjäryhmien ja käyttäjien luominen

Luodaan osastoille käyttäjäryhmät, ryhmälle annetaan tietty GID, jota käytetään myös työaseman vastaaville käyttäjäryhmille.
```
groupadd -g 1111 Hallinto
groupadd -g 2222 Myynti
```

Ryhmien pitäisi nyt näkyä `/etc/group` tiedoston viimeisillä riveillä.

Jotta hakemistojen jakaminen toimisi ongelmitta, luodaan serverille samat käyttäjät kuin työasemalla. Käyttäjillä täytyy olla sama käyttäjätunnus ja `uid` sekä työasemalla että serverillä. Luodaan ensin Jaskan ja sitten Mikan käyttäjätunnus ja noudatetaan samaa järjestystä luotaessa työaseman käyttäjätunnuksia.
```
useradd -c "<käyttäjän nimi>" -G <osasto> <käyttäjätunnus>
```
Jos käyttäjien ei ole välttämätöntä päästä kirjautumaan palvelimelle, voidaan käyttää optiota `-s /sbin/nologin`. Käyttäjätunnukset ovat tarpeellisia vain jaettujen kansioiden käytöoikeuksien hallinnoinnin helpottamiseksi, mutta muuten muille kuin ylläpidosta vastaaville ei pitäisi olla tarvetta päästä kirjautumaan palvelimelle.

Käyttäjälle voidaan luoda salasana vaikkapa komennolla `pwmake <numero>`, joka luo `<numero>` merkkiä pitkän merkkijonon, joka voidaan sitten kopioida käyttäjän salasanaksi. 
```
passwd <käyttäjätunnus>
<salasana>
<salasana uudelleen>
```
Salasanan voisi lisätä myös käyttäjän luomisen yhteydessä, mutta sitä ei voi lisästä selkokielisenä vaan se on salattava, joten on helpompaa lisätä salasana tällä tavalla. Käyttäjätunnukset ja salasanat toimitetaan käyttäjille henkilökohtaisesti.

### Jaetut kansiot

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

Asennetaan `nfs`
```
dnf install nfs-utils -y
```

Käynnistetään `nfs`
```
systemctl enable --now nfs-server
```
Jotta muutokset tulevat voimaan, käynnistetään `nfs-utils`
```
systemctl start nfs-utils
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

Varmuuskopiointiin käytetään `rsyncìä, joka on valmiiksi asennettu serveriin.

Varmuuskopioinnissa käyttäjän tunnistus tapahtuu ssh-avaimella. Luodaan root-käyttäjälle ssh-avain ja kopioidaan se Server02:lle.
```
ssh-keygen <painetaan Enter kaikkiin kysymyksiin>
ssh-copy-id root@Server02 <vastataan: yes, syötetään Server02 rootin salasana>
```

Testataan avaimen toimivuus: `ssh root@Server02`

Varmuuskopiointi tapahtuu komennolla:
```
rysnc -a /shared Server02:/backups
```

Ajastetaan varmuuskopiointi tehtäväksi päivittäin klo 23.

Tehdään hakemisto ajastettaville skripteille:`mkdir scripts`. Lisätään sinne skripti `backup_shared`
```
#!/bin/bash

# Backup files from directory /shared to Server02 directory /backups
rsync -a /shared Server02:/backups
```

Lisätään tiedostolle suoritusoikeudet: `chmod u+x backup_shared`.

Ajastetaan varmistus: `crontab -e`
```
00 23 * * * /root/scripts/backup_shared
```

`rsync` luo ns. lisäysvarmistuksen eli jokaisella varmistuksella lisätään muutokset vanhojen tiedostojen päälle. Varmuuskopioista ei ole mahdollista palauttaa tiettyä versiota tiedostosta vain viimeisimmän version palauttaminen on mahdollista. Palautus voidaan tehdä kääntämällä `rsync`in argumentit toisin päin esim.
```
rsync -a Server02:/backups/shared/Hallinto /shared/Hallinto
```

Parempi on kuitenkin tehdä palautusta varten väliaikainen tiedosto ja kopioida halutut tiedostot siihen kansioon, johon ne halutaan palauttaa esimerkiksi
```
rsync -a Server02:/backups/shared/Myynti /tmp/restore_myynti
cp /tmp/restore_myynti/tiedosto /shared/Myynti/tiedosto
```

`rsync` ei automaattisesti poista varmuuskopiosta poistettuja tiedostoja, mikä pikkuhiljaa kasvattaa `/backups` -hakemiston kokoa. Tiedostoja voi siivota varmistuksista `--delete` optiolla. Koska käyttäjät ovat laiskoja siivoamaan turhia tiedostoja pois, tämä tuskin auttaa pidemmän päälle ja onkin syytä harkita jonkinlaista kierrätystä varmistuksille.

### Estetään salasanan käyttö ssh-yhteyksissä

Tiedostossa `/etc/ssh/sshd_config` asetus
```
PasswordAuthentication yes
```
vaihdetaan muotoon
```
PasswordAuthentication no
```
Tämän jälkeen Server01 ei salli kirjautumista salasanalla. Sama asetus voidaan tehdä myös Server02:ssa. 
