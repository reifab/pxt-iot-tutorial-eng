```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Smart Toilet Tutorial mit Sensor

## 📗 Einführung

**Voraussetzungen**

🌱 IoT Basics abgeschlossen und das Smart-Toilet-Tutorial [Teil 2 – mit Internetverbindung](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/smart-toilet-part2) erfolgreich beendet.

Schwierigkeitsgrad: 🔥🔥⚪⚪

## 👁️ Voraussetzungen @showdialog
* Schliesse den IoT-Cube so an, falls du das noch nicht erledigt hast:
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-anschliessen-klein.png)
* Stelle die Schalter vorerst so ein:
    * Battery Switch: **off**
    * LoRa-Module: **on**
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-power-switches-klein.png)
* Ein LoRa-Gateway🛜 muss in Reichweite und mit TTN (The Things Network) verbunden sein.
  In jedem Klassensatz ist mindestens eines vorhanden, das Hunderte von IoT-Cubes bedienen kann.
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/gateway-klein.png)

## Lernergebnis

Aus den vorherigen Tutorials kennst du bereits, wie man den Status der Toilette 🚽 ("Besetzt" oder "Frei") per LoRa🛜 ins Internet sendet. 
Bisher hast du den Status durch Tastendruck ausgelöst. In einer echten Anwendung übernimmt das ein Sensor – und genau darum geht es in diesem Tutorial.

* Du baust ein Toilettenhäuschen-Modell mit einem Magnetschalter (Magnetic Switch).

Am Ende hast du ein Programm, das …

* eine LoRa-Verbindung🛜 aufbaut
* den Status der Toilette 🚽 über den Magnetschalter erkennt
* den Status 🚽 über LoRa🛜 ins Internet sendet

Brauchbare Funktionen aus [Teil 2](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/smart-toilet-part2) sind schon integriert; das Auswerten der Tasten A und B
wurde hingegen entfernt.

Falls dir am bestehenden Code etwas unklar ist, lohnt es sich,
diesen Teil noch einmal in Ruhe durchzugehen.

## Hardware vorbereiten

* Du bekommst dieses WC-Häuschen-Modell:
  <div>
    <img src="https://reifab.github.io/pxt-iot-tutorial/static/tutorials/wc-haus.png" width="220" alt="WC-Häuschen außen">
    <img src="https://reifab.github.io/pxt-iot-tutorial/static/tutorials/wc-haus-innen.png" width="220" alt="WC-Häuschen innen mit Sensorik">
  </div>

* Falls du das Modell bereits fertig montiert bekommen hast, schliesse den Magnetschalter am IoT-Cube bei **J3** an und mache
beim nächsten Schritt weiter.
* Falls du das Modell nicht hast, kannst du selbst etwas Ähnliches bauen:
  * Für hohe Ansprüche lade das STL-Datei hier herunter: [🌍 STL-3D-Modell](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/3dModel/magnet-schalter-halterung.stl), die
  Du hier anschauen kannst: [🌍 Taster-Halterung](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/3dModel/schalter-halterung.html)
  * Den Halter kannst du mit einem 3D-Drucker ausdrucken
  * Baue die Sensorik sowie einen Magneten🧲 in dein Modell ein (ähnlich wie im Bild rechts oben). 
  Achte darauf, dass der Magnet im geschlossenen Zustand nicht genau mittig auf dem Magnetschalter liegt, sondern leicht nach rechts oder links versetzt zum länglichen, schwarzen Schalterteil positioniert ist.
   
## Magnetschalter auslesen 🧲
Der Magnetschalter (Magnetic Switch) liefert ein digitales Signal,
das wir einfach auswerten können:

0 → kein Magnet in der Nähe → Tür offen

1 → Magnet 🧲 in der Nähe → Tür geschlossen 

Damit wir den Zustand der Tür laufend erkennen, lesen wir den Sensor in der
bestehenden ``||basic:dauerhaft||``-Schleife regelmässig
aus und übersetzen das Signal in "offen" oder "geschlossen".

* ``||variables:Erstelle eine Variable...||`` und benenne sie **zustandTür**.
* Setze die Variable **zustandTür** zuoberst in der
``||basic:dauerhaft||``-Schleife auf den Zustand, der am Pin P2 gemessen wird. Verwende
  dazu die Blöcke ``||variables:setze zustandTür auf 0||`` sowie
   ``||SmartfeldSensoren:erkenne Magnetfeld||`` (unter •••Mechanische Sensoren) und ersetze die 0 durch den "erkenne Magnetfeld"-Block.
* Stelle im "erkenne Magnetfeld"-Block P0 auf **P2** und kontrolliere, ob du den Magnetschalter an **J3** angeschlossen hast.

```blocks
//@hide
function macheFrei () {
    statusFreiOderBesetzt = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function macheBesetzt () {
    statusFreiOderBesetzt = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function sendeDaten (status: number) {
    if (control.millis() > msBeiLetztemSenden + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        spaeterSenden = false
        msBeiLetztemSenden = control.millis()
    } else {
        spaeterSenden = true
    }
}

basic.forever(function () {
    //@highlight
    zustandTür = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    if (spaeterSenden) {
        sendeDaten(statusFreiOderBesetzt)
    }
})
```

## Logik zum Senden 🛜

Wenn die Tür geschlossen ist, nehmen wir an, das WC sei **Besetzt**. Andernfalls
nehmen wir an, das WC sei **Frei**. Diese Zustände wollen wir einerseits
anzeigen, andererseits über LoRa🛜 in die Claviscloud schicken, was 
zum Glück beides in Form einer Funktion schon vorbereitet ist.

* Setze unter das Auslesen des Magnetschalters einen
  ``||logic:wenn wahr dann||``-Block.
* Prüfe mit ``||logic:Vergleich 0 = 0||``, ob ``||variables:zustandTür||``
  den Wert 1 hat (Tür geschlossen). Den Vergleich setzt du in den ``||logic:wenn wahr dann||``-Block
  anstelle von **wahr** ein.
* Wenn dies der Fall ist, rufe die Funktion ``||function:Aufruf macheBesetzt||`` auf,
  andernfalls ``||function:Aufruf macheFrei||``. (Die Funktionen findest du im Bereich
  **Fortgeschritten** – klappe ihn bei Bedarf zuerst auf.)
* Klicke auf 📥 `|Download|`.
* Prüfe, ob in der Cloud die Änderung des Zustands (frei oder besetzt) angezeigt wird:
  [iot.claviscloud.ch](https://iot.claviscloud.ch/dashboards/)
* Behebe gegebenenfalls aufgetretene Fehler. Klicke auf das 💡-Symbol bei Schwierigkeiten.
* Fällt dir sonst noch etwas auf? Gibt es Dinge, die du optimieren könntest?


```blocks
//@hide
function macheFrei () {
    statusFreiOderBesetzt = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function macheBesetzt () {
    statusFreiOderBesetzt = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function sendeDaten (status: number) {
    if (control.millis() > msBeiLetztemSenden + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        spaeterSenden = false
        msBeiLetztemSenden = control.millis()
    } else {
        spaeterSenden = true
    }
}

basic.forever(function () {
    zustandTür = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    //@highlight
    if (zustandTür == 1) {
        macheBesetzt()
    } else {
        macheFrei()
    }
    if (spaeterSenden) {
        sendeDaten(statusFreiOderBesetzt)
    }
})
```

## Optimieren 🪫

Im Moment wird alle 5 Sekunden gesendet, auch ohne Zustandswechsel. 
Das kostet unnötig Energie. Sinnvoller ist es, nur bei einer Änderung 
des Türzustands zu senden.

* ``||variables:Erstelle eine Variable...||`` und benenne sie **zustandTürDavor**.
* Setze ganz am Ende im Block ``beim Start`` die Variable
  ``||variables:zustandTürDavor||`` auf -1, damit sie sich beim ersten Durchlauf garantiert
  vom gemessenen Wert unterscheidet: ``||variables:setze zustandTürDavor auf -1||``
* Prüfe in der ``||basic:dauerhaft||``-Schleife, vor **wenn späterSenden dann**, ob sich die Variablen ``||variables:zustandTür||`` und ``||variables:zustandTürDavor||``
  unterscheiden (≠-Vergleich). Wenn ja, aktualisiere **zustandTürDavor**
  und führe nur dann die bestehende Logik aus. Gehe z.B. so vor:
  * Nimm den Block ``||logic:wenn wahr dann||`` sowie ``||logic:0 ≠ 0||`` (Für den Vergleich der beiden Variablen)
  * Ersetze die Nullen mit den beiden Variablen ``||variables:zustandTür||`` und ``||variables:zustandTürDavor||``
  * Wenn die Bedinung erfüllt ist ``||variables:setze zustandTürDavor auf zustandTür||``
  * Verschiebe nun deine bisherige Abfrage (Tür = 1) in diesen neuen Wenn-Block. 
  Selektiere dazu den zu verschiebenden Wenn-Dann-Block, drücke ctrl+X zum ausschneiden und ctrl+V zum einsetzen. 
  Dadurch wird das Anzeigen und Senden nur noch ausgeführt, wenn sich der Türzustand ändert.

```blocks
//@hide
function macheFrei () {
    statusFreiOderBesetzt = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function macheBesetzt () {
    statusFreiOderBesetzt = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
//@hide
function sendeDaten (status: number) {
    if (control.millis() > msBeiLetztemSenden + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        spaeterSenden = false
        msBeiLetztemSenden = control.millis()
    } else {
        spaeterSenden = true
    }
}
let statusFreiOderBesetzt = 0
let spaeterSenden = false
let msBeiLetztemSenden = 0
IoTCube.LoRa_Join(
eBool.enable,
eBool.enable,
10,
8
)

while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
    basic.showLeds(`
        # . # . #
        # . # . #
        # # # # #
        . . # . .
        . . # . .
        `)
    basic.pause(1000)
}
basic.showIcon(IconNames.Yes)
basic.pause(5000)
msBeiLetztemSenden = control.millis()
spaeterSenden = false
let zustandTür = 0
macheFrei()
//@highlight
let zustandTürDavor = -1
basic.forever(function () {
    zustandTür = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    //@highlight
    if (zustandTür != zustandTürDavor) {
        zustandTürDavor = zustandTür
        if (zustandTür == 1) {
            macheBesetzt()
        } else {
            macheFrei()
        }
    }
    if (spaeterSenden) {
        sendeDaten(statusFreiOderBesetzt)
    }
})
```

## Gratuliere 🏆 – du hast das Tutorial erfolgreich bearbeitet 🚀

* Falls noch nicht gemacht, verbinde deine smarte Toilette mit dem Toiletten-Widget 
der [Claviscloud](https://iot.claviscloud.ch/).
* Teste, ob die Daten auf dem LED-Display sowie in der Cloud☁️ korrekt angezeigt werden.
* Falls etwas noch nicht richtig läuft, findest du hier eine funktionierende Version zum Testen: 
[Lösung Teil 3](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/smart-toilet-part-3-solution)

```template
function macheFrei () {
    statusFreiOderBesetzt = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
function macheBesetzt () {
    statusFreiOderBesetzt = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
function sendeDaten (status: number) {
    if (control.millis() > msBeiLetztemSenden + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        spaeterSenden = false
        msBeiLetztemSenden = control.millis()
    } else {
        spaeterSenden = true
    }
}
let statusFreiOderBesetzt = 0
let spaeterSenden = false
let msBeiLetztemSenden = 0
IoTCube.LoRa_Join(
eBool.enable,
eBool.enable,
10,
8
)
while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
    basic.showLeds(`
        # . # . #
        # . # . #
        # # # # #
        . . # . .
        . . # . .
        `)
    basic.pause(1000)
}
basic.showIcon(IconNames.Yes)
basic.pause(5000)
msBeiLetztemSenden = control.millis()
spaeterSenden = false

macheFrei()

basic.forever(function () {
    if (spaeterSenden) {
        sendeDaten(statusFreiOderBesetzt)
    }
})

```
