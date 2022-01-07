[![](https://img.shields.io/badge/Buy%20me%20-coffee!-orange.svg?logo=buy-me-a-coffee&color=795548)](https://buymeacoff.ee/PatrickHallek)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![PRs welcome!](https://img.shields.io/badge/contributions-welcome-green.svg?style=flat)](https://github.com/PatrickHallek/automated-irrigation-system/issues)

# Automatisches Bewässerungssystem von Patrick Hallek
## WIP 
### Es funktioniert alles - es werden noch Kleinigkeiten angepasst

Dieses Projekt ist eine Kopie von [Patrick Hallek Projekt](https://github.com/PatrickHallek/automated-irrigation-system/). 
Ich möchte es für deutsche Benutzer anpassen und die Benutzererfahrung verbessern. 

Es handelt sich um eine Open-Source-Anwendung zur automatischen Bewässerung von Pflanzen. Bis jetzt gibt es fast keine kostenlose professionelle Software und Anleitungen, um eine DYI-Bewässerung zu bauen, die skalierbar, genau und vor allem langlebig ist. Die App ist auch nicht nur dazu da, um gut auszusehen und aus Liebe zu den Daten. Sie ist vor allem ein Werkzeug, um die Sensoren genau auf die Bedürfnisse der Pflanzen abzustimmen. Hier scheitern die meisten Bewässerungssysteme mit direkter Bodenfeuchtemessung, da jeder Boden und jede Pflanze anders ist und daher eine manuelle Kalibrierung und möglicherweise nach einiger Zeit auch eine Rekalibrierung erforderlich ist.

Die App enthält die folgenden Funktionen:
- Überwachung und Anzeige von Zeitraumdaten auf Minuten-, Stunden-, Tages-, Wochen- und Monatsebene
- Einstellung des Wasserstandes, ab dem die automatische Bewässerung ausgelöst werden soll.
- Einstellung, wie lange die Pumpe während einer Bewässerung arbeitet
- Manuelle Aktivierung der Bewässerung mit einer Taste
- Umschalten zwischen verschiedenen Sensorprofilen / Pflanzen
- Umschalten zwischen dunklem und hellem Design

| Dunkels Design  | Helles Design |
| ------------- | ------------- |
| ![App dark themed](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/docs/images/app-dark.png) | ![App light themed](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/docs/images/app-light.png)|

# Inhaltsverzeichnis 

1. [ Hardware Liste ](#part-list)
2. [ Hardware Architecture ](#hardware-architecture)
3. [ Software Architecture ](#software-architecture)
4. [ Setup NodeMCU ESP8266 ](#nodemcu)
5. [ Raspberry Pi mit Docker konfigurieren (empfohlen) ](#raspi-docker)
6. [ Raspberry Pi manuell konfigurieren ](#raspi-manually)
7. [ Bedienung ](#usage)
8. [ Beiträge ](#contributing)
9. [ Lizenz ](#license)

<a name="part-list"></a>
## Part list

| Name  | Anzahl | Beschreibung |
| ------------- | ------------- | ------------- |
| [CHEAP ALL IN ONE OFFER](https://amzn.to/39jLexe)  | 1 - n  | Pumpe, Schlauch, Leistungssensor und Relais |
| [NodeMCU ESP8266](https://amzn.to/3dh7IPR)  | 1 - n  | Wifi-Modul zum Auslesen der Kapazitäten und Senden an das Backend (Raspi)|
| [Raspberry Pi Zero](https://amzn.to/2NeY05X)  | 1  | Ausführen der gesamten Software und Auslösen der Pumpe(n) |
| [Raspberry Pi SD Card](https://amzn.to/3hLghpt)  | 1  | Dies ist der Datenspeicher für den Raspberry Pi  |
| [Relay](https://amzn.to/2YgcvNt)  | 1 - n  | Zum Schließen oder Öffnen des Pumpenkreislaufs auf Signal des Raspi |
| [Capacitive Soil Moisture Sensors](https://amzn.to/3dh9PTU)  | 1-n  | Zur Messung der Bodenfeuchtigkeit. Die kapazitiven Sensoren lösen sich nicht auf. Verwenden Sie niemals elektrodynamische Feuchtesensoren, da diese sehr schnell verschleißen. |
| [Pump](https://amzn.to/2WO8Zsf)  | 1 - n  | Theoretisch kann jede Pumpe verwendet werden, da sie über eine separate Stromversorgung und das Relais gesteuert wird. |
| [Aquarium tube and irrigation nozzles](https://amzn.to/3153W9I)  | -  | Wasserübertragung zu den Pflanzen und Verteilung des Wassers auf der Erde |

*Alle Produktlinks sind auch Partnervorschläge für genau die Produkte, die ich für das System gekauft habe. Wenn Sie über den Link bestellen, erhalte ich <b>Patrick Hallek</b> eine kleine Provision.

Das "n" in der Anzahl ergibt sich aus der Anzahl der Pumpen oder der verschiedenen Pflanzen. In einem Hochbeet zum Beispiel reicht normalerweise eine Pumpe und ein Sensor aus. Wenn Sie jedoch verschiedene Topfpflanzen haben, müssen sie alle separat bewässert werden, und deshalb müssen Sie für jede Topfpflanze eine Pumpe und einen Sensor kaufen.

<a name="hardware-architecture"></a>
## Hardware Architektur
![Hardware Architecture](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/docs/images/hardware-architecture.png)

Die Architektur wurde so gewählt, dass die Pumpenlogik und die Aufzeichnung der Messdaten getrennt sind. Dadurch ist es möglich, bis zu 26 Pumpen mit dem Raspberry Pi zu steuern (Anzahl der standardmäßig verfügbaren GPIO-Pins). Es ist auch nicht möglich, die analogen Signale des kapazitiven Sensors mit dem Raspberry selbst auszulesen, da der Raspberry nur digitale Signale verarbeiten kann. Sicherlich ist es möglich, die Sensoren mit einem MCP3008 und der seriellen Schnittstelle auszulesen, aber das erfordert mehr Pins und der Aufbau ist nicht mehr so sauber wie früher. Die Pumpen sind auch separat an eine Stromversorgung angeschlossen, deren Schaltung durch das Relais gesteuert wird. So ist es auch möglich, Pumpen mit 12V oder mehr zu verwenden.

<a name="software-architecture"></a>
## Software Architektur
![Software Architecture](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/docs/images/software-architecture.png)

Für die Software-Architektur ist die [MERN Stack](https://www.educative.io/edpresso/what-is-mern-stack) verwendet wurden. Die Software besteht aus [Node.js](https://nodejs.org/de/about/) backend und [Express.js](https://expressjs.com/de/), eine [Mongo Datenbank](https://www.mongodb.com/) und einem [React](https://reactjs.org/) frontend. Das C++ Script steuert das NodeMCU ESP8266, welches die Daten über [REST-Schnittstelle](https://en.wikipedia.org/wiki/Representational_state_transfer) des Backends sendet. Die Daten werden im Backend verarbeitet, wo entschieden wird, ob bewässert werden soll oder nicht. Außerdem werden die Daten dann in der MongoDB gespeichert. Mit dem Frontend können diese Daten auch über REST vom Backend abgefragt werden.

<a name="nodemcu"></a>
## NodeMCU ESP8266 einrichten

Um den MCU Controller einzurichten, schau dir am besten [das Video](https://www.youtube.com/watch?v=flLMMHCNkQE) an.

Bevor du das Programm hochlädst, musst du dein WLAN-Passwort, den WLAN-Namen (ssid), die IP des Raspberry Pi (Host) und den Sensornamen festlegen. Der Sensorname wird der Name sein, der in der App angezeigt wird. Es ist also am besten, den Namen der Pflanze zu wählen, mit der der Sensor verbunden werden soll.

Wenn die Arduino-IDE erfolgreich für den NodeMCU konfiguriert ist, können Sie das Programm, das Sie in diesem Repository finden, hochladen von [arduino-code/ESP8266_moisture/ESP8266_moisture.ino](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/arduino-code/ESP8622_moisture.ino/ESP8622_moisture.ino.ino) zum ESP8266 Controller.


<a name="raspi-docker"></a>
## Raspberry Pi mit [Docker](https://www.docker.com/) konfigurieren (empfohlen)

Um die erforderlichen Programme nicht manuell installieren zu müssen, kannst du die Anwendung auch mit Docker in Containern ausführen. Führe dazu die folgenden Schritte aus:

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
sudo apt-get install -y libffi-dev libssl-dev
sudo apt-get install -y python3 python3-pip
sudo apt-get remove python-configparser
sudo pip3 install docker-compose
```

Nun musst du die IP-Adresse deines Pi´s in die Umgebungsvariable `REACT_APP_BACKEND_URL=http://<DEINE-RaspberryPi-IP>:3000` in der docker-compose-Datei eintragen:

```bash
sudo nano docker-compose.yml
```

Du kannst die IP mit dem Befehl `ifconfig` herausfinden. Sie sollte so etwas aussehen *192.168.178.44*. Du kannst deine  Eingabe im Nano-Editor mit `ctr + x` speichern, dann `yes` eintippen und schließlich mit `enter` beenden.

Jetzt sollte alles fertig sein und du kannst die Anwendung mit dem folgenden Befehl starten:

```bash
sudo docker-compose up
```

**Achtung**: Wenn du einen Raspberry Pi mit einem *ARMv7* Prozessor hast, musst du das image in der docker-compose Datei anpassen. Denn diese ist nur für *ARMv6* geeignet.

<a name="raspi-manually"></a>
## Raspberry Pi manuell konfigurieren

Um alles einzurichten, müssen wir zunächst ein Image auf die SD-Karte brennen und uns über eine ssh-Verbindung mit dem Raspberry Pi verbinden. Schau [das Video](https://www.youtube.com/watch?v=upY4Fusi4zI&t=714s) um diese Schritte durchzuführen.

Nachdem alles geklappt hat, verbinde dich mit dem Raspberry Pi und führe die nächsten Schritte aus.

<a name="node"></a>
### **Installiere [Node](https://nodejs.org/en/about/)**

Node.js ist eine Open-Source-Serverumgebung, mit der wir das Backend und damit die Logik für die automatische Bewässerung entwickelt haben. Das Backend ist das Herzstück der Anwendung und verbindet die Sensordaten, die Benutzeroberfläche, die Datenbank und die Hardware (Relais zur Pumpe).

Führe die folgenden Befehle auf dem Raspi aus, um Node zu installieren:

```bash
wget https://nodejs.org/dist/v11.9.0/node-v11.9.0-linux-armv6l.tar.gz
tar -xvf node-v11.9.0-linux-armv6l.tar.gz
cd node-v11.9.0-linux-armv6l
sudo cp -R * /usr/local/
```

Dass die Installation funktioniert hat, kann mit den beiden Befehlen zur Versionsabfrage von Node.js und NPM überprüft werden:

```bash
node -v
npm -v
```

Jetzt sollte dir eine Versionsnummer ausgegeben werden.

<a name="mongodb"></a>
### **Installiere [MongoDB](https://www.mongodb.com/de)**
MongoDB ist eine universelle, dokumentenbasierte, verteilte noSQL-Datenbank, in der wir unsere Einstellungen und Zeitreihendaten speichern werden.

Führe die folgenden Befehle auf dem Raspi aus, um MongoDB zu installieren:

```bash
sudo apt update
sudo apt upgrade

sudo apt install mongodb

sudo systemctl enable mongodb
sudo systemctl start mongodb
```

Dass die Installation funktioniert hat, kann mit dem folgenden Befehl überprüft werden:

```bash
mongo
```

#### 32 Bit Mongo Version 
Bei mir hatte es leider nicht funktioniert, weil ich eine 32Bit Version benötigt habe. 
Dazu kannst du diese Alternative Variante nutzen, welche von [der Seite](https://www.alcher.me/databases-ru-en/mongodb/install-32-and-64-mongodb/) stammt: 

```bash
curl -s https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt update
```

Jetzt noch die eigentliche Installation

```bash
sudo apt install mongodb
```

Teste jetzt nochmal ob es geklappt hat, mit dem Befehl:

```bash
mongo
```

<a name="project-installation"></a>
### **Installing the Project**

Download the project from this repository with the following command and go in the project directory:

```bash
git clone https://github.com/PatrickHallek/automated-irrigation-system
cd automated-irrigation-system
```

After downloading the project you have to create environment files for the frontend and backend with the following commands:

```bash
sudo nano .env
```
If you are in nano edit mode, copy the following text into it and type in your raspi ip. You can find the ip with the command `ifconfig`. It should be something like *192.168.178.44*
```nano
SKIP_PREFLIGHT_CHECK=true
PORT=4200
REACT_APP_BACKEND_URL="http://<YOUR-RASPI-IP>:3000"
```
You can save your input in the Nano editor with `ctr + x`, then type in `yes`, finally exit with `enter`.

We need to do the same for the backend environment:
```bash
sudo nano backend/.env
```
Copy the following line into the editor in order to set the database connection:
```
MONGO_DB="mongodb://localhost/irrigation"
```
In order to install all dependencies in the frontend and backend, you need to run the following 
```bash
npm install
cd backend
npm install
```

If everything is installed, you are able to start the frontend and backend separately 
```bash
npm run build
npm start &
cd backend 
npm start &
```

<a name="usage"></a>
## Usage
The frontend can be accessed at the following URL if you are in your home wifi network:
http://<RASPI_IP>:5000

In order to get the Raspberry Pi IP-address, execute `ifconfig` on the Raspi.
If everything worked out fine, you should see the measurements in the `Last Minute` view in the statistics and the default preferences (which do not equal to 0).

<a name="contributing"></a>
## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

<a name="licence"></a>
## License
[MIT](https://choosealicense.com/licenses/mit/)
