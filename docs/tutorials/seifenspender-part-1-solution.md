```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# Seifenspender Teil 1
## Lösung

* Unten die Lösung von Tutorial Teil 1 
* Drücke 📥 `|Download|` und teste das Programm.

```template
let seifenstandInProzent = 100
led.plotBarGraph(
seifenstandInProzent,
100
)
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
    }
    if (input.buttonIsPressed(Button.B)) {
        seifenstandInProzent = 100
        led.plotBarGraph(
        seifenstandInProzent,
        100
        )
    }
    basic.pause(150)
})
```