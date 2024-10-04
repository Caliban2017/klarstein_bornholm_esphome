# klarstein_bornholm_esphome
Replacement of TuyaMCU WBR3 to ESP8266 for Klarstein Bornholm Heater

In den neuen Klarstein Bornholm Modellen mit 2500W werden nicht mehr wie bei der vorherigen 2000W Version auf ESP basierende MCU's verbaut. In den neueren Modellen befindet sich ein auf RT basierender Chip, welcher nicht mehr ohne weiteres mit Tasmota geflashed werden kann. Glücklicherweise lässt sich die WBR3 MCU jedoch gegen ein ESP8266 oder ESP-12F Board austauschen. Die Kommunikation findet auch hier wie beim Vorgänger Seriell statt.

Natürlich lässt sich das ganze auch problemlos per Tasmota betreiben, noch besser funktioniert es jedoch für meinen Geschmack mit ESPHome für eine nahtlose Einbindung ohne MQTT etc. in Home-Assistant. 

Vorbereitung:
Zunächst muss das Gerät geöffnet werden, dazu werden die Schrauben Seitlich am Gerät entfernt. Davon gibt es an beiden Seiten jeweils zwei Stück. Nachdem man die Front-Glasscheibe entfernt hat, kann man auf der rechten Seite unter der Segment-Anzeige eine kleine Platine mit der blauen WBR3 MCU sehen.

Diese MCU muss nun ausgelötet werden. Am besten funktioniert dies mit einer Heißluftstation, alternativ kann man es aber auch mit Entlötlitze oder Entlötpumpe probieren. Dabei aber natürlich darauf achten nicht die Lötpads zu zerstören!

![alt text](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/bild1.png?raw=true)

Ich habe für meinen Test einen Wemos D1 Mini benutzt. Auf diese wird zunächst ESPHome aufgespielt, dazu kann die von mir zur Verfügung gestellte Vorlage benutzt werden, die schon alle Datenpunkte für die UART Verbindung zur Verfügung stellt. Die Verkabelung erfolgt so:

Wemos/ESP8266 -> TuyaMCU Board
------------------------------
3,3V > VCC
GND > GND
RX > TXD
TX > RXD

![alt text](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/schematic1.png?raw=true)

Spätestens jetzt die ESPHome Vorlage auf den ESP flashen. Natürlich muss diese nach eigenen Wünschen noch angepasst werden. Zur Verfügung steht eine Deutschne, sowie eine Englische Version.

Die Vorlage stellt drei Schalter zur Verfügung: 
- Heizen An- Aus
- Display-Dimmer An- Aus
- Kindersperre An- Aus

Folgende Sensoren:
- Raumtemperatur

Folgende Nummern (die als Slider eingestellt werden können):
- Zieltemperatur (05-45°C)
- Heizmodus (0 = Maximaler Modus (2500W), 1 = Mittlerer Modus (1000W), 2 = Minimaler Modus)
- Timer (0-24 Stunden, dabei ist 0 zum ausschalten des Timer)
