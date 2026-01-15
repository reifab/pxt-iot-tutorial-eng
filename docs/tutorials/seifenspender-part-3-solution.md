```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Soap Dispenser Part 3
## Solution

* Below is the solution for Tutorial Part 3.
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
let intermediateResult = 0
let distanceSensorToSoap = 0
let previousSoapLevelPercent = -1
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
if (true) {
    initializeLoRaConnection()
    IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
    IoTCube.SendBufferSimple()
    wait5SecondsAndShowProgress()
}
basic.forever(function () {
    smartfeldAktoren.oledClear()
    distanceSensorToSoap = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    intermediateResult = 25 - distanceSensorToSoap
    intermediateResult = intermediateResult / 25
    soapLevelPercent = intermediateResult * 100
    soapLevelPercent = Math.round(soapLevelPercent)
    if (soapLevelPercent < 0) {
        soapLevelPercent = 0
    }
    smartfeldAktoren.oledWriteNum(soapLevelPercent)
    smartfeldAktoren.oledWriteStr(" %")
    led.plotBarGraph(
    soapLevelPercent,
    100
    )
    if (previousSoapLevelPercent != soapLevelPercent) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
        previousSoapLevelPercent = soapLevelPercent
    }
    basic.pause(150)
    basic.clearScreen()
})

```
