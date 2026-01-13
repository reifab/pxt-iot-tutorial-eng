```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# IoT Tutorial Teil 1


## 📗 Einführung,  Teil 1

**Voraussetzungen**
* Micro:Bit Basics: 
    * Du kannst Programme erstellen und herunterladen.
    * Du kennst die Einstiegspunkte "Beim Start" und "Dauerhaft".
    * Dir ist klar, dass Programme in der Regel schrittweise (von oben nach unten) abgearbeitet werden. Zudem kannst Du Schleifen und Verzweigungen einsetzen.
    * Es ist bekannt, dass Kategorien einzelne Blöcke (z.B. ``||basic:Grundlagen||``) beinhalten, welche in Programmen genutzt werden können.
    * Variablen können erstellt, verwendet und verändert werden

**Lernergebnis**

In diesem Tutorial baust du Schritt für Schritt ein Programm auf, 
das einen Seifenstand simuliert und über 🛜 LoRa ins Internet sendet. Am Ende hast 
du ein funktionsfähiges Programm, das...

* den Seifenstand 🧼 anzeigt.
* per ``||Input:Knopfdruck||`` den Seifenstand reduziert oder wieder auffüllt:
    * ``||Input:Knopf A ist geklickt||``: Seifenstand 🧼 wird durch Knopf A um 20% reduziert.
    * ``||Input:Knopf B ist geklickt||``: Seifenstand 🧼 wird durch Knopf B wieder auf 100% aufgefüllt.

**Schwierigkeitsgrad:** 🔥🔥⚪⚪

Klicke auf das 💡- Symbol, falls Du zusätzliche Hilfe brauchst und um deinen Code zu überprüfen.

```blocks
//Super! Du hast den Hinweis gefunden. Nutze ihn, wenn du nicht weiterkommst.
let hinweisGefunden = true;
```

## 👁️ Voraussetzungen @showdialog
* Für Teil 1 brauchst Du grundsätzlich nur einen Micro:Bit. 
* Falls du lieber gleich den IoT- Cube nehmen möchtest, kannst du ihn so anschliessen. Achte auf
die rote Markierung:
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-anschliessen-klein.png)
* Stelle die Schalter vorerst so ein:
    * Battery Switch: **off**
    * LoRa Module: **on**
![Bild](https://reifab.github.io/pxt-iot-tutorial/static/tutorials/iot-cube-power-switches-klein.png)
* Überprüfe, ob der micro:bit verbunden ist.

## 🧼 Variable für den Seifenstand
Um den Seifenstand des Seifenspenders zu speichern, nutzen wir eine Variable.
* Um den aktuellen Seifenstand zu speichern, benötigen wir eine Variable, die den Seifenstand in Prozent anzeigt: 
``||variables:Erstelle eine Variable...||`` und benenne sie mit **seifenstandInProzent** 🧼.
* Der Seifenspender ist am Beginn vollständig gefüllt. Setze deshalb ``||basic:beim Start||`` den Seifenstand auf 100 %. Nutze dazu die zuvor angelegte Variable: ``||variables:setze seifenstandInProzent auf 100||``🧼



```blocks
let seifenstandInProzent = 100
```

## 🧼 Seifenstand anzeigen
Ziel ist es, den aktuellen Seifenstand am IoT Cube anzuzeigen.
* Hol dir den Block ``||led:Zeichne Säulendiagramm||``🟥 und ziehe diesen in den Block **beim Start** direkt unter die Variable **seifenstandInProzent**🧼
* Setze die Variable ``||variables:seifenstandInProzent||``🧼 in das erste Feld des Befehls **zeichne Säulendiagramm von**. 
* Ändere den Bereich von **seifenstandInProzent**🧼 bis 100. 
* 📥 Drücke `|Download|` und kontrolliere die LED-Anzeige:  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
Leuchten alle LEDs?

```blocks
let seifenstandInProzent = 100
// @highlight
led.plotBarGraph(
seifenstandInProzent,
100
)
```

## ➖ Seifenstand reduzieren mit Knopf A
Ziel ist es bei jedem Knopfdruck auf A den Seifenstand jeweils um 20% zu reduzieren.
Dazu benötigen wir eine Verzweigung, die prüft, ob Knopf A gedrückt wurde. Wenn dies der Fall ist, 
dann soll der Seifenstand um 20% reduziert werden.
* Um diese Verzweigung einzufügen, hol dir den Block ``||Logic:wenn wahr dann||`` und 
ziehe ihn in die bestehende ``||basic:dauerhaft||`` Schleife
* Schiebe einen neuen Block ``||Input:Knopf A ist geklickt||`` auf das Feld ``wahr``
* Ändere die Variable ``||variables:seifenstandInProzent||`` 🧼 um -20.
* Zeichne erneut das Säulendiagramm.🟥 Dupliziere diesen Teil aus ``beim Start``
* Verzögere die bestehende ``||basic:dauerhaft||`` Schleife zum Schluss nach dem bestehenden 
``||Logic:wenn wahr dann||`` Block um 150 ms mit ``||basic:pausiere (ms)||``.

```blocks
basic.forever(function () {
    // @highlight
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    // @highlight
    basic.pause(150)
})
```

## 🧼 Füllstand kleiner 0 verhindern
Um zu vermeiden, dass der Füllstand unter 0% fällt, benötigen wir eine weitere Bedingung, 
die prüft, ob der Seifenstand unter 0% gefallen ist. Wenn dies der Fall ist, soll der
Seifenstand auf 0% gesetzt werden.
* Ergänze unter dem Block ``||variables:ändere seifenstandInProzent um -20||`` 
einen weiteren Block ``||Logic:wenn wahr dann||`` und überprüfe, ob der 
``||seifenstandInProzent < 0||`` ist. Wenn ja, dann setze den 
``||variables:seifenstandInProzent auf 0||``.
* Setze den ``||variables:seifenstandInProzent||`` auf 0% in dem du den Seifenstand 🧼 
auf 0 setzt.
[Hier findest du weitere Informationen zu logischen Operatoren](https://makecode.microbit.org/blocks/logic/boolean)
* 📥 Drücke `|Download|` und kontrolliere die 🟥 LED-Anzeige. Drücke öfters Knopf A, bis der Seifenstand unter 0% fällt. 

⬛⬛🟥⬛⬛  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
🟥🟥🟥🟥🟥  
Was passiert? Bleibt die LED-Anzeige bei 0 stehen?  

```blocks
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        // @highlight
        if(seifenstandInProzent<0){
            // @highlight
            seifenstandInProzent = 0
        }
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    basic.pause(150)
})
```

## ➕ Seifenspender auffüllen mit Knopf B
Nun wollen wir den Seifenstand 🧼 wieder auffüllen, wenn Knopf B gedrückt wird.
Dazu benötigen wir eine Bedingung, die prüft, ob Knopf B gedrückt wurde. Wenn dies der Fall ist, soll der Seifenstand 🧼 auf 100% gesetzt werden.
* Hol dir den Block ``||Logic:wenn wahr dann||`` und ziehe ihn in
die bestehende ``||basic:dauerhaft||`` Schleife, oberhalb von ``||basic:pausiere (ms)||``
* Schiebe den Block ``||Input:Knopf A ist geklickt||`` auf das Feld ``wahr``
und ändere Knopf A zu Knopf **B**
* Setze den Seifenstand auf 100% indem du die Variable ``||variables:seifenstandInProzent||``🧼 auf 100 setzt.
* Zeichne erneut das Säulendiagramm. Kopiere diesen Teil aus ``beim Start``
* 📥 Drücke `|Download|` und kontrolliere die 🟥 LED-Anzeige... 
Füllt sich der Seifenstand auf 100% auf?

```blocks
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        
        if(seifenstandInProzent<0){
           
            seifenstandInProzent = 0
        }
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    // @highlight
    if (input.buttonIsPressed(Button.B)) {
        // @highlight
        seifenstandInProzent = 100
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    basic.pause(150)
   
})
```

## Gratuliere 🏆 - du hast den Teil 1 erfolgreich bearbeitet 🚀

* Weiter gehts mit Teil 2: [Teil 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/seifenspender-part-2)
* Falls irgendwas noch nicht richtig läuft, hier hast Du eine funktionierende Version zum testen: [Lösung Teil 1](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial/docs/tutorials/seifenspender-part-1-solution)


