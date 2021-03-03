# Työsuunnitelma

Asennetaan VMWare Workstation 16 Prolla kaksi CentOS 8 serveriä (Server01, Server02) ja yksi CentOS 8 työasema (wks1), käyttäen verkosta ladattua asennuspakettia.

## Server01
Asennus:
* asennuksessa asetetaan kiinteä ip 172.19.20.10

Muut toimenpiteet:
* asennetaan ja konfiguroidaan dnsmasq
* lisätään jaettuja hakemistoja varten oma levy, partitioidaan ja mountataan levy 
* luodaan käyttäjät (2 kpl) ja osastojen käyttäjäryhmät
* luodaan osastokohtaiset hakemistot ja määritellään niiden käyttöoikeudet
* asennetaan backup-ohjelma ja konfiguroidaan päivittäiset varmistukset (jälkimmäinen tehdään kun Server02 on asennettu ja valmisteltu)
* ssh-konfigurointi käyttämään pelkkiä avaimia (wks1 asennuksen ja avainten generoinnin ja testaamisen jälkeen)
* varmistetaan tietojen palautus backupeista

## Server02
Asennus:
* asennuksessa asetetaan kiinteä ip: 172.19.20.11

Muut toimenpiteet:
* luodaan backup-hakemisto, johon Server01 varmuuskopiot tallennetaan
* muut tarvittavat konfiguraatiot varmistuksiin ja varmistusten palauttamiseen.

## wks1
Asennus:
* verkkoasetuksissa DHCP

Muut toimenpiteet:
* luodaan käyttäjät (voidaan tehdä myös asennusvaiheessa)
* luodaan osastojen käyttäjäryhmät ja liitetään käyttäjät oikeisiin ryhmiin
* generoidaan käyttäjille ssh-avaimet ja kopioidaan ne palvelimelle
* asennetaan sshfs
* luodaan mount pointit jaetuille kansioille
* konfiguroidaan jaettujen kansioiden mounttaus käynnistyksen yhteydessä
* luodaan käyttäjien kotihakemistoihin linkitys jaettuihin hakemistoihin


