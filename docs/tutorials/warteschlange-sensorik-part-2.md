```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
neopixel=github:microsoft/pxt-neopixel#v0.7.6
```
### @explicitHints false

# Queue Sensor System Part 2

## 👁️ Prerequisites @showdialog
* Connect the Cube like this if you have not done so yet:
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-anschliessen-klein.png)
* Set the switches for now:
    * Battery switch: **off**
    * LoRa module: **on**
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-power-switches-klein.png)

* A LoRa gateway 🛜 must be in range and connected to TTN (The Things Network).
In class there is one gateway that can serve hundreds of IoT Cubes.
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/gateway-klein.png)

## Learning outcome

From Tutorial Part 1 you already created a program that automatically counts the number of people in a queue.
Now we want to send the number of people in the queue to the internet via LoRa 🛜. At the end you will have a working program that...

* Establishes a LoRa connection 🛜.
* Sends the automatically detected number of people in the queue 👥 over LoRa 🛜.

The full program from Part 1 is already included. You will also find three functions that we only need to use in this tutorial:

* initialisiereLoRaVerbindung(): builds a connection to the internet
* warte5SekundenUndZeigeFortschritt(): wait function so values are not sent too often
* sendeUndZeigePersonenanzahl(): sends the number of people in the queue

If anything about these functions is unclear, consider first doing [Part 2 of the queue tutorial](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/warteschlange-part-2).

## 🛜 Establish the internet connection

At the start we build a connection to the internet. For that we need a function call.

* Get the block ``||functions:call initialisiereLoRaVerbindung||`` and place it at the bottom of **on start**.

* 📥 Press `|Download|` and check the OLED display 🖥️:
Do you see **Connecting...** on the OLED display? <br />
Do you then see **Connected!**? <br />

```blocks
//@hide
function writeInfoToDisplay (position: number, value: number, _symbol: string) {
    line = "P" + position + ": " + _symbol + " :" + value
    smartfeldAktoren.oledWriteStrNewLine(line)
}
//@hide
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
//@hide
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
//@hide
function sendAndShowPeopleCount () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, peopleInQueueCount)
    IoTCube.SendBufferSimple()
    basic.showNumber(peopleInQueueCount)
    wait5SecondsAndShowProgress()
}
//@hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
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
//@highlight
initializeLoRaConnection()
//@hide
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
    basic.showNumber(peopleInQueueCount)
})

```

## 🛜 Send the number of people via LoRa

Until now the number of people is only shown in the 'forever' loop and not yet sent. Let's change that:

* Replace the block ``||basic:showNumber(anzahlPersonenInWarteschlange)||`` with ``||functions:call sendeUndZeigePersonenanzahl||``.
* 📥 Press `|Download|` and check the LED display 🟥 and the OLED display 🖥️:
Test whether the people are correctly counted when you place, for example, three Duplo figures 🦹‍♂️. Right after the number is shown, the loading bar should appear on the OLED display 🖥️ for 5 seconds.<br />
🟥🟥🟥🟥🟥<br>
⬛⬛⬛⬛🟥<br>
🟥🟥🟥🟥🟥<br>
⬛⬛⬛⬛🟥<br>
🟥🟥🟥🟥🟥<br>

```blocks
//@hide
function writeInfoToDisplay (position: number, value: number, _symbol: string) {
    line = "P" + position + ": " + _symbol + " :" + value
    smartfeldAktoren.oledWriteStrNewLine(line)
}
//@hide
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
//@hide
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
//@hide
function sendAndShowPeopleCount () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, peopleInQueueCount)
    IoTCube.SendBufferSimple()
    basic.showNumber(peopleInQueueCount)
    wait5SecondsAndShowProgress()
}
//@hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
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
    //@highlight
    sendAndShowPeopleCount()
})
```

## ☁️ Create a dashboard in Clavis Cloud

Now we visualize the data in Clavis Cloud ☁️.
* Go to [🌍iot.claviscloud.ch](https://iot.claviscloud.ch/home).
* Sign in (your teacher or course lead will give you the login info).
* Expand your dashboard with a queue widget, if it is not there yet.
* In the widget, choose your IoT Cube (as labeled) as the **data source** under **device**.
* Choose **Ganzzahl_ID_0** as the **data key**.
* Click **Save** and test whether the number of people is sent and displayed.

## 🪫 Optimizations

Currently, data is sent via LoRa after every counting cycle, no matter whether the number of waiting people has changed. It would save energy to send data only when there is actually a change in the number of waiting people.

* ``||variables:Make a Variable...||`` and name it **anzahlPersonenVorher**.
* At the bottom of the ``on start`` block set **anzahlPersonenVorher** to -1 (so it definitely differs from the first measured value). You can do this with ``||variables:set anzahlPersonenVorher to -1||``.
* At the end of the ``forever`` loop check whether ``||variables:anzahlPersonenInWarteschlange||`` and ``||variables:anzahlPersonenVorher||`` differ (≠). If they are different, set **anzahlPersonenVorher** to **anzahlPersonenInWarteschlange**.
Use ``||logic:if true then||``, ``||logic:0 ≠ 0||``, and
``||variables:set anzahlPersonenVorher to anzahlPersonenInWarteschlange||``.

```blocks
//@hide
function writeInfoToDisplay (position: number, value: number, _symbol: string) {
    line = "P" + position + ": " + _symbol + " :" + value
    smartfeldAktoren.oledWriteStrNewLine(line)
}
//@hide
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
//@hide
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
//@hide
function sendAndShowPeopleCount () {
    IoTCube.addUnsignedInteger(eIDs.ID_0, peopleInQueueCount)
    IoTCube.SendBufferSimple()
    basic.showNumber(peopleInQueueCount)
    wait5SecondsAndShowProgress()
}
//@hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
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
//@highlight
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
    //@highlight
    if (peopleInQueueCount != previousPeopleInQueueCount) {
        //@highlight
        sendAndShowPeopleCount()
        //@highlight
        previousPeopleInQueueCount = peopleInQueueCount
    }
})
```

## Congratulations 🏆 - you have successfully completed Part 2 🚀

* If something is not working yet, here is a working version to test: [Solution Part 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/warteschlange-sensorik-part-2-solution)


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
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
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
    basic.showNumber(peopleInQueueCount)
})

```
