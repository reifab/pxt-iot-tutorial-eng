```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
neopixel=github:microsoft/pxt-neopixel#v0.7.6
```
### @explicitHints false

# Warteschlangen-Sensorik Teil 2

## 👁️ Voraussetzungen @showdialog
* Schliesse den Cube so an, falls noch nicht gemacht:
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-anschliessen-klein.png)
* Stelle die Schalter vorerst so ein:
    * Battery Switch: **off**
    * LoRa Module: **on**
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-power-switches-klein.png)

* Ein LoRa-Gateway🛜 muss in Reichweite sein, welches mit TTN (The Things Network) verbunden ist.
Dies ist im Klassensatz einmal vorhanden und kann hunderte von IoT-Cubes bedienen.
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/gateway-klein.png)


## Lernergebnis

Aus dem Tutorial Teil 1 hast du bereits ein Programm erstellt, welches die Anzahl an Personen einer Warteschlange automatisiert zählt.
Nun wollen wir die Anzahl an Personen in der Warteschlange über LoRa🛜 ins Internet senden. Am Ende hast du ein funktionsfähiges Programm, das ...

* Eine LoRa-Verbindung🛜 aufbaut. 
* Die automatisch ermittelte Anzahl an Personen in der Warteschlange 👥 über LoRa🛜 sendet. 

Das vollständige Programm aus Teil 1 ist bereits integriert. Zudem findest Du drei Funktionen, welche wir in diesem Tutorial bloss noch anwenden:

* initialisiereLoRaVerbindung (): Baut eine Verbindung zum Internet auf
* warte5SekundenUndZeigeFortschritt (): Warte- Funktion, um nicht zu oft Werte zu senden.
* sendeUndZeigePersonenanzahl (): Sendet die Anzahl Personen in der Warteschlange

Falls dir bezüglich dieser Funktionen etwas unklar ist, überlege, vorgängig nochmals den [Teil 2 des Tutorials ](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/warteschlange-part-2) zu bearbeiten.

## 🛜 Verbindung mit Internet aufbauen

Zu Beginn bauen wir eine Verbindung zum Internet auf. Dazu benötigen wir einen Funktionsaufruf.

* Hol dir den Block ``||functions:Aufruf initialisiereLoRaVerbindung ||`` und ziehe diesen zuunterst in den Block **beim Start**.

* 📥 Drücke `|Download|` und kontrolliere das OLED-Display🖥️:  
Wird auf dem OLED- Display **Verbinde...** angezeigt? <br />
Wird danach **Verbunden!** angezeigt? <br />

```blocks
//@hide
function schreibeInfosAufDisplay (position: number, wert: number, _symbol: string) {
    zeile = "P" + position + ": " + _symbol + " :" + wert
    smartfeldAktoren.oledWriteStrNewLine(zeile)
}
//@hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
//@hide
function initialisiereLoRaVerbindung () {
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbinde")
    IoTCube.LoRa_Join(
    eBool.enable,
    eBool.enable,
    10,
    8
    )
    while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
        basic.pause(1000)
        smartfeldAktoren.oledWriteStr(".")
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
//@hide
function sendeUndZeigePersonenanzahl () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, anzahlPersonenInWarteschlange)
    IoTCube.SendBufferSimple()
    basic.showNumber(anzahlPersonenInWarteschlange)
    warte5SekundenUndZeigeFortschritt()
}
//@hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
let zeile = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
//@highlight
initialisiereLoRaVerbindung()
//@hide
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        if (h_unterschied < 100) {
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})

```

## 🛜 Personenanzahl über LoRa senden

Bis jetzt wird die Personenanzahl in der 'Dauerhaft'- Schleife lediglich angezeigt, aber noch nicht gesendet. Dies wollen wir nun ändern:

* Ersetze den Block ``||basic:showNumber(anzahlPersonenInWarteschlange)||`` durch ``||functions:Aufruf sendeUndZeigePersonenanzahl||``.
* 📥 Drücke `|Download|` und kontrolliere das LED-Anzeige🟥 und das OLED-Display🖥️:  
Teste, ob die Personen beim Platzieren von z.B.drei Duplo- Figuren🦹‍♂️ korrekt gezählt werden. Direkt nach der Anzeige der Zahl müsste auch der Ladebalken auf dem OLED-Display🖥️ für 5 Sekunden erscheinen<br />
🟥🟥🟥🟥🟥  
⬛⬛⬛⬛🟥  
🟥🟥🟥🟥🟥  
⬛⬛⬛⬛🟥  
🟥🟥🟥🟥🟥  

```blocks
//@hide
function schreibeInfosAufDisplay (position: number, wert: number, _symbol: string) {
    zeile = "P" + position + ": " + _symbol + " :" + wert
    smartfeldAktoren.oledWriteStrNewLine(zeile)
}
//@hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
//@hide
function initialisiereLoRaVerbindung () {
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbinde")
    IoTCube.LoRa_Join(
    eBool.enable,
    eBool.enable,
    10,
    8
    )
    while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
        basic.pause(1000)
        smartfeldAktoren.oledWriteStr(".")
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
//@hide
function sendeUndZeigePersonenanzahl () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, anzahlPersonenInWarteschlange)
    IoTCube.SendBufferSimple()
    basic.showNumber(anzahlPersonenInWarteschlange)
    warte5SekundenUndZeigeFortschritt()
}
//@hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
let zeile = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
initialisiereLoRaVerbindung()
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        if (h_unterschied < 100) {
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    //@highlight
    sendeUndZeigePersonenanzahl()
})
```


## ☁️ Dashboard erstellen auf Clavis Cloud 

Nun geht es an die Visualisierung der Daten auf der Clavis Cloud ☁️.
* Rufe die Website [🌍iot.claviscloud.ch](https://iot.claviscloud.ch/home) auf.
* Melde dich an (Login- Informationen kriegst Du von der Lehrperson/ Kursleitung).
* Erweitere dein Dashboard mit einem Warteschlangen- Widget, falls noch nicht vorhanden.
* Wähle im Widget als **Datenquelle** als **Gerät** deinen IoT- Cube (gemäss Aufschrift) aus.
* Wähle als **Data key** **Ganzzahl_ID_0** aus.
* Klicke auf **Speichern** und teste, ob die Anzahl Personen gesendet und dargestellt wird.

## 🪫 Optimierungen

Derzeit werden nach jedem Zähldurchlauf Daten per LoRa übertragen, unabhängig davon, ob sich die Anzahl der wartenden Personen geändert hat. Energiesparender wäre es jedoch, die Daten nur dann zu senden, wenn tatsächlich eine Veränderung in der Anzahl der wartenden Personen festgestellt wurde.

* ``||variables:Erstelle eine Variable...||`` und benenne sie mit **anzahlPersonenVorher**.
* Setze zuunterst im Block ``beim Start`` die Variable ``anzahlPersonenVorher`` auf -1 (damit sie sich beim ersten Mal ganz sicher vom gemessenen wert unterscheidet). Die kannst Du mit ``||variables:setze anzahlPersonenVorer auf -1||`` erreichen.
* Prüfe am Schluss der ``dauerhaft``- Schleife, ob sich die Variablen ``||variables:anzahlPersonenInWarteschlange||`` und ``||variables:anzahlPersonenVorher||``
voneinander unterscheiden (≠ Zeichen). Wenn die Variablen unterschiedlich sind, dann soll die Variable **anzahlPersonenVorher** mit **anzahlPersonenInWarteschlange** gleichgesetzt werden.
Nutze dazu ``||logic:wenn wahr dann||`` sowie ``||logic:0 ≠ 0||`` und 
``||variables:setze anzahlPersonenVorher auf anzahlPersonenInWarteschlange||``

```blocks
//@hide
function schreibeInfosAufDisplay (position: number, wert: number, _symbol: string) {
    zeile = "P" + position + ": " + _symbol + " :" + wert
    smartfeldAktoren.oledWriteStrNewLine(zeile)
}
//@hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
//@hide
function initialisiereLoRaVerbindung () {
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbinde")
    IoTCube.LoRa_Join(
    eBool.enable,
    eBool.enable,
    10,
    8
    )
    while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
        basic.pause(1000)
        smartfeldAktoren.oledWriteStr(".")
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
//@hide
function sendeUndZeigePersonenanzahl () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, anzahlPersonenInWarteschlange)
    IoTCube.SendBufferSimple()
    basic.showNumber(anzahlPersonenInWarteschlange)
    warte5SekundenUndZeigeFortschritt()
}
//@hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
let zeile = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
initialisiereLoRaVerbindung()
//@highlight
let anzahlPersonenVorher = -1
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        if (h_unterschied < 100) {
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    //@highlight
    if (anzahlPersonenInWarteschlange != anzahlPersonenVorher) {
        //@highlight
        sendeUndZeigePersonenanzahl()
        //@highlight
        anzahlPersonenVorher = anzahlPersonenInWarteschlange
    }
})
```

## Gratuliere 🏆 - du hast den Teil 2 erfolgreich bearbeitet 🚀

* Falls irgendwas noch nicht richtig läuft, hier hast Du eine funktionierende Version zum testen: [Lösung Teil 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/warteschlange-sensorik-part-2-solution)


```template
function initialisiereLoRaVerbindung () {
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbinde")
    IoTCube.LoRa_Join(
    eBool.enable,
    eBool.enable,
    10,
    8
    )
    while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
        basic.pause(1000)
        smartfeldAktoren.oledWriteStr(".")
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}

function sendeUndZeigePersonenanzahl () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, anzahlPersonenInWarteschlange)
    IoTCube.SendBufferSimple()
    basic.showNumber(anzahlPersonenInWarteschlange)
    warte5SekundenUndZeigeFortschritt()
}

function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
function schreibeInfosAufDisplay (position: number, wert: number, _symbol: string) {
    zeile = "P" + position + ": " + _symbol + " :" + wert
    smartfeldAktoren.oledWriteStrNewLine(zeile)
}
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
let zeile = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        if (h_unterschied < 100) {
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})

```