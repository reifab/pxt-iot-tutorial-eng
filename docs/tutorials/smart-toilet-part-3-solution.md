```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Smart Toilet Part 3
## Solution

* Below is the solution for Tutorial Part 3.
* Press 📥 `|Download|` and test the program.

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
let doorState = 0
let previousDoorState = -1
setFree()
basic.forever(function () {
    doorState = smartfeldSensoren.fieldDetected(DigitalPin.P2)
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
