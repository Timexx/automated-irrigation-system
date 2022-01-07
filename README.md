[![](https://img.shields.io/badge/Buy%20me%20-coffee!-orange.svg?logo=buy-me-a-coffee&color=795548)](https://buymeacoff.ee/PatrickHallek)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![PRs welcome!](https://img.shields.io/badge/contributions-welcome-green.svg?style=flat)](https://github.com/PatrickHallek/automated-irrigation-system/issues)

# Automatisches Bewässerungssystem von Patrick Hallek

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
5. [ Setup the Raspberry Pi mit Docker (empfohlen) ](#raspi-docker)
6. [ Setup the Raspberry Pi manuell ](#raspi-manually)
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
## Setup the NodeMCU ESP8266

To flash the NodeMCU microcontroller you have to follow the steps described in [this video](https://www.youtube.com/watch?v=flLMMHCNkQE).

Before you upload the program you have to set your wifi password, wifi name (ssid), the ip of the raspberry pi (host) and the sensor name. The sensor name will be the name that is displayed in the app. So it's best to choose the name of the plant the sensor should be associated with.

If the Arduino IDE is successfully configured for the NodeMCU, you can upload the program you find in this repository under [arduino-code/ESP8266_moisture/ESP8266_moisture.ino](https://github.com/PatrickHallek/automated-irrigation-system/blob/master/arduino-code/ESP8622_moisture.ino/ESP8622_moisture.ino.ino) to the NodeMCU.


<a name="raspi-docker"></a>
## Setup the Raspberry Pi with [Docker](https://www.docker.com/) (recommended)

To avoid having to install the required programs manually, you can also run the application with Docker in containers. To do this, carry out the following steps:

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
sudo apt-get install -y libffi-dev libssl-dev
sudo apt-get install -y python3 python3-pip
sudo apt-get remove python-configparser
sudo pip3 install docker-compose
```

Now you have to pass the ip address of your pi into the `REACT_APP_BACKEND_URL=http://<YOUR-RASPI-IP>:3000` environment variable in the docker-compose file:

```bash
sudo nano docker-compose.yml
```

You can find the ip with the command `ifconfig`. It should be something like *192.168.178.44*. You can save your input in the Nano editor with `ctr + x`, then type in `yes`, finally exit with `enter`.

Now everything should be ready and you can start the application with the following command:

```bash
sudo docker-compose up
```

**Attention**: If you have a Raspberry Pi with a processor other than *ARMv7*, you need to adjust the image for the mongodb in the docker-compose file. Since this is only suitable for *ARMv6*.

<a name="raspi-manually"></a>
## Setup the Raspberry Pi manually

To set everything up we first have to burn an image to the SD card and connect to the Raspberry pi via an ssh connection. Follow [this video](https://www.youtube.com/watch?v=upY4Fusi4zI&t=714s) to perform these steps.

After everything has worked out, connect to the raspberry pi and take the next steps

<a name="node"></a>
### **Installing [Node](https://nodejs.org/en/about/)**

Node.js is an open source server environment with which we developed the backend and thus the logic for automated irrigation. The backend is the heart of the application and connects the sensor data, user interface, database and hardware (relay to the pump).

Execute the following commands on the raspi in oder to install Node:

```bash
wget https://nodejs.org/dist/v11.9.0/node-v11.9.0-linux-armv6l.tar.gz
tar -xvf node-v11.9.0-linux-armv6l.tar.gz
cd node-v11.9.0-linux-armv6l
sudo cp -R * /usr/local/
```

That the installation has worked can be checked with the two commands for version query of Node.js and NPM:

```bash
node -v
npm -v
```

<a name="mongodb"></a>
### **Installing [MongoDB](https://www.mongodb.com/de)**
MongoDB is a universal, document-based, distributed noSQL database where we will store our settings and time series data.

Execute the following commands on the raspi in oder to install MongoDB:

```bash
sudo apt update
sudo apt upgrade

sudo apt install mongodb

sudo systemctl enable mongodb
sudo systemctl start mongodb
```

That the installation has worked can be checked with the command below:

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
