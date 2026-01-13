```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Smart Toilet Teil 3
## Lösung

* Unten die Lösung von Tutorial Teil 3 
* Drücke 📥 `|Download|` und teste das Programm.

```template
function macheFrei () {
    statusFreiOderBesetzt = 1
    basic.showLeds(`
        . . # . .
        . # # # .
        # . # . #
        . . # . .
        . . # . .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
function macheBesetzt () {
    statusFreiOderBesetzt = 0
    basic.showLeds(`
        . . . . #
        . . . . #
        . . . . #
        # # # # #
        . # # # .
        `)
    sendeDaten(statusFreiOderBesetzt)
}
function sendeDaten (status: number) {
    if (control.millis() > msBeiLetztemSenden + 5000) {
        IoTCube.addBinary(eIDs.ID_0, status)
        IoTCube.SendBufferSimple()
        music.play(music.tonePlayable(262, music.beat(BeatFraction.Whole)), music.PlaybackMode.UntilDone)
        spaeterSenden = false
        msBeiLetztemSenden = control.millis()
    } else {
        spaeterSenden = true
    }
}
let statusFreiOderBesetzt = 0
let spaeterSenden = false
let msBeiLetztemSenden = 0
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
msBeiLetztemSenden = control.millis()
spaeterSenden = false
let zustandTür = 0
let zustandTürDavor = -1
macheFrei()
basic.forever(function () {
    zustandTür = smartfeldSensoren.fieldDetected(DigitalPin.P2)
    if (zustandTür != zustandTürDavor) {
        zustandTürDavor = zustandTür
        if (zustandTür == 1) {
            macheBesetzt()
        } else {
            macheFrei()
        }
    }
    if (spaeterSenden) {
        sendeDaten(statusFreiOderBesetzt)
    }
})
```
