# Webserver installieren und konfigurieren

An sich würde es reichen, wenn man einen Webserver aufsetzt, indem man einfach einen solchen installiert. Aber das ist weder sicher genug noch hat man dann nichts für die Leistung des Systems getan. 

Einige der folgenden Schritte sind optional und für den Betrieb eines Webservers nicht notwendig. Aber leider ist die kostenlose Variante des Servers auf der Google Cloud nicht unbedingt der schnellste und leistungsfähigste, daher schlage ich vor, dass wir zunächst eine Swap-Partition erstellen und in Betrieb nehmen, damit bei Engpässen beim RAM auf diese Partition ausgelagert werden kann.

## Optional: Swap-Partition erstellen

Von haus aus ist ein Server in der Google Cloud nicht mit einer Swap-Partition ausgestattet. Sicherheitshalber kontrollieren wir, ob nicht doch schon eine besteht:

```
sudo swapon --show
```

Kommt keine Ausgabe des Systems auf diesen Befehl, existiert keine Swap-Partition. Also müssen wir eine erstellen. Dies machen wir mit dem folgenden Befehl:

```
sudo fallocate -l 1G /swapfile
```

Das "1G" bedeutet, dass hier 1 GB der Festplatte für eine Swap-Partition genutzt und reserviert werden. Wem das nicht reicht, kann aus "1GB" auch "4GB" machen. Wie dem auch sei, danach muss man die Rechte an dieser Datei, die durch den vorhergehenden Befehl erstellt wurde so einschränken, dass kein "normaler" Benutzer da reinschauen kann:

```
sudo chmod 600 /swapfile
```

Mit dem folgenden Befehl teilt man dem System mit, dass er die erstellte Datei für die Nutzung einer Swap-Partition nutzen soll:

```
sudo mkswap /swapfile
```

Wenn es funktioniert hat, kommt eine Ausgabe, ähnlich wie der hier:

```
Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
no label, UUID=746df872-36b2-4606-a29f-e627e707cbf6
```

So kann man sehen, dass die Größe der Swap-Partition auf 4 GB festgelegt wurde. Jetzt fehlt nur noch der Befehl, der die Swap-Partition einschaltet.

```
sudo swapon /swapfile
```

Jetzt erfolgt nochmals die Kontrolle, ob das Erstellen und in Betrieb nehmen funktioniert hat:

```
sudo swapon --show
```

In unserem Beispiel lautet dann die Ausgabe des Systems:

```
NAME      TYPE SIZE USED PRIO
/swapfile file   4G   0B   -1

```

Jetzt läuft also die Swap-Partition. Wir müssen jetzt nur noch dafür sorgen, dass sie bei jedem Start des Systems bereitgestellt wird. Ubuntu hat eine Datei, in der die Struktur der Partitionen für das System beschrieben sind. Das ist die fstab-Datei. Mit den folgenden beiden Befehlen werden wir von der bestehenden Datei ein Backup erstellen und dann die Swap-Partition eintragen:

```
sudo cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Jetzt ist dafür gesorgt, dass auch nach einem Reboot die Swap-Partition zur Verfügung steht. Was man jetzt noch wissen muss: Man kann dem Server beibringen, ab welcher Auslastung des Arbeitsspeichers er auf die freie Swap-Partition zurückgreifen kann. Wir können mit den beiden folgenden Befehlen dem System jetzt sagen, dass es die Swappartition früher (also schon bereits ab 10 % der Speicherauslastung) und intensiver nutzen soll.

```
sudo sysctl vm.swappiness=10
sudo sysctl vm.vfs_cache_pressure=50
```

Natürlich wäre es schön, wenn das System das bereits weiß, wenn es hochfährt. Dazu muss man die Voreinstellungen überschreiben, die nicht unbedingt auf eine Swap-Nutzung hinweisen oder gar nicht vorhanden sind. Ähnlich wie im Schritt zuvor erstellen wir ein Backup der betreffenden Einstellungsdatei und tragen dann die von uns gewünschten Einstellungen ein:

```
sudo cp /etc/sysctl.conf /etc/sysctl.conf.bak
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
```

Wenn man möchte, kann man jetzt einmal mit dem folgenden Befehl einen Neustart des Servers initiieren:

```
sudo reboot
```

Keine Sorge. Es kommt von PuTTY dann eine Fehlermeldung, die sagt, dass es die Verbindung zum Server verloren hat. Nach ein paar Minuten Wartezeit werden wir uns wie gehabt wieder mit dem Server verbinden und mit dem Befehl von eben die Existenz der Swap-Partition nachweisen:

```
sudo swapon --show
```

Mit den folgenden Befehlen kann man sich weitere Details über die Nutzung des Speichers bzw. der Festplatten ansehen:

```
free -h
df -h
```

Damit wäre die Vorbereitung bezüglich der Swap-Partition abgeschlossen.

## Synchronisation mit Zeitserver
sudo apt-get update
sudo apt-get dist-upgrade -y
sudo apt-get autoremove --purge -y
sudo apt-get autoclean -y


## Installation von nginx
sudo apt-get install nginx-full -y

## Firewall einstellen
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
sudo ufw allow "nginx Full"
sudo ufw reload
sudo ufw status verbose

## Einstellungen in nginx für SSL-Betrieb
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot

## Testen der SSL-Qualität

SSLlabs (Qualys):

Hierzu mal checken:
- https://github.com/TrullJ/ssllabs
- http://www.h4hacks.com/2015/06/qualys-ssl-labs-api-multithreaded.html
