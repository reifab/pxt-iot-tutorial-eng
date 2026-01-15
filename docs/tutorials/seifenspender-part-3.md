```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# IoT Tutorial Part 3

## 📗 Introduction, Part 3

Prerequisites: 🌱 IoT Basics completed and IoT Tutorial [Part 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-2) completed.
Difficulty: 🔥🔥🔥⚪

From Tutorial Part 2 you already have a program that simulates the soap level with buttons A and B and sends it to the internet via LoRa 🛜. To get closer to the real application, you replace buttons A and B with an ultrasonic sensor that measures the soap level.

At the end you will have a program that...

* Measures the soap level with an ultrasonic sensor 🦇.
* Establishes a LoRa connection 🛜.
* Sends the soap level 🧼 over LoRa 🛜.
* Shows a loading bar animation ⏳ for waiting times.

## 👁️ Prerequisites @showdialog

Compared to Tutorial Part 2, you need the following:
* Connect the ultrasonic sensor 🦇 to J1.
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/14_Tutorial_Seifenspender_Ultraschallsensor.png)

## Check the previous functionality

Check that the data still arrives in Clavis Cloud ☁️.
* Upload the existing program to the IoT Cube.
* Go to [🌍iot.claviscloud.ch](https://iot.claviscloud.ch/home).
* Navigate to your dashboard.

Is the 🧼 soap level (100%) shown on the dashboard? Does the soap level drop when you press button A?

## Remove code that is no longer needed

Buttons A and B will be replaced by a real measurement using the ultrasonic sensor. Which code is no longer needed?
* Click the 💡 bulb to see what the code looks like without the buttons.
The sending of the soap level and the waiting function are kept.
* Adjust your code according to the example (💡 bulb).

```blocks
// @highlight
basic.forever(function () {  
    led.plotBarGraph(soapLevelPercent,100)
    IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
    IoTCube.SendBufferSimple()
    wait5SecondsAndShowProgress()
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Temporarily prevent sending

For now we do not want to send anything to the ☁️ cloud.
* Modify **forever** as shown under 💡.
Use the block ``||logic:if true then||``.
This prevents sending as soon as you set the value to **false**.
* In **on start** also use the block ``||logic:if true then||`` to prevent sending data (set the block to **false**).

```blocks
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
// @highlight
if (false) {
    initializeLoRaConnection()
    IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
    IoTCube.SendBufferSimple()
    wait5SecondsAndShowProgress()
}

basic.forever(function () {
    led.plotBarGraph(soapLevelPercent,100)
    // @highlight
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}

// @hide
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
```

## Use the ultrasonic sensor

* Connect the ultrasonic sensor 🦇 to J1.
* To store the measured distance 📏 between soap and sensor, we need a variable.
``||variables:Make a Variable...||`` and name it **distanceSensorToSoap**.
* Drag the block ``||variables:set distanceSensorToSoap to 0||`` to the top of the forever loop.
* To assign the measured value to the variable, insert the block ``||SmartfeldSensoren:distance in cm||`` instead of the 0. Leave the pin on P0.

```blocks
basic.forever(function () {
    // @highlight
    distanceSensorToSoap = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    led.plotBarGraph(
    soapLevelPercent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Show the measured value

The sensor gives the distance in cm. To check that the sensor works, we show the sensor value on the OLED display 🖥️.

* Place the block 🖥️ ``||SmartfeldAktoren:clear display||`` at the top of the forever loop.
This clears existing content on the OLED display 🖥️.
* Under the variable, add the block ``||SmartfeldAktoren:write number||``.
* Replace the 0 with the variable ``||variables:distanceSensorToSoap||``.
* Round the value to whole numbers using ``||math:round||``.
* Write the unit **cm** after the value using ``||SmartfeldAktoren:write string||``.
* Press 📥`|Download|` and check the OLED display 🖥️. Is the distance between the sensor and obstacle shown?

```blocks
basic.forever(function () {
    // @highlight
    smartfeldAktoren.oledClear()
    // @highlight
    distanceSensorToSoap = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    // @highlight
    smartfeldAktoren.oledWriteNum(Math.round(distanceSensorToSoap))
    // @highlight
    smartfeldAktoren.oledWriteStr(" cm")
    led.plotBarGraph(
    soapLevelPercent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Build a soap dispenser model

To test the application, we build a cardboard model that acts as a holder for the ultrasonic sensor 🦇.
A slider lets you change the distance, which simulates the soap level 🧼.

* Download the model and print it on A3.
It is recommended to glue the model to cardboard or thicker paper. This makes it more stable. Template download:
[🌍Cardboard template](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/Seifenspender_Kartonvorlage_Abwicklung_zum_Ausdrucken_A3.pdf)

* The following link shows how the model looks:
 [🌍Cardboard model](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/3dModel/seifenspender3dModellViewer.html)

## Calculate the soap level 🧼

* From the measurement distanceSensorToSoap you can calculate the soap level 🧼 in %. Do you see the relationships when you study the image below?
* Calculate the soap level in percent. Use ``||math:Math||`` and ``||variables:set soapLevelPercent to ...||``.
You may create another variable for intermediate results.
* Round the soap level to whole numbers with ``||math:round||``.
* Soap levels below 0% should be limited to 0%. Use ``||logic:if true then||``.
* Show the soap level 🧼 in % on the OLED display 🖥️ (instead of the distance).
* Press 📥`|Download|` and check the OLED display 🖥️. Is the soap level shown in %? Move the slider and test the program.
* Use the 💡 bulb to see a possible solution if you do not succeed. If the distance is 0 cm, that means 100%; at 25 cm it means 0%.

<img src="https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/Seifenspender_Distanzen.png" width="800">

```blocks
basic.forever(function () {
    smartfeldAktoren.oledClear()
    distanceSensorToSoap = smartfeldSensoren.measureInCentimetersV2(DigitalPin.P0)
    // @highlight
    intermediateResult = 25 - distanceSensorToSoap
    // @highlight
    intermediateResult = intermediateResult / 25
    // @highlight
    soapLevelPercent = intermediateResult * 100
    // @highlight
    soapLevelPercent = Math.round(soapLevelPercent)
    // @highlight
    if (soapLevelPercent < 0) {
        soapLevelPercent = 0
    }
    // @highlight
    smartfeldAktoren.oledWriteNum(soapLevelPercent)
    // @highlight
    smartfeldAktoren.oledWriteStr(" %")
    led.plotBarGraph(
    soapLevelPercent,
    100
    )
    if (false) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## Send to the cloud ☁️ when the soap level changes

Because we want to send the soap level only when it changes, we need to extend the program with a variable that detects changes.

* To detect changes in the soap level, we need to store the old value. Create a new variable with
``||variables:Make a Variable...||`` and name it **previousSoapLevelPercent**.
* In ``||basic:on start||`` set ``||variables:set previousSoapLevelPercent to -1||`` (or any value that is different from reality at the first run).
* In the existing ``||basic:forever||`` loop there is a ``||logic:if false then||`` block that prevents sending.
Modify it for your purpose: use ``||logic:comparison 0 ≠ 0||`` to check if ``||variables:previousSoapLevelPercent||`` and ``||variables:soapLevelPercent||`` are different.
If yes, send the current value to the cloud (already present) and set ``||variables:set previousSoapLevelPercent to soapLevelPercent||``.
* In **on start**, set the condition back to **true** so the connection setup runs again.

```blocks
let intermediateResult = 0
let distanceSensorToSoap = 0
// @highlight
let previousSoapLevelPercent = -1
smartfeldAktoren.oledInit(128, 64)
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
    // @highlight
    if (previousSoapLevelPercent != soapLevelPercent) {
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
        // @highlight
        previousSoapLevelPercent = soapLevelPercent
    }
    basic.pause(150)
    basic.clearScreen()
})

// @hide
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}

// @hide
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
```
## Test, fix, optimize

* Press 📥`|Download|`.
* Check if changes of the soap level 🧼 are shown in the cloud: [iot.claviscloud.ch](https://iot.claviscloud.ch/dashboards/)
* Fix any errors. Click the 💡 icon to show the full "soap dispenser" code.
* Save energy: Displays (both LED and OLED) use a lot of power. Can you optimize the code so the displays are only active when the soap level changes?

```blocks
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

## Congratulations 🏆 - you have successfully completed the soap dispenser tutorial 🚀

* If something is not working yet, here is a working version to test: [Solution Part 3](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-3-solution)

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
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
initializeLoRaConnection()
IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
IoTCube.SendBufferSimple()
wait5SecondsAndShowProgress()
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        soapLevelPercent += -20
        if (soapLevelPercent < 0) {
            soapLevelPercent = 0
        }
        led.plotBarGraph(
        soapLevelPercent,
        100
        )
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    if (input.buttonIsPressed(Button.B)) {
        soapLevelPercent = 100
        led.plotBarGraph(
        soapLevelPercent,
        100
        )
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        IoTCube.SendBufferSimple()
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
    basic.clearScreen()
})
```
