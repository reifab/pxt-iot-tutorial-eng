```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Smart Toilet Tutorial with Sensor

## 📗 Introduction

**Prerequisites**

🌱 IoT Basics completed and the Smart Toilet tutorial [Part 2 - with internet connection](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/smart-toilet-part2) completed successfully.

Difficulty: 🔥🔥⚪⚪

## 👁️ Prerequisites @showdialog
* Connect the IoT Cube like this if you have not done so yet:
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-anschliessen-klein.png)
* Set the switches for now:
    * Battery switch: **off**
    * LoRa module: **on**
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-power-switches-klein.png)
* A LoRa gateway 🛜 must be in range and connected to TTN (The Things Network).
  Each class set has at least one gateway that can serve hundreds of IoT Cubes.
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/gateway-klein.png)

## Learning outcome

From the previous tutorials you already know how to send the toilet 🚽 status ("Occupied" or "Free") to the internet via LoRa 🛜.
So far you triggered the status by button presses. In a real application a sensor does this - and that is exactly what this tutorial is about.

* You build a toilet cabin model with a magnetic switch.

At the end you will have a program that...

* establishes a LoRa connection 🛜
* detects the toilet 🚽 status using the magnetic switch
* sends the status 🚽 to the internet via LoRa 🛜

Useful functions from [Part 2](https://makecode.microbit.org/#tutorial:github:fave-smartfeld/pxt-smart-toilet-tutorial/docs/tutorials/smart-toilet-part2) are already included; the evaluation of buttons A and B has been removed.

If anything in the existing code is unclear, it is worth going through this part again calmly.

## Prepare the hardware

* You will receive this toilet cabin model:
  <div>
    <img src="https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/wc-haus.png" width="220" alt="Toilet cabin exterior">
    <img src="https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/wc-haus-innen.png" width="220" alt="Toilet cabin interior with sensor">
  </div>

* If the model is already assembled, connect the magnetic switch to the IoT Cube at **J3** and continue with the next step.
* If you do not have the model, you can build something similar yourself:
  * For higher quality you can download the STL file here: [🌍 STL 3D model](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/3dModel/magnet-schalter-halterung.stl), which you can view here: [🌍 Button holder](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/3dModel/schalter-halterung.html)
  * You can print the holder with a 3D printer.
  * Build the sensor system and a magnet 🧲 into your model (similar to the image on the right). Make sure the magnet in the closed state is not exactly centered on the magnetic switch, but slightly offset to the right or left of the elongated black switch part.

## Read the magnetic switch 🧲
The magnetic switch provides a digital signal that we can read easily:

0 → no magnet nearby → door open

1 → magnet 🧲 nearby → door closed

To detect the door state continuously, we read the sensor in the existing ``||basic:forever||`` loop and translate the signal into "open" or "closed".

* ``||variables:Make a Variable...||`` and name it **doorState**.
* At the top of the ``||basic:forever||`` loop set **doorState** to the state measured at pin P2. Use the blocks ``||variables:set doorState to 0||`` and ``||SmartfeldSensoren:detect magnetic field||`` (under ••• Mechanical Sensors) and replace the 0 with the "detect magnetic field" block.
* In the "detect magnetic field" block change P0 to **P2** and make sure the magnetic switch is connected to **J3**.

```blocks
//@hide
function setFree () {
    freeOrOccupiedStatus = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function setOccupied () {
    freeOrOccupiedStatus = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function sendData (status: number) {
    if (control.millis() > msAtLastSend + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        sendLater = false
        msAtLastSend = control.millis()
    } else {
        sendLater = true
    }
}

basic.forever(function () {
    //@highlight
    doorState = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    if (sendLater) {
        sendData(freeOrOccupiedStatus)
    }
})
```

## Logic for sending 🛜

If the door is closed, we assume the toilet is **occupied**. Otherwise we assume the toilet is **free**. We want to display these states and send them to Clavis Cloud via LoRa 🛜, which is already prepared as a function.

* Add an ``||logic:if true then||`` block below the magnetic switch reading.
* Use ``||logic:comparison 0 = 0||`` to check whether ``||variables:doorState||`` is 1 (door closed). Place the comparison in the ``||logic:if true then||`` block instead of **true**.
* If that is the case, call ``||function:call macheBesetzt||``, otherwise ``||function:call macheFrei||``. (The functions are in **Advanced** - open it if needed.)
* Click 📥 `|Download|`.
* Check if the status change (free or occupied) is shown in the cloud:
  [iot.claviscloud.ch](https://iot.claviscloud.ch/dashboards/)
* Fix any errors if needed. Click the 💡 icon if you get stuck.
* Do you notice anything else? Can you optimize anything?


```blocks
//@hide
function setFree () {
    freeOrOccupiedStatus = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function setOccupied () {
    freeOrOccupiedStatus = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function sendData (status: number) {
    if (control.millis() > msAtLastSend + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        sendLater = false
        msAtLastSend = control.millis()
    } else {
        sendLater = true
    }
}

basic.forever(function () {
    doorState = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    //@highlight
    if (doorState == 1) {
        setOccupied()
    } else {
        setFree()
    }
    if (sendLater) {
        sendData(freeOrOccupiedStatus)
    }
})
```

## Optimize 🪫

Right now data is sent every 5 seconds, even if the status has not changed.
That costs unnecessary energy. It makes more sense to send only when the door status changes.

* ``||variables:Make a Variable...||`` and name it **previousDoorState**.
* At the very end of the ``on start`` block set ``||variables:previousDoorState||`` to -1 so it definitely differs from the first measured value: ``||variables:set previousDoorState to -1||``.
* In the ``||basic:forever||`` loop, before **if spaeterSenden then**, check whether ``||variables:doorState||`` and ``||variables:previousDoorState||`` are different (≠ comparison). If yes, update **previousDoorState** and only then run the existing logic. For example:
  * Use ``||logic:if true then||`` and ``||logic:0 ≠ 0||`` (to compare the two variables).
  * Replace the zeros with the variables ``||variables:doorState||`` and ``||variables:previousDoorState||``.
  * If the condition is met, ``||variables:set previousDoorState to doorState||``.
  * Now move your previous check (door = 1) into this new if block.
  Select the if block to move, press ctrl+X to cut and ctrl+V to paste.
  This makes the display and sending happen only when the door status changes.

```blocks
//@hide
function setFree () {
    freeOrOccupiedStatus = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function setOccupied () {
    freeOrOccupiedStatus = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendData(freeOrOccupiedStatus)
}
//@hide
function sendData (status: number) {
    if (control.millis() > msAtLastSend + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        sendLater = false
        msAtLastSend = control.millis()
    } else {
        sendLater = true
    }
}
let freeOrOccupiedStatus = 0
let sendLater = false
let msAtLastSend = 0
IoTCube.LoRa_Join(
eBool.enable,
eBool.enable,
10,
8
)

while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
    basic.showLeds(`
        # . # . #
        # . # . #
        # # # # #
        . . # . .
        . . # . .
        `)
    basic.pause(1000)
}
basic.showIcon(IconNames.Yes)
basic.pause(5000)
msAtLastSend = control.millis()
sendLater = false
let doorState = 0
setFree()
//@highlight
let previousDoorState = -1
basic.forever(function () {
    doorState = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    //@highlight
    if (doorState != previousDoorState) {
        previousDoorState = doorState
        if (doorState == 1) {
            setOccupied()
        } else {
            setFree()
        }
    }
    if (sendLater) {
        sendData(freeOrOccupiedStatus)
    }
})
```

## Congratulations 🏆 - you have successfully completed the tutorial 🚀

* If you have not done it yet, connect your smart toilet to the toilet widget in [Clavis Cloud](https://iot.claviscloud.ch/).
* Test whether the data is shown correctly on the LED display and in the cloud ☁️.
* If something is not working yet, you can find a working version to test here:
[Solution Part 3](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/smart-toilet-part-3-solution)

```template
function setFree () {
    freeOrOccupiedStatus = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendData(freeOrOccupiedStatus)
}
function setOccupied () {
    freeOrOccupiedStatus = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendData(freeOrOccupiedStatus)
}
function sendData (status: number) {
    if (control.millis() > msAtLastSend + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        sendLater = false
        msAtLastSend = control.millis()
    } else {
        sendLater = true
    }
}
let freeOrOccupiedStatus = 0
let sendLater = false
let msAtLastSend = 0
IoTCube.LoRa_Join(
eBool.enable,
eBool.enable,
10,
8
)
while (!(IoTCube.getStatus(eSTATUS_MASK.JOINED))) {
    basic.showLeds(`
        # . # . #
        # . # . #
        # # # # #
        . . # . .
        . . # . .
        `)
    basic.pause(1000)
}
basic.showIcon(IconNames.Yes)
basic.pause(5000)
msAtLastSend = control.millis()
sendLater = false

setFree()

basic.forever(function () {
    if (sendLater) {
        sendData(freeOrOccupiedStatus)
    }
})

```
