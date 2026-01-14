```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Soap Dispenser Part 2
## Solution

* Below is the solution for Tutorial Part 2.
* Press 📥 `|Download|` and test the program.

```template

function initializeLoRaConnection () {
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Connecting")
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
    smartfeldAktoren.oledWriteStr("Connected!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
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
initializeLoRaConnection()
IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
IoTCube.SendBufferSimple()
wait5SecondsAndShowProgress()
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
        wait5SecondsAndShowProgress()
    }
    if (input.buttonIsPressed(Button.B)) {
        seifenstandInProzent = 100
        led.plotBarGraph(
        seifenstandInProzent,
        100
        )
        IoTCube.addUnsignedInteger(eIDs.ID_0, seifenstandInProzent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})

```
