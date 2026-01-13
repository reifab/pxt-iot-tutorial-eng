```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# IoT Tutorial Teil 3


## 📗 Einführung, Teil 3

Voraussetzungen: 🌱 IoT Basics abgeschlossen und IoT Tutorial [Teil 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/seifenspender-part-2) abgeschlossen.
Schwierigkeitsgrad: 🔥🔥🔥⚪

Aus dem Tutorial Teil 2 hast du bereits ein Programm, das den Seifenstand mit Knopf A und B 
simuliert und über LoRa🛜 ins Internet sendet. Um der realen Anwendung näher zu kommen, 
ersetzt Du die Knöpfe A und B durch einen Ultraschallsensor, der den Seifenstand misst.

Am Schluss hast du ein Programm, welches...

* Den Seifenstand mit einem Ultraschallsensor🦇 misst.
* Eine LoRa-Verbindung🛜 aufbaut. 
* Den Seifenstand🧼 über LoRa🛜 sendet. 
* Eine Ladebalken-Animation⏳ für Wartezeiten darstellt.

## 👁️ Voraussetzungen @showdialog

Du brauchst gegenüber Tutorial Teil 2 folgendes:
* Schliesse den Ultraschallsensor🦇 an J1 an.
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/14_Tutorial_Seifenspender_Ultraschallsensor.png)

## Bisherige Funktion überprüfen

Prüfe, ob die Daten immer noch auf der Clavis Cloud ☁️ ankommen.
* Lade das bestehende Programm auf den IoT Cube
* Rufe die Website [🌍iot.claviscloud.ch](https://iot.claviscloud.ch/home) auf.
* Navigiere zu Deinem Dashboard. 

Wird der🧼 Seifenstand (100 %) auf dem Dashboard angezeigt? Reduziert
sich der Seifenstand beim Drücken von Knopf A?

## Nicht mehr benötigten Code entfernen

Die Knöpfe A und B sollen durch eine reale Messung mittels Ultraschallsensor 
ersetzt werden. Welcher Code wird neu nicht mehr benötigt? 
* Klicke auf die Glühbirne 💡, dann siehst du den Code, wie er ohne die Knöpfe aussieht.
Das Senden des Seifenstands und die Wartefunktion wurden belassen. 
* Passe den Code entsprechend dem Beispiel (Glühbirne 💡) an.

```blocks
// @highlight
basic.forever(function () {  
    led.plotBarGraph(seifenstandInProzent,100)
    IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
    IoTCube.SendBufferSimple()
    warte5SekundenUndZeigeFortschritt()
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Senden vorerst verhindern

Wir wollen vorerst noch nichts an die ☁️ Cloud senden.
* Modifiziere **dauerhaft** so, wie unter der💡angezeigt. 
Verwende dazu den Block ``||Logic:wenn wahr dann||``. 
Dieser verhindert das Senden sobald du wahr auf **falsch** stellst.
* **beim Start** verwendest Du ebenfalls den Block ``||Logic:wenn wahr dann||``,
um das Senden der Daten zu verhindern (Block auf **falsch** stellen).

```blocks
let seifenstandInProzent = 100
led.plotBarGraph(
seifenstandInProzent,
100
)
smartfeldAktoren.oledInit(128, 64)
// @highlight
if (false) {
    initialisiereLoRaVerbindung()
    IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
    IoTCube.SendBufferSimple()
    warte5SekundenUndZeigeFortschritt()
}

basic.forever(function () {
    led.plotBarGraph(seifenstandInProzent,100)
    // @highlight
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}

// @hide
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
        smartfeldAktoren.oledWriteStr(".")
        basic.pause(1000)
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
```

## Ultraschallsensor verwenden

* Schliesse den Ultraschallsensor🦇  an J1 an. 
* Um die gemessene Distanz 📏 zwischen Seife und Sensor zu speichern, benötigen wir eine Variable.
``||variables:Erstelle eine Variable...||`` und benenne sie mit **distanzSensorZuSeife** .
* Ziehe den Block ``||variables:setze distanzSensorZuSeife auf 0||`` zuoberst in die Dauerhaft-Schleife .
* Um der Variable den Messwert zuzuweisen, füge den Block ``||SmartfeldSensoren:Distanz in cm||``
anstelle der 0 ein. Belasse den Pin auf P0.

```blocks
basic.forever(function () {
    // @highlight
    distanzSensorZuSeife = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    led.plotBarGraph(
    seifenstandInProzent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Messwert anzeigen

Der Sensor liefert die Distanz in cm. Um die Funktion des Sensors zu überprüfen,
zeigen wir den Sensorwert auf dem OLED- Display 🖥️ an. 

* Setze den Block 🖥️ ``||SmartfeldAktoren:Lösche Displayinhalt||``
zuoberst in der Dauerhaft-Schleife ein.
Damit löschst du bestehende Inhalte auf dem OLED- Display 🖥️.
* Unter der Variable setzt du den Block ``||SmartfeldAktoren:schreibe Nummer||``
ein. 
* Ersetze die 0 mit der Variable ``||variables:distanzSensorZuSeife||``
* Runde den Wert auf ganze Zahlen mit dem Block ``||math:runden||``
* Schreibe nach dem Messwert die Masseinheit **cm** auf das OLED- Display 🖥️ mit ``||SmartfeldAktoren:schreibe String||``
* Drücke 📥`|Download|` und kontrolliere die Anzeige am OLED Display 🖥️. Wird die Distanz zwischen Sensor und Hindernis angezeigt?

```blocks
basic.forever(function () {
    // @highlight
    smartfeldAktoren.oledClear()
    // @highlight
    distanzSensorZuSeife = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    // @highlight
    smartfeldAktoren.oledWriteNum(Math.round(distanzSensorZuSeife))
    // @highlight
    smartfeldAktoren.oledWriteStr(" cm")
    led.plotBarGraph(
    seifenstandInProzent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Modell für Seifenspender bauen

Um die Anwendung des Seifenspenders zu testen, bauen wir ein Karton-Modell, 
welches als Halter für den Ultraschallsensor🦇 dient. 
Über einen Schieber kann die Distanz eingestellt werden, welche den Seifenstand🧼 simuliert.

* Lade dir das Modell herunter und drucke es auf A3 aus. 
Empfehlenswert ist es, das Modell auf einen Karton oder dickeres Papier zu kleben. 
Dadurch erhält das Modell mehr Stabilität. Download Vorlage:
[🌍Kartonmodell-Abwicklung](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/Seifenspender_Kartonvorlage_Abwicklung_zum_Ausdrucken_A3.pdf)

* Unter folgendem Links siehst du, wie das Modell aussieht:
 [🌍Kartonmodell](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/3dModel/seifenspender3dModellViewer.html)

## Seifenstand🧼 berechnen

* Aus der Messgrösse distanzSensorZuSeife kannst du den Seifenstand🧼 in % berechnen. Siehst Du die Zusammenhänge, wenn Du das Bild weiter unten studierst?
* Berechne den Seifenstand in Prozent. Nutze ``||math:Mathematik||`` sowie ``||variables:setze seifenstandInProzent auf ... ||``.
Es steht Dir frei, eine weitere Variable für Zwischenresultate zu erstellen.
* Runde den Seifenstand in Prozent auf ganze Zahlen mit ``||math:runden||``
* Seifenstände kleiner 0% sollen auf 0% begrenzt werden. Verwende dazu ``||Logic:wenn wahr dann||``.
* Gib den Seifenstand🧼 in % auf dem OLED- Display 🖥️ aus (Anstelle der Distanz)
* Drücke 📥`|Download|` und kontrolliere die Anzeige am OLED Display 🖥️. Wird der Seifenstand in % angezeigt? Bewege den Schieber und teste das Programm!
* Lass dir mit der Glühbirne 💡 eine mögliche Lösung anzeigen, falls Du kein Erfolg hast. Wenn der Abstand 0 cm ist, bedeutet das 100 %, bei 25 cm bedeutet es 0 %.

<img src="https://reifab.github.io/pxt-iot-tutorial/static/tutorials/Seifenspender_Distanzen.png" width="800">

```blocks
basic.forever(function () {
    smartfeldAktoren.oledClear()
    distanzSensorZuSeife = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    // @highlight
    zwischenresultat = 25 - distanzSensorZuSeife
    // @highlight
    zwischenresultat = zwischenresultat / 25
    // @highlight
    seifenstandInProzent = zwischenresultat * 100
    // @highlight
    seifenstandInProzent = Math.round(seifenstandInProzent)
    // @highlight
    if (seifenstandInProzent < 0) {
        seifenstandInProzent = 0
    }
    // @highlight
    smartfeldAktoren.oledWriteNum(seifenstandInProzent)
    // @highlight
    smartfeldAktoren.oledWriteStr(" %")
    led.plotBarGraph(
    seifenstandInProzent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Senden in die Cloud☁️ bei Änderung des Seifenstandes

Da wir nicht immer, sondern nur dann, wenn sich der Seifenstand ändert, 
diesen an die Cloud senden möchten, müssen wir unser Programm noch etwas erweitern, 
d.h. eine Variable nutzen, um Änderungen zu erkennen. 

* Um Änderungen am Seifenstand zu detektieren, müssen wir jeweils den alten Wert speichern. Dazu benötigen wir eine neue Variable.
``||variables:Erstelle eine Variable...||`` und benenne sie mit **seifenstandAlt** .
* beim Start ``||variables:setze seifenstandAlt auf -1||``, bzw. auf einen Wert, der sich beim ersten Mal von der Realität unterscheidet.
* In der bestehenden ``||basic:dauerhaft||``- Schleife befindet sich eine Logik ``||logic:wenn falsch dann||``,
um das Senden zu verhindern.
Diese baust du für deine Zwecke um: Prüfe mit ``||logic:Vergleich 0 ≠ 0||``, 
ob sich die Variablen ``||variables:seifenstandAlt||`` und 
``||variables:seifenstandInProzent||`` unterscheiden. 
Falls ja, schicke den aktuellen Wert in die Cloud (ist bereits vorhanden) und setze 
``||variables:seifenstandAlt||`` auf ``||variables:seifenstandInProzent||``.
* Setze beim Start den Wahrheitswert der Bedingung wieder auf **Wahr**, 
damit der Verbindungsaufbau wieder ausgeführt wird.

```blocks
let zwischenresultat = 0
let distanzSensorZuSeife = 0
// @highlight
let seifenstandAlt = -1
smartfeldAktoren.oledInit(128, 64)
let seifenstandInProzent = 100
led.plotBarGraph(
seifenstandInProzent,
100
)
smartfeldAktoren.oledInit(128, 64)

if (true) {
    initialisiereLoRaVerbindung()
    IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
    IoTCube.SendBufferSimple()
    warte5SekundenUndZeigeFortschritt()
}
basic.forever(function () {
    smartfeldAktoren.oledClear()
    distanzSensorZuSeife = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    zwischenresultat = 25 - distanzSensorZuSeife
    zwischenresultat = zwischenresultat / 25
    seifenstandInProzent = zwischenresultat * 100
    seifenstandInProzent = Math.round(seifenstandInProzent)
    if (seifenstandInProzent < 0) {
        seifenstandInProzent = 0
    }
    smartfeldAktoren.oledWriteNum(seifenstandInProzent)
    smartfeldAktoren.oledWriteStr(" %")
    led.plotBarGraph(
        seifenstandInProzent,
        100
    )
    // @highlight
    if (seifenstandAlt != seifenstandInProzent) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
        // @highlight
        seifenstandAlt = seifenstandInProzent
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}

// @hide
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
        smartfeldAktoren.oledWriteStr(".")
        basic.pause(1000)
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
```
## Testen, Fehler beheben, optimieren

* Drücke 📥`|Download|`
* Prüfe, ob in der Cloud Änderungen des Seifenstands 🧼 angezeigt werden: [iot.claviscloud.ch](https://iot.claviscloud.ch/dashboards/)
* Behebe gegebenenfalls aufgetretene Fehler. Klicke auf das 💡- Symbol, um den gesamten Code des "Seifenspenders" anzuzeigen.
* Energie sparen: Displays (das LED- sowie das OLED- Display) brauchen relativ viel Strom. Kannst Du den Code optimieren, sodass die beiden Displays nur bei Seifenstand- Änderungen aktiv sind?

```blocks
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
        smartfeldAktoren.oledWriteStr(".")
        basic.pause(1000)
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
let zwischenresultat = 0
let distanzSensorZuSeife = 0
let seifenstandAlt = -1
let seifenstandInProzent = 100
led.plotBarGraph(
seifenstandInProzent,
100
)
smartfeldAktoren.oledInit(128, 64)
if (true) {
    initialisiereLoRaVerbindung()
    IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
    IoTCube.SendBufferSimple()
    warte5SekundenUndZeigeFortschritt()
}
basic.forever(function () {
    smartfeldAktoren.oledClear()
    distanzSensorZuSeife = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    zwischenresultat = 25 - distanzSensorZuSeife
    zwischenresultat = zwischenresultat / 25
    seifenstandInProzent = zwischenresultat * 100
    seifenstandInProzent = Math.round(seifenstandInProzent)
    if (seifenstandInProzent < 0) {
        seifenstandInProzent = 0
    }
    smartfeldAktoren.oledWriteNum(seifenstandInProzent)
    smartfeldAktoren.oledWriteStr(" %")
    led.plotBarGraph(
    seifenstandInProzent,
    100
    )
    if (seifenstandAlt != seifenstandInProzent) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
        seifenstandAlt = seifenstandInProzent
    }
    basic.pause(150)
    basic.clearScreen()
})
```

## Gratuliere 🏆 - du hast das Tutorial für den Seifenspender erfolgreich bearbeitet 🚀

* Falls irgendwas noch nicht richtig läuft, hier hast Du eine funktionierende Version zum testen: [Lösung Teil 3](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/seifenspender-part-3-solution)

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
        smartfeldAktoren.oledWriteStr(".")
        basic.pause(1000)
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Verbunden!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
function warte5SekundenUndZeigeFortschritt () {
    smartfeldAktoren.oledClear()
    for (let fortschritt = 0; fortschritt <= 100; fortschritt++) {
        smartfeldAktoren.oledLoadingBar(fortschritt)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
let seifenstandInProzent = 100
led.plotBarGraph(
seifenstandInProzent,
100
)
smartfeldAktoren.oledInit(128, 64)
initialisiereLoRaVerbindung()
IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
IoTCube.SendBufferSimple()
warte5SekundenUndZeigeFortschritt()
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        if (seifenstandInProzent < 0) {
            seifenstandInProzent = 0
        }
        led.plotBarGraph(
        seifenstandInProzent,
        100
        )
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    if (input.buttonIsPressed(Button.B)) {
        seifenstandInProzent = 100
        led.plotBarGraph(
        seifenstandInProzent,
        100
        )
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        warte5SekundenUndZeigeFortschritt()
    }
    basic.pause(150)
    basic.clearScreen()
})
```
