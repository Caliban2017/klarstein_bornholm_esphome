# Klarstein Bornholm 2500W Konvektionsheizer MCU Austausch

In den neuen Klarstein Bornholm Modellen mit 2500W werden nicht mehr wie bei der vorherigen 2000W Version auf ESP basierende MCU's verbaut. In den neueren Modellen befindet sich ein auf RT Chip, welcher nicht mehr ohne weiteres mit Tasmota geflashed werden kann. Glücklicherweise lässt sich die Tuya WBR3 MCU jedoch gegen ein ESP8266- oder ESP-12F Board austauschen. Die Kommunikation findet auch hier wie beim Vorgänger Seriell statt.

Natürlich lässt sich das ganze auch problemlos per Tasmota betreiben, noch besser funktioniert es jedoch für meinen Geschmack mit ESPHome. Hier ist kein rumgefummel mit dem Backlog und IDs nötig. ESPHome liest die ID's von dem Tuya Gerät aus und stellt sie dann mit den Datenpunkten zur Verfügung. Man muss anschließend nur noch rausfinden, welche ID zu welcher Funktion gehört. Dies habe ich bereits gemacht. Einfach meine ESPHome Vorlage benutzen.

Vorbereitung:
Zunächst muss das Gerät geöffnet werden, dazu werden die Schrauben Seitlich am Gerät entfernt. Davon gibt es an beiden Seiten jeweils zwei Stück. Nachdem man die Front-Glasscheibe entfernt hat, kann man auf der rechten Seite unter der Segment-Anzeige eine kleine Platine mit der blauen WBR3 MCU sehen. Dafür muss man nur eine Schraube lösen. Anschließend kann man den Stecker samt Platine von der Hauptplatine des Bornholm lösen.

Die alte Tuya MCU muss nun ausgelötet werden. Am besten funktioniert dies mit einer Heißluftstation, alternativ kann man es aber auch mit Entlötlitze oder Entlötpumpe probieren. Natürlich darauf achten keine Lötpads zu zerstören!

![Foto der Platine mit der alten MCU](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/bild1.png?raw=true)

Ich habe für meinen Test einen Wemos D1 Mini benutzt, da ich diesen gerade zur Hand hatte. Auf diesem wird zunächst ESPHome aufgespielt. Dazu kann die von mir zur Verfügung gestellte Vorlage benutzt werden, die schon alle Datenpunkte für die UART Verbindung zur Verfügung stellt.

Verbindungsschema
------------------------------

Jetzt wird der neue ESP8266 mit den Lötpads der Bornholm Platine verbunden, so wie auf dem oberen Bild zu sehen ist. Alternativ sollte auch ein ESP-12F funktionieren, dieser kann einfach 1:1 mit der alten MCU getauscht werden, da dass Pinout kompatibel ist. Dies habe ich jedoch noch nicht getestet, spricht aber theoretisch nichts dagegen.

Ansonsten werden die Kabel wie folgt angelötet:

Wemos/ESP8266 -> TuyaMCU Board

3,3V > VCC

GND > GND

RX > TXD

TX > RXD

![Anschlussschematik](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/schematic1.png?raw=true)

ESPHome Vorlage
------------------------------

Spätestens jetzt die ESPHome Vorlage auf den ESP flashen. Natürlich muss diese nach eigenen Wünschen noch angepasst werden. Zur Verfügung steht eine Deutsche, sowie eine Englische Version. Ich rate natürlich ESPHome schon vor den Einbau in das Gerät aufzuspielen, damit man hinterher jegliche Änderungen OTA auf den ESP aufspielen kann.

Die Vorlage stellt drei Schalter zur Verfügung: 
- Heizen: An/Aus
- Display-Dimmer: An/Aus
- Kindersperre: An/Aus

Folgende Sensoren:
- Raumtemperatur

Folgende Nummern (die als Slider eingestellt werden können):
- Zieltemperatur (05-45°C)
- Heizmodus (0 = Maximaler Modus (2500W), 1 = Mittlerer Modus (1000W), 2 = Minimaler Modus)
- Timer (0-24 Stunden, dabei ist 0 zum ausschalten des Timer)

Ein Beispiel für die Einbindung ins Dashboard:
------------------------------

![Beispiel HA](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/ha_1.jpg?raw=true)

![Beispiel HA](https://github.com/Caliban2017/klarstein_bornholm_esphome/blob/main/ha_2.jpg?raw=true)

Dafür wird benötigt:

[Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)

[Bubble Cards](https://github.com/Clooos/Bubble-Card)

[Custom Card](https://github.com/thomasloven/lovelace-card-mod)


Code für die Bubble Card:
```
type: vertical-stack
cards:
  - type: custom:bubble-card
    card_type: pop-up
    hash: "#bornholm"
    name: Klarstein Bornholm
    icon: mdi:heat-pump
  - type: custom:mushroom-entity-card
    entity: switch.esphome_klarstein_power
    tap_action:
      action: toggle
    name: Heizmodus
  - type: custom:mushroom-number-card
    entity: number.esphome_klarstein_zieltemperatur
    name: Zieltemperatur
  - type: custom:mushroom-number-card
    entity: number.esphome_klarstein_heizmodus
    name: Heizstufe
    primary_info: name
    layout: horizontal
    card_mod:
      style:
        mushroom-state-info$: |
          .secondary {
             visibility: hidden;
           }
           .secondary:before {
             visibility: visible;
             content:
               {% set sensor = states('number.esphome_klarstein_heizmodus') %}
               {% if sensor | int == 0 %}"Maximum"
               {% elif sensor | int == 1 %}"Medium"
               {% elif sensor | int == 2 %}"Minimum"
               {% endif %}
               
          }
  - type: custom:mushroom-number-card
    entity: number.esphome_klarstein_timer_stunden
    name: "Timer (Stunden):"
    layout: horizontal
```
Für das Dashboard habe ich die Mushroom-Entity Card benutzt;
```
type: custom:mushroom-entity-card
entity: switch.esphome_klarstein_power
name: Klarstein Bornholm
layout: horizontal
tap_action:
  action: navigate
  navigation_path: "#bornholm"
hold_action:
  action: toggle
card_mod:
  style:
    mushroom-state-info$: |
      .secondary {
         visibility: hidden;
       }
       .secondary:before {
         visibility: visible;
         content:
           "{{ 'Modus: ' }}"
           {% set sensor = states('switch.esphome_klarstein_power') %}
           {% if sensor == 'on' %}"An "
           {% else %}"Aus "
           {% endif %}
          "{{ "| Temperatur: " + states('sensor.esphome_klarstein_temperatur', with_unit=True) + " | Zieltemp: " + states('number.esphome_klarstein_zieltemperatur') + " °C" }}";
       }
```
