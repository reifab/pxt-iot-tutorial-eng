```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Seifenspender Teil 3
## Lösung

* Unten die Lösung von Tutorial Teil 3 
* Drücke 📥 `|Download|` und teste das Programm.

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