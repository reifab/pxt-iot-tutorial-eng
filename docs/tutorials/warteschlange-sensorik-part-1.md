```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
neopixel=github:microsoft/pxt-neopixel#v0.7.6
```
### @explicitHints false

# Warteschlangen-Sensorik Teil 1

## 🎬 Messprinzip 

Wir beleuchten nacheinander bestimmte Positionen mit einer LED💡und messen 
mit dem Lichtsensor👁️, wie viel Licht jeweils ankommt.
Steht etwas zwischen Sensor👁️ und LED💡, zum Beispiel eine Lego-Figur🦹‍♂️, 
verringert sich die Helligkeit.
So erkennen wir, ob an einer bestimmten Position etwas im Lichtstrahl steht.
Die Messungen werden an neun Positionen in der Warteschlange nacheinander 
vorgenommen, sodass die Lego-Figuren🦹‍♂️gezählt werden können.

Schau Dir dieses Video an, welches das Messprinzip illustriert:
* [Video🎬 ansehen: Warteschlange Sensorik](https://wiki.smartfeld.ch/lib/exe/fetch.php?media=warteschlange_sensorik.mp4)

## 👁️ Hardware- Voraussetzungen @showdialog
* Du brauchst dieses Kunststoffteil: [Wartebereich als Kunststoffteil](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/3dModel/warteschlange3dViewer.html)
* Falls du das Teil selbst 3D- Drucken möchtest, lade das STL- File hier herunter: [🌍STL-3D-Modell](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/3dModel/Wartebereich.stl)
* Schliesse den RGB-LED-Streifen💡 an **J7** an.
* Verbinde den Lichtsensor👁️ an **J0** mit dem IoT Cube.
* Schliesse das OLED- Display 🖥️  an **J5** an. 

## 💡 LEDs in Betrieb nehmen

Wir wollen zu Beginn die erste LED, welche im 3D- Modell hinter einem Loch
hervorscheint blinken lassen. Auf dem 16er- LED streifen wäre dies die LED
mit Index **2**.

* ``||basic:beim Start||`` initialisierst Du den LED-Streifen mit ``||neopixel:setze strip auf NeoPixels an Pin||`` **P1** mit **16** Pixeln und im Modus RGB.
* Unter **...mehr** findest du den Block ``||neopixel:setzeHelligkeit||``, womit du die Helligkeit auf 255 setzt.
* In der ``dauerhaft``- Schleife verwendest Du den Block ``||neopixel:strip setze Farbe von Neopixel||`` **2** auf **schwarz**  (unter **...mehr**)
* darunter verwendest Du den Block ``||neopixel:strip anzeigen||``
* ``||basic:pausiere (ms)||`` nach dem Anzeigen für 100 ms
* danach den Block ``||neopixel:strip setze Farbe von Neopixel||`` **2** auf **weiss**  (unter **...mehr**)
* darunter verwendest Du den Block ``||neopixel:strip anzeigen||``
* ``||basic:pausiere (ms)||`` nach dem Anzeigen für 200 ms
* 📥 Drücke `|Download|` und kontrolliere, ob die LED 2 (erste LED in der Warteschlange) 
blinkt.

```blocks
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    basic.pause(200)
})
```

## 🔍 Helligkeit messen und anzeigen auf OLED- Display 

Nun messen wir mit dem Sonnenlichtsensor 👁️ zweimal die Helligkeit,
einmal mit LED- Beleuchtung und einmal ohne (nur Fremdlicht) und zeigen
die Werte auf dem Display an.

* ``||basic:beim Start||`` initialisierst du das Display mit 🖥️ ``||SmartfeldAktoren:init OLED Breite 128 Höhe 64||`` 
*  ``||basic:beim Start||`` initialisierst Du den Sonnenlichtsensor mit ``||SmartfeldSensoren:init Sonnenlicht Sensor||``
* Setze den Block 🖥️ ``||SmartfeldAktoren:Lösche Displayinhalt||``
zuoberst in der Dauerhaft-Schleife ein.
*  ``||variables:Erstelle eine Variable...||`` und benenne sie mit **h_umgebung**.
* In der ``dauerhaft`` - Schleife verwendest du den Block ``||variables:setze lichtmessung auf 0||``
* ersetze die 0 durch eine Messung mit ``||SmartfeldSensoren:gib sichtbares Licht [lm]||`` und
führe diese Messung direkt nach dem ersten **strip anzeigen** durch.
* Unter der Messung setzt du den Block ``||SmartfeldAktoren:schreibe Nummer||``
ein. 
* Ersetze die 0 mit der Variable ``||variables:h_umgebung||``
* Setze den Block  ``||SmartfeldAktoren:schreibe String "-" ||``ein, um einen Bindestrich
(als Trennzeichen zum nächsten Messwert) auf dem Display🖥️ auszugeben.
* ``||variables:Erstelle eine Variable...||`` und benenne sie mit **h_mitLED**.
* Wiederhole die Messung sowie die Anzeige unter Verwendung der Variable **h_mitLED**. Die Messung kannst Du nach dem zweiten **strip anzeigen** einfügen. 
* 📥 Drücke `|Download|` und beobachte die Werte auf dem Display🖥️ . Beantworte
für dich folgende Fragen:
  * Wie gross ist der Unterschied der Messwerte (Umgebungslicht - Licht mit LED)
  * Wie stark lassen sich die Werte von Fremdlicht beeinflussen?
  * Wie stark streuen die Messwerte bei scheinbar konstantem Fremdlicht?

```blocks
let h_umgebung = 0
let h_mitLED = 0
// @highlight
smartfeldSensoren.initSunlight()
// @highlight
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    // @highlight
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    // @highlight
    h_umgebung = smartfeldSensoren.getHalfWord_Visible()
    // @highlight
    smartfeldAktoren.oledWriteNum(h_umgebung)
    // @highlight
    smartfeldAktoren.oledWriteStr("-")
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
     // @highlight
    h_mitLED= smartfeldSensoren.getHalfWord_Visible()
    // @highlight
    smartfeldAktoren.oledWriteNum(h_mitLED)
    basic.pause(200)
})
```

## 🔍 Messfunktion (messeMax) für Mehrfachmessung erstellen

Nach diesen Fragestellungen hast du wohlmöglich bemerkt, dass
  * die Einzelmessungen auch unter gleichen Bedingungen 
  relativ stark variieren, ca. +/- 40 Lumen
  * Erhöht man das Fremdlicht, erhöht sich der Hell- sowie der Dunkelwert in etwa gleich

Gründe für die Unterschiede: Künstliches Licht (z. B. LEDs oder Neonröhren) 
kann stark Flackern, auch wenn wir dies nicht wahrnehmen.

Damit die Messung auch bei flackerndem Licht konstanter wird, 
hilft folgender Trick:
* Helligkeit mehrmals messen (z. B. 10-mal) und höchsten Wert bestimmen.

Du baust jetzt diese Messfunktion nach, welche Dir im Tooltip
(die 💡Glühbirne links unten) angezeigt wird.

* ``||functions:Erstelle eine Funktion...||`` (im Bereich Fortgeschritten zu finden)
    * Benenne die Funktion mit **messeMax** und klicke auf **Fertig**.
    * ``||variables:Erstelle eine Variable...||`` und benenne sie mit **ANZAHL_MESSUNGEN**. 
    Initialisiere die Variable mit 10. 
    * ``||variables:Erstelle eine Variable...||`` und benenne sie mit **maximum**.
    Setze das Maximum vorerst auf 0. 
    * ``||loops:für Index von 0 bis 4||`` 
      * ersetze die 4 mit **ANZAHL_MESSUNGEN**.
    * Bestimme mit dem Block ``||math:maximal||`` den grösseren der beiden 
    Werte **maximum** oder dem Sensorwert 
    ``||SmartfeldSensoren:gib sichtbares Licht[lm] ||``
    und überschreibe damit die Variable **maximum**. Schau Dir den Tooltip 💡
    dafür an.
    * gib am Schluss das gerundete **maximum** zurück mithilfe den Blöcken ``||functions:0 zurückgeben||`` und ``||math:runden||``

```blocks
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
    }
    return Math.round(maximum)
}
```

## 🔍 Mehrfachmessung (Funktion messeMax) einsetzen und testen

* Ersetze in der ``dauerhaft`` - Schleife die zwei Blöcke ``||SmartfeldSensoren:gib sichtbares Licht [lm]||``
  durch je einen Funktionsaufruf ``||functions:Aufruf messeMax||``
* 📥 Drücke `|Download|` und beobachte die Werte auf dem Display🖥️. Beantworte
für dich folgende Fragen:
  * Sind die Messwerte gegenüber vorher konstanter?
  * Wie stark variieren die Werte noch bei gleichen Bedingungen?
  * Könnte man das Umgebungslicht mathematisch herausfiltern mithilfe der
  Umgebungsmessung?

```blocks
// @hide
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

let h_mitLED = 0
let h_umgebung = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    // @highlight
    h_umgebung = messeMax()
    smartfeldAktoren.oledWriteNum(h_umgebung)
    smartfeldAktoren.oledWriteStr("-")
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    // @highlight
    h_mitLED = messeMax()
    smartfeldAktoren.oledWriteNum(h_mitLED)
    basic.pause(200)
})
```

## Änderung der Lichtstärke aufgrund der LED

Du hast nach Deinen Beobachtungen und Überlegungen möglicherweise erkannt, dass die 
Messungen durch die Mehrfachmessung stabiler geworden sind 
(nur noch ca. +/- 10 Lumen Unterschiede).
Es müsste zudem möglich sein, das Umgebungslicht (h_umgebung) vom zweiten 
Messwert (h_mitLED) abzuziehen, um das Umgebungslicht mathematisch zu eliminieren.
Versuchen wir es!

* ``||variables:Erstelle eine variable...||`` mit dem Namen h_unterschied
* Nimm den Block ``||math:0 - 0||`` und ziehe das Umgebungslicht (h_umgebung) 
vom zweiten Messwert (h_mitLED) ab.
* Stelle auf dem Display🖥️ nur noch den Unterschied dar. Entferne die nicht 
mehr benötigten Display- Ausgaben.
* Die erste Pause kannst du ebenfalls entfernen, die zweite Pause ist immer noch sinnvoll, 
damit du die Werte besser ablesen kannst.
* 📥 Drücke `|Download|` und beobachte die Werte auf dem Display🖥️.
  * Stelle eine Duplo- Figur zwischen LED und Lichtsensor.
  * Beobachte, wie sich der Helligkeitsunterschied dabei verändert.

```blocks
// @hide
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
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    h_umgebung = messeMax()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    h_mitLED = messeMax()
    // @highlight
    h_unterschied = h_mitLED - h_umgebung
    // @highlight
    smartfeldAktoren.oledWriteNum(h_unterschied)
    basic.pause(200)
})
```

## 👥 Figur erkennen und zählen

* ``||variables:Erstelle eine variable...||`` mit dem Namen **anzahlPersonenInWarteschlange** und
setze sie zu beginn der ``dauerhaft``- Schleife auf 0. 
* Prüfe am Schluss der ``dauerhaft``- Schleife, ob der Helligkeitsunterschied 
(h_unterschied) kleiner als 100 ist (du kannst diesen Wert auch anpassen,
falls nötig), dann soll die Anzahl **anzahlPersonenInWarteschlange** hochgezählt werden.
Nutze dazu ``||logic:wenn wahr dann||`` sowie ``||logic:0 < 0||`` und 
``||variables:ändere anzahlPersonenInWarteschlange um 1||``
* Zeige **anzahlPersonenInWarteschlange** mit ``||basic:zeige Zahl ||`` auf dem Micro:Bit an.
* 📥 Drücke `|Download|` und kontrolliere die  LED-Anzeige🟥. 
  * Wird die Person an der ersten Stelle korrekt gezählt?
  * Hast Du schon Ideen, wie man an allen neun Positionen Personen 
  zählen könnte? <br />
⬛⬛🟥⬛⬛  
⬛🟥🟥⬛⬛  
⬛⬛🟥⬛⬛  
⬛⬛🟥⬛⬛  
⬛🟥🟥🟥⬛  

```blocks
// @hide
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
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    h_umgebung = messeMax()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    h_mitLED = messeMax()
    h_unterschied = h_mitLED - h_umgebung
    smartfeldAktoren.oledWriteNum(h_unterschied)
    // @highlight
    if (h_unterschied < 100) {
        anzahlPersonenInWarteschlange += 1
    }
    // @highlight
    basic.showNumber(anzahlPersonenInWarteschlange)
    basic.pause(200)
})

```

## Idee entwickeln für das Messen an verschiedenen Positionen

Wenn Du dir im Tooltip (``|💡|``Glühbirne links unten) den bisherigen Code in der 
``dauerhaft`` Schleife ansiehst, könnte man auf die Idee kommen, den gelb 
markierten Teil neunfach (einmal für jedes Loch im Modell) auszuführen, jedoch 
mit einer kleinen Änderung: Die LED- Nummer muss bei jeder Ausführung um eins 
erhöht werden. Natürlich könnte man dies durch stupides neunfaches kopieren 
erreichen, ein Programmierer würde hier aber **eine Schleife drum herum** bauen.

* Sobald Du die grundlegende Idee verstanden hast, drücke auf ``|weiter|``

```blocks
// @hide
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
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()

    // @highlight
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    // @highlight
    strip.show()
    // @highlight
    h_umgebung = messeMax()
    // @highlight
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    // @highlight
    strip.show()
    // @highlight
    h_mitLED = messeMax()
    // @highlight
    h_unterschied = h_mitLED - h_umgebung
    // @highlight
    smartfeldAktoren.oledWriteNum(h_unterschied)
    // @highlight
    if (h_unterschied < 100) {
        anzahlPersonenInWarteschlange += 1
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
    basic.pause(200)
})

```
## An mehreren Positionen Personen erkennen und zählen

* Bewerkstellige mit dem Block ``||loops:für Index von 0 bis 4|`` eine neunfache
Ausführung der vorhergehend erwähnten Blöcken. Du musst dafür die Zahl von 4 auf 8
ändern.
* Ersetze die LED- Nummer (im Moment = 2) durch ``||math:0+0||``, wobei du 0 + 0
wiederum änderst zu der Addition ``||variables:Index||``+ 2. Dies ist nötig, 
damit zuerst die LED mit Index 2 brennt.
* Schalte nach jedem Durchlauf (zuunterst in der Schleife), alle LEDs aus mit dem
Block  ``||neopixel:strip ausschalten||``.
* Entferne alle Blöcke ``||basic:pausiere (ms)||`` aus dem Code, diese sind nicht 
mehr nötig.
* Bewerkstellige einen Zeilenumbruch mit dem Block
``||SmartfeldAktoren:Zeilenumbruch||`` direkt nach der Ausgabe 
des Helligkeitsunterschieds auf dem OLED-Display🖥️.
* 📥 Drücke `|Download|` und teste, ob die Personen beim Platzieren von z.B.
drei Duplo- Figuren🦹‍♂️ korrekt gezählt werden.<br />
🟥🟥🟥🟥🟥  
⬛⬛⬛⬛🟥  
🟥🟥🟥🟥🟥  
⬛⬛⬛⬛🟥  
🟥🟥🟥🟥🟥  

```blocks
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
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    //@highlight
    for (let Index = 0; Index <= 8; Index++) {
        //@highlight
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        //@highlight
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        smartfeldAktoren.oledWriteNum(h_unterschied)
        //@highlight
        smartfeldAktoren.oledNewLine()
        if (h_unterschied < 100) {
            anzahlPersonenInWarteschlange += 1
        }
        //@highlight
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})
```

## Schönere Ausgabe auf dem OLED- Display🖥️

Die Ausgabe auf dem OLED- Display könnten wir noch etwas zweckmässiger gestalten.
Für die Fehlersuche könnte es hilfreich sein, die LED- Position, den Messwert, 
sowie die Auswertung (Person **X** / keine Person **-**) anzuzeigen. Beispiel-
Ausgabe auf dem Display:

    P0: X :120
    P1: - :5
    P2: - :2
    usw.

* ``||functions:Erstelle eine Funktion...||`` (im Bereich Fortgeschritten zu finden)
    * Füge drei Parameter hinzu: Zahl, Zahl und Text
    * Benenne die Funktion mit **schreibeInfosAufDisplay** 
    * Benenne **num** mit **position**
    * Benenne **num2** mit **wert**
    * Benenne **Text** mit **symbol** und klicke auf **Fertig**.
*  ``||variables:Erstelle eine Variable...||`` und benenne sie mit **zeile**. 
Darin wollen wir den Text einer Zeile (z.B. P0: X :120) abspeichern.
* Nimm nun den Block ``||variables:setze zeile auf 0||`` und platziere ihn in die 
erstellte Funktion.
* Klappe ``||Fortgeschritten||`` auf. Unter ``Text`` findest Du den Block ``||text:verbinde "Hallo" "Welt" - +||``.
Weise diesen Block der Variable **zeile** zu (anstelle der **0**). 
* Verbinde nun den Text **"P"**, die Variable **position**, einen **": "**, die Variable **symbol**, ein **" :"**
sowie die Variable **wert**. 
* Danach zeigst Du die Variable ``||variables:zeile||`` mithilfe des Blocks 
``||SmartfeldAktoren:Schreibe String und Zeilenumbruch||`` auf dem OLED-Display🖥️ an.
* setze die Funktion ``||functions:schreibeInfosAufDisplay||`` nun in der ``dauerhaft`` Schleife an den 
korrekten zwei Stellen ein, so wie im Tooltip (💡Links unten) angezeigt.
* Bisherige OLED-Display- Ausgaben kannst du jetzt entfernen.
* 📥 Drücke `|Download|` und teste, ob die OLED- Display🖥️- Ausgaben beim 
Platzieren von z.B. drei Duplo- Figuren🦹‍♂️ korrekt angegeben werden 
(Deine Ausgabe kann natürlich variieren):<br />

    P0: X :120  
    P1: - :5  
    P2: - :2  
    P3: X :204  
    P4: - :4  
    P5: - :2  
    P6: X :140  
    P7: - :2  

* Falls Du eine andere Idee hast für die Visualisierung auf dem OLED- Display, 
fühle Dich frei, etwas anderes zu programmieren!

```blocks
//@highlight
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
            //@highlight
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            //@highlight
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})

```

## Gratuliere 🏆 - du hast den Teil 1 erfolgreich bearbeitet 🚀

* Weiter gehts mit Teil 2: [Teil 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/warteschlange-sensorik-part-2)
* Falls irgendwas noch nicht richtig läuft, hier hast Du eine funktionierende Version zum testen: [Lösung Teil 1](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/warteschlange-sensorik-part-1-solution)