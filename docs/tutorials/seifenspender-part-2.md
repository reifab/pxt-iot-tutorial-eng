```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# IoT Tutorial Part 2

## 📗 Introduction, Part 2

Prerequisites: 🌱 IoT Basics completed and IoT Tutorial [Part 1 - still without internet connection](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-1) completed.
Difficulty: 🔥🔥⚪⚪

From Tutorial Part 1 you already have a program that simulates the soap level.
Now we want to send the soap level to the internet via LoRa 🛜. At the end you will have a working program that:

* Establishes a LoRa connection 🛜.
* Sends the soap level 🧼 over LoRa 🛜.
* Shows a loading bar animation ⏳ during waiting times.

## 👁️ Prerequisites @showdialog
* You need an IoT Cube with the OLED display 🖥️ connected to J5.
* Connect the cube like this if you have not done so yet:
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-anschliessen-klein.png)
* Set the switches for now:
    * Battery switch: **off**
    * LoRa module: **on**
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-power-switches-klein.png)

* A LoRa gateway 🛜 must be in range and connected to TTN (The Things Network).
In class there is one gateway that can serve hundreds of IoT Cubes.
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/gateway-klein.png)

## 🖥️ Show a status message on the OLED
On the small display of the IoT Cube we want to show the text "Connecting".

* Open the area ``||SmartfeldAktoren:•••Display||`` and get the block 🖥️ ``||SmartfeldAktoren:init OLED width 128 height 64||``.
Add it at the bottom of the ``||basic:on start||`` block.
* Under it, add the block 🖥️ ``||SmartfeldAktoren:clear display||``.
This clears existing content on the display.
* Now use the block ``||SmartfeldAktoren:write string||``.
This writes the text "Connecting" on the display.
* Press 📥`|Download|` and check the OLED display 🖥️.

Do you see "Connecting"?

```blocks
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
// @highlight
smartfeldAktoren.oledInit(128, 64)
// @highlight
smartfeldAktoren.oledClear()
// @highlight
smartfeldAktoren.oledWriteStr("Connecting")
```

## 🛜 Build the internet connection
Now we build a connection to the internet.
On the OLED display 🖥️ we want to show the connection process with **...**.
* Drag the block 🛜``||IoTCube:LoRa network connection||`` to the bottom of
``||basic:on start||``.
* Under it, place the block ``||loops:while false do||``. Because connecting can take 5 to 30 seconds, we want to stay in this loop as long as the connection does **not** exist.
* Put the block ``||logic:not||`` into the loop to negate the condition.
* Now insert 🛜``||IoTCube:read device status bit||`` into the **not** block.
Change the bit to **connected**. The loop now reads "while not read device status bit connected". Programmers read this as: "While the device is not connected."
* Use the block 🖥️ ``||SmartfeldAktoren:write string||`` to write a "." to the display on each loop iteration.
* Wait 1 second (1000 ms) inside the loop using ``||basic:pause (ms)||``.

```blocks
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
smartfeldAktoren.oledClear()
smartfeldAktoren.oledWriteStr("Connecting")
// @highlight
IoTCube.LoRa_Join(
eBool.enable,
eBool.enable,
10,
8
)
// @highlight
while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
    smartfeldAktoren.oledWriteStr(".")
    basic.pause(1000)
}
```

## 🛜 Show "Connected" status
The loop ends when the connection is established, so we can show "Connected" on the display after the loop:

* Under the ``||loops:while||`` loop, clear the display using 🖥️``||SmartfeldAktoren:clear display||``.
* Then write "Connected!" to the OLED display 🖥️.
* ``||basic:pause (ms)||`` for 2 seconds (=2000 ms) after showing "Connected!" so the text stays on the display.
* Then clear the text with ``||SmartfeldAktoren:clear display||`` to save energy.
* Press 📥`|Download|` and check the OLED display 🖥️.

Do you first see "Connecting" and then more and more dots while the connection is built?
Do you see "Connected!" for 2 seconds?

```blocks
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
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
// @highlight
smartfeldAktoren.oledClear()
// @highlight
smartfeldAktoren.oledWriteStr("Connected!")
// @highlight
basic.pause(2000)
// @highlight
smartfeldAktoren.oledClear()
```

## 🛜 Function for connection setup

To keep the program clear, we often use functions. We want to create a function named **initializeLoRaConnection** that contains the connection setup.
* Open **Advanced**, get the block ``||functions:Make a Function...||`` and name it **initializeLoRaConnection**.
* Move all code blocks related to the connection setup into this new function.
Click the 💡 bulb to see which blocks are meant.
* Get the block ``||functions:call initializeLoRaConnection||`` and place it at the bottom of **on start**.

```blocks
// @highlight
function initializeLoRaConnection() {
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

smartfeldAktoren.oledInit(128, 64)
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
initializeLoRaConnection()
```

## 🧼 Send the soap level at start

At the beginning the soap level is 100%.
We want to send this after initializing the LoRa connection.

* Under the function call ``||functions:initializeLoRaConnection||`` add the block ``||IoTCube:add unsigned integer with ID_0 = 0||``.
* Replace the 0 with the variable ``||variables:soapLevelPercent||``.
* Now send the soap level to the ☁️ cloud with ``||IoTCube:send data||``.
* Press 📥`|Download|`.

```blocks
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)
smartfeldAktoren.oledInit(128, 64)
initializeLoRaConnection()
// @highlight
IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
// @highlight
IoTCube.SendBufferSimple()
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

## ☁️ Create a dashboard in Clavis Cloud

Now we visualize the data in Clavis Cloud ☁️.
* Go to [🌍iot.claviscloud.ch](https://iot.claviscloud.ch/home).
* Sign in (your teacher or course lead will give you the login info).
* Watch this [📹 video](https://wiki.smartfeld.ch/lib/exe/fetch.php?media=dashboard_erstellen_seifenspender.mp4). It includes these steps.
Follow them and create a dashboard.
 * Create a new dashboard in your class group.
 * Create a widget to display the soap level 🧼.

## 🧼 Send the soap level when it changes

Whenever the soap level changes, the current value should be sent to the cloud. Do the following:

* When button A is pressed, send the current soap level using the blocks ``||IoTCube:add unsigned integer with ID_0 = 0||`` and ``||IoTCube:send data||``. If you are unsure, click the 💡 bulb in the lower left.
* Do the same when button B is pressed. If you are unsure, click the 💡 bulb in the lower left.
* Do not forget to send the variable (not just a 0).
* Note: You could test your program now, but it is better to finish the next two steps to avoid side effects.

```blocks
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
        // @highlight
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        // @highlight
        IoTCube.SendBufferSimple()
    }
    if (input.buttonIsPressed(Button.B)) {
        soapLevelPercent = 100
        led.plotBarGraph(
        soapLevelPercent,
        100
        )
        // @highlight
        IoTCube.addUnsignedInteger(eIDs.ID_0, soapLevelPercent)
        // @highlight
        IoTCube.SendBufferSimple()
    }
    basic.pause(150)
})

```

## ⏱️ Create a waiting function

After sending data over LoRa, you must wait at least 5 seconds before sending again.
Background: During these 5 seconds there is a receive window for data from the cloud to the Cube.

Recreate the waiting function shown in the tooltip (the 💡 bulb in the lower left) using these blocks:

* ``||function:Make a function||`` (found in Advanced)
    * Name the function "wait5SecondsAndShowProgress" and click **Done**.
    * Clear the display inside with ``||SmartfeldAktoren:clear display||``.
    * ``||loops:for index from 0 to 4||``
        * click on index --> click **New Variable**, new variable name: **progress**
        * replace the 4 with 100.
    * ``||SmartfeldAktoren:plot loading bar at 0 percent||``
        * replace the 0 with the variable **progress**.
    * Wait in the loop with ``||basic:pause (ms)||`` for 50 ms each time
     (100x 50 ms = 5 seconds).
    * Clear the display again after the loop.

```blocks
// @highlight
function wait5SecondsAndShowProgress () {
    smartfeldAktoren.oledClear()
    for (let progress = 0; progress <= 100; progress++) {
        smartfeldAktoren.oledLoadingBar(progress)
        basic.pause(50)
    }
    smartfeldAktoren.oledClear()
}
```

## ⏱️ Add the waiting time

* After each ``||IoTCube:send data||`` command (3x in total) add the Advanced function ``||function:wait5SecondsAndShowProgress||``.
* 📥 Press `|Download|` and check if...
    * the loading bar appears on the OLED display when a button is pressed
    * the data is shown on your dashboard: [iot.claviscloud.ch](https://iot.claviscloud.ch/dashboards/)

```blocks
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
// @hide
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
// @highlight
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
        // @highlight
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
        // @highlight
        wait5SecondsAndShowProgress()
    }
    basic.pause(150)
})
```

## 🪫 Save energy

* To save energy you can clear the soap level display 🧼 in the existing ``||basic:forever||`` loop after each iteration.
Use ``||basic:clear screen||``.
* 📥 Press `|Download|` and test your final program.

```blocks
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
// @hide
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
    // @highlight
    basic.clearScreen()
})

```

## Congratulations 🏆 - you have successfully completed Part 2 🚀

* Continue with Part 3: [Part 3](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-3)
* If something is not working yet, here is a working version to test: [Solution Part 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-2-solution)

```template
let soapLevelPercent = 100
led.plotBarGraph(
soapLevelPercent,
100
)

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
    }
    if (input.buttonIsPressed(Button.B)) {
        soapLevelPercent = 100
        led.plotBarGraph(
        soapLevelPercent,
        100
        )
    }
    basic.pause(150)
})
```
