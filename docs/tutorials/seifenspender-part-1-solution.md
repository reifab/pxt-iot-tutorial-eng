```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Soap Dispenser Part 1
## Solution

* Below is the solution for Tutorial Part 1.
* Press 📥 `|Download|` and test the program.

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
