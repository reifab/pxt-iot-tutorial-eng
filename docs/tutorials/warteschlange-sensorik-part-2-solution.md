```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
neopixel=github:microsoft/pxt-neopixel#v0.7.6
```
### @explicitHints false

# Queue Sensor System Part 2
## Solution

* Below you will find the complete solution.
* Press 📥 `|Download|` and test the program.

```template

function writeInfoToDisplay (position: number, value: number, _symbol: string) {
    line = "P" + position + ": " + _symbol + " :" + value
    smartfeldAktoren.oledWriteStrNewLine(line)
}
function measureMax () {
    numMeasurements = 10
    maximum = 0
    for (let index = 0; index < numMeasurements; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
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
        basic.pause(1000)
        smartfeldAktoren.oledWriteStr(".")
    }
    smartfeldAktoren.oledClear()
    smartfeldAktoren.oledWriteStr("Connected!")
    basic.pause(2000)
    smartfeldAktoren.oledClear()
}
function sendAndShowPeopleCount () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, peopleInQueueCount)
    IoTCube.SendBufferSimple()
    basic.showNumber(peopleInQueueCount)
    wait5SecondsAndShowProgress()
}
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(10)
    }
    smartfeldAktoren.oledClear()
}
let lightDifference = 0
let lightWithLed = 0
let ambientLight = 0
let peopleInQueueCount = 0
let maximum = 0
let numMeasurements = 0
let line = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
initializeLoRaConnection()
let previousPeopleInQueueCount = -1
basic.forever(function () {
    peopleInQueueCount = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        ambientLight = measureMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        lightWithLed = measureMax()
        lightDifference = lightWithLed - ambientLight
        if (lightDifference < 100) {
            writeInfoToDisplay(Index, lightDifference, "X")
            peopleInQueueCount += 1
        } else {
            writeInfoToDisplay(Index, lightDifference, "-")
        }
        strip.clear()
    }
    if (peopleInQueueCount != previousPeopleInQueueCount) {
        sendAndShowPeopleCount()
        previousPeopleInQueueCount = peopleInQueueCount
    }
})

```
