```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Seifenspender Teil 2
## Lösung

* Unten die Lösung von Tutorial Teil 2 
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