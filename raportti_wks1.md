## wks1

Virtuaalikoneen sijainti: V:\VMBox\wks1

Virtuaalilevyn sijainti: V:\VMBOX\wks1\wks1.vmdk

Kieli yms. asetukset asennuksessa samoin kuin servereiden asennuksessa. Verkkoasetuksiin ei tehdä muuta kuin Host name asetus: `wks1.projekti.local` ja kun verkko laitetaan päälle, kone saa automaattisesti ip-osoitteen DHCPllä. Koska Server01 on DHCP-palvelin, se on oltava käynnissä, jotta DHCP toimii.

**Software selection** osiosta valitaan asennettavaksi **Workstation** ja lisäksi asennetaan **Office Suite and Productivity**.

Luodaan käyttäjätunnus Jaska Johtajalle ja asetetaan root-käyttäjän salasana. 

Työaseman käynnistyessä pyydetään hyväksymään lisenssiehdot, kun ne on hyväksytty jatketaan eteenpäin painamalla *Finish configuration*.

Kone ehdottaa kirjautumista Jaska Johtajana, valitaan kuitenkin kirjautumiskentän alapuolella olevasta linkistä *Not listed?* ja kirjaudutaan sisään root-tunnuksella.

Käydään läpi Gnomen asetukset, estetään sijaintitietojen lähettäminen, eikä liitetä tunnusta mihinkään ehdotetuista verkkopalveluista. Kun asetukset on saatu tehtyä ja tervetulotoivotukset suljettua, voidaan avata terminaali-ikkuna. Painetaan ruudun ylälaidasta **Activities** ja avautuvasta valikosta valitaan terminaali (kuvassa toiseksi alin kuvake).

![Activities](images/wks_terminaali.png?raw=true)

### Käyttäjien ja ryhmien luonti

Luodaan ryhmät Hallinto ja Myynti käyttäen samoja GID kuin ServerO1 vastaavilla ryhmillä.

```
groupadd -g 1111 Hallinto
groupadd -g 2222 Myynti
```

Lisätään Jaska Johtaja ryhmään hallinto
```
usermod -aG Hallinto <ktunnus>
```

Luodaan käyttäjätunnus Mika Myyjälle:
```
useradd -mc "Mika Myyjä" -G Myynti <ktunnus>
```

Luodaan käyttäjille salasanat esim. `pwmake 16` (luo 16 merkkiä pitkän merkkijonon). Salasanan lisääminen käyttäjätunnukselle:

```
passwd <käyttäjätunnus>
```

### Jaettu hakemisto

Asennetaan `nfs-utils`ja `nfs4-acl-tools`
```
dnf install nfs-utils nfs4-acl-tools -y
```

Serverin NFS jaot nähdään komennolla:
```
showmount -e Server01
```

Hakemistot mountataan hakemistoon `/shared`, johon kummallekin osastolle tehdään oma alihakemisto, eli luodaan hakemistot `/shared/Hallinto` ja `/shared/Myynti`. 
```
mkdir -p /shared/Hallinto
mkdir -p /shared/Myynti
```

Mounttaus voidaan tehdä komennolla:
```
mount -t nfs Server01:/shared/<osasto> /shared/<osasto>
```

Lisätään mounttaus `/etc/fstab` tiedostoon, jotta se tapahtuu automaattisesti koneen käynnistyksessä.

```
# mount shared directories
Server01:/shared/Hallinto /shared/Hallinto nfs defaults 0 0
Server01:/shared/Myynti /shared/Myynti nfs defaults 0 0
```

Lisätään käyttäjien kotihakemistoon linkki osaston hakemistoon:
```
sudo -u <käyttäjätunnus> ln -s /shared/<osasto> /home/<käyttäjätunnus>/<osasto>
```

Linkkien toimivuus voidaan testata kirjautumalla sisään Jaskan tai Mikan käyttäjätunnuksilla. 

Jaska Johtajan kotihakemistossa näkyy kansio, jonka nimi on Hallinto ja joka on linkki jaettuun kansioon.

![Kotihakemisto](images/hallinto_kansio.png?raw=True)

Jaskan näkymässä `/shared`-hakemistoon, huomataan että `Myynti`-kansion päällä on ruksi ja kun kansiota yrittää avata, se pyytää autentikointitietoja.

![Jaskan shared](images/jaska_shared.png?raw=True)

### SSH-avainten luonti ja kopiointi palvelimelle

Käyttäjät voivat luoda ssh-avaimia itse komennolla `ssh-keygen` ja avain kopioidaan kohteeseen komennolla `ssh-copy-id <käyttäjätunnus>@<kohdekone>`. Luodaan kuitenkin Jaskalle ja Mikalle avaimet käyttövalmiiksi.
```
sudo -u <käyttäjätunnus> ssh-keygen
<vastataan kysymyksiin enterin painamisella>
```
Koska avaimen siirrossa pyydetään käyttäjän salasanaa, on parempi, että käyttäjä tekee sen itse ja testaa sen jälkeen avaimen toimivuuden.
