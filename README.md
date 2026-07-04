# Heidelberg Energy Control PV-Überschussladen ☀️🚗 mit ESP32 und Home Assistant / ESPHome

Dieses Projekt ermöglicht eine materialschonende, dynamische PV-Überschusssteuerung für die Heidelberg (Amperfied) Energy Control Wallbox via Modbus (ESP32) und Home Assistant. 

*Hinweis: Grundwissen beim Arbeiten an elektrischen/elektronischen Geräten wird vorausgesetzt.*

## Features
- **Direkte Stromstärken-Regelung**: Das Modbus-Register 261 ermöglicht eine sehr schnelle Änderung der Ladeleistung.
- **Echtzeit-Ampere**: Home Assistant und ESPHome kommunizieren nativ in echten Ampere-Werten (6.0 bis 16.0 A).
- **Schonendes An- und Abschalten**: Nutzt das Modbus-Register 259 für die Ladefreigabe, um den Verschleiß der internen Box-Schütze zu verhindern (kein Schalten unter Last).
- **Wolkenschutz**: Eine 3-minütige Verzögerung in Home Assistant verhindert hektisches An- und Ausschalten der Box bei vorbeiziehenden Wolken.
- **Fehlerfreies Log**: Kein Absturz des Modbus-Watchdogs der Wallbox im Standby durch ein intelligentes 30-Sekunden-Lebenszeichen.
- **Saubere Integration**: Der ESP32 ist im Home Assistant sichtbar, die Wallbox läuft komplett im Hintergrund.

## Hardware
- **Heidelberg / Amperfied Energy Control**: Verfügt ab Werk nur über eine serielle RS485 Modbus-RTU-Schnittstelle und ist nicht netzwerkfähig.
- **ESP32**: Wird als Gateway benötigt. Ein ESP8266 verfügt leider nicht über die benötigten Ressourcen für die vollständige Funktion.
- **UART TTL zu RS485 Adapter**: Verbindet die seriellen Pins des ESP32 mit dem Bus der Wallbox.
- **Geschirmte Datenleitung**: Zwecks störungsfreier Übertragung des Modbus-Signals zur Wallbox.

## Prinzip
Der ESP32 wird über ESPHome geflasht und dient rein als Brücke (Schnittstelle) zur Wallbox. Er verarbeitet die Signale aus Home Assistant zu Modbus-RTU-Signalen. 
Home Assistant übernimmt die komplette Logik und Regelung basierend auf deiner PV-Sensorik. Der ESP32 selbst hat keine Überwachungs- oder Regelfunktion.

## Verkabelung

### 1. ESP32 zu RS485-Adapter
- **TX-Pin** (Adapter) $\rightarrow$ **GPIO16** (ESP32)
- **RX-Pin** (Adapter) $\rightarrow$ **GPIO17** (ESP32)
- **GND** (Adapter) $\rightarrow$ **GND** (ESP32)
- **VCC** (Adapter) $\rightarrow$ **3.3V oder 5V** (je nach Ausführung des Adapters)
- **Umschaltkontakt (DE/RE)** am Adapter $\rightarrow$ **GPIO4** (ESP32)

### 2. RS485-Adapter zu Wallbox
- Verbinde die Kontakte **A** und **B** des Adapters mittels geschirmter Leitung mit den Kontakten **A+B** (serieller Anschluss) auf der Platine der Wallbox.

### 3. Hardware-Freigabe (Potentialfreier Kontakt)
- **GPIO21** des ESP32 steuert ein Relais an.
- Die sekundären Kontakte des Relais werden an die **PF-Kontakte** der Wallbox angeschlossen. Dieser Kontakt dient im Alltag als "Hauptschalter" oder dauerhafte Sperre. Bitte entnehme die genauen Anschlussklemmen der Dokumentation des Herstellers.

## Installation & Inbetriebnahme

❗ **SICHERHEITSHINWEIS:** Sorge für deine Sicherheit und schalte vor der Montage den Strom im Sicherungskasten komplett ab. Vor allem, wenn die Wallbox geöffnet ist!

1. Flash den ESP32 mit der Datei `wallbox-esphome.yaml`. *Tipp: Nutze dafür die ESPHome-Dashboard oder die Desktop-Version, um spätere Updates bequem über WLAN (OTA) einzuspielen.*
2. Trage deine WLAN-Daten in den Geheimnis-Sektor (`!secret`) ein.
3. Schalte die Geräte ein. Sobald der ESP32 im Netzwerk sichtbar ist, zeigt das ESPHome-Protokoll (Logger) die ausgelesenen Register an.
4. **Inbetriebnahme-Test**: Überprüfe, ob die Werte für Spannungen und Ladestatus im Log erscheinen. Versuche das Register 261 manuell über das UI zu beschreiben. Wenn sich die Stromstärke im Bereich von 6.0 bis 16.0 A ändern lässt, steht die Hardware-Verbindung erfolgreich!
5. Füge die Sensoren und Automationen aus der Datei `automations.yaml` in dein Home Assistant ein. Passe die PV-Sensoren an deine eigene PV-Anlage an (Tipp: Nutze eine KI wie ChatGPT oder Claude, um die Sensor-Namen schnell auf dein System umzuschreiben).

