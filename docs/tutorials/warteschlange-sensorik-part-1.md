```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
neopixel=github:microsoft/pxt-neopixel#v0.7.6
```
### @explicitHints false

# Queue Sensor System Part 1

## 🎬 Measurement principle

We light up specific positions one after another with an LED 💡 and measure with the light sensor 👁️ how much light arrives.
If something is between the sensor 👁️ and the LED 💡, for example a LEGO figure 🦹‍♂️, the brightness decreases.
This tells us if something is standing in the light beam at a certain position.
We take measurements at nine positions in the queue one after another so the LEGO figures 🦹‍♂️ can be counted.

Watch this video that illustrates the measurement principle:
* [Watch video 🎬: Queue sensor system](https://wiki.smartfeld.ch/lib/exe/fetch.php?media=warteschlange_sensorik.mp4)

## 👁️ Hardware prerequisites @showdialog
* You need this plastic part: [Waiting area as plastic part](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/3dModel/warteschlange3dViewer.html)
* If you want to 3D print the part yourself, download the STL file here: [🌍STL 3D model](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/3dModel/Wartebereich.stl)
* Connect the RGB LED strip 💡 to **J7**.
* Connect the light sensor 👁️ to **J0** on the IoT Cube.
* Connect the OLED display 🖥️ to **J5**.

## 💡 Power up the LEDs

At the beginning we want the first LED (in the 3D model it shines through a hole) to blink. On the 16 LED strip this is LED index **2**.

* In ``||basic:on start||`` initialize the LED strip with ``||neopixel:set strip to NeoPixels at pin||`` **P1** with **16** pixels and RGB mode.
* Under **...more** you can find ``||neopixel:set brightness||`` and set it to 255.
* In the ``forever`` loop use ``||neopixel:strip set pixel color||`` **2** to **black** (under **...more**).
* Under it use ``||neopixel:strip show||``.
* ``||basic:pause (ms)||`` after showing for 100 ms.
* Then use ``||neopixel:strip set pixel color||`` **2** to **white** (under **...more**).
* Under it use ``||neopixel:strip show||``.
* ``||basic:pause (ms)||`` after showing for 200 ms.
* 📥 Press `|Download|` and check if LED 2 (first LED in the queue) blinks.

```blocks
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    basic.pause(200)
})
```

## 🔍 Measure brightness and show it on the OLED display

Now we measure the brightness with the sunlight sensor 👁️ twice: once with LED lighting and once without (ambient light only), and show the values on the display.

* In ``||basic:on start||`` initialize the display with 🖥️ ``||SmartfeldAktoren:init OLED width 128 height 64||``.
* In ``||basic:on start||`` initialize the sunlight sensor with ``||SmartfeldSensoren:init sunlight sensor||``.
* Place the block 🖥️ ``||SmartfeldAktoren:clear display||`` at the top of the forever loop.
* ``||variables:Make a Variable...||`` and name it **h_umgebung**.
* In the ``forever`` loop use ``||variables:set lichtmessung to 0||``.
* Replace 0 with a measurement ``||SmartfeldSensoren:visible light [lm]||`` and perform this measurement directly after the first **strip show**.
* Under the measurement add ``||SmartfeldAktoren:write number||``.
* Replace the 0 with the variable ``||variables:h_umgebung||``.
* Add ``||SmartfeldAktoren:write string "-"||`` to show a dash as a separator on the OLED display 🖥️.
* ``||variables:Make a Variable...||`` and name it **h_mitLED**.
* Repeat the measurement and display using **h_mitLED**. You can insert this measurement after the second **strip show**.
* 📥 Press `|Download|` and observe the values on the OLED display 🖥️. Answer these questions for yourself:
  * How big is the difference between the values (ambient light - light with LED)?
  * How strongly can ambient light influence the values?
  * How much do the values vary with seemingly constant ambient light?

```blocks
let h_umgebung = 0
let h_mitLED = 0
// @highlight
smartfeldSensoren.initSunlight()
// @highlight
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    // @highlight
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    // @highlight
    h_umgebung = smartfeldSensoren.getHalfWord_Visible()
    // @highlight
    smartfeldAktoren.oledWriteNum(h_umgebung)
    // @highlight
    smartfeldAktoren.oledWriteStr("-")
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
     // @highlight
    h_mitLED= smartfeldSensoren.getHalfWord_Visible()
    // @highlight
    smartfeldAktoren.oledWriteNum(h_mitLED)
    basic.pause(200)
})
```

## 🔍 Create a measurement function (messeMax) for repeated measurements

After these observations you likely noticed that:
  * single measurements vary quite a lot even under the same conditions (around +/- 40 lumens)
  * if you increase ambient light, both the bright and dark values increase by a similar amount

Reason for the differences: Artificial light (for example LEDs or fluorescent tubes) can flicker strongly even if we do not notice it.

To make the measurement more stable even with flickering light, this trick helps:
* Measure brightness multiple times (for example 10 times) and take the highest value.

Now recreate this measurement function shown in the tooltip (the 💡 bulb in the lower left).

* ``||functions:Make a Function...||`` (found in Advanced)
    * Name the function **messeMax** and click **Done**.
    * ``||variables:Make a Variable...||`` and name it **ANZAHL_MESSUNGEN**.
    Initialize the variable with 10.
    * ``||variables:Make a Variable...||`` and name it **maximum**.
    Set the maximum to 0 for now.
    * ``||loops:for index from 0 to 4||``
      * replace the 4 with **ANZAHL_MESSUNGEN**.
    * Use ``||math:max||`` to select the larger of **maximum** and the sensor value
    ``||SmartfeldSensoren:visible light [lm]||``
    and overwrite **maximum**. Check the tooltip 💡 for this.
    * Finally return the rounded **maximum** using ``||functions:return 0||`` and ``||math:round||``.

```blocks
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
    }
    return Math.round(maximum)
}
```

## 🔍 Use and test repeated measurements (function messeMax)

* Replace the two blocks ``||SmartfeldSensoren:visible light [lm]||`` in the ``forever`` loop
  with the function call ``||functions:call messeMax||``.
* 📥 Press `|Download|` and observe the values on the OLED display 🖥️. Answer these questions for yourself:
  * Are the measurements more stable than before?
  * How much do the values still vary under the same conditions?
  * Could you mathematically filter out ambient light using the ambient measurement?

```blocks
// @hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}

let h_mitLED = 0
let h_umgebung = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    // @highlight
    h_umgebung = messeMax()
    smartfeldAktoren.oledWriteNum(h_umgebung)
    smartfeldAktoren.oledWriteStr("-")
    basic.pause(200)
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    // @highlight
    h_mitLED = messeMax()
    smartfeldAktoren.oledWriteNum(h_mitLED)
    basic.pause(200)
})
```

## Change in brightness caused by the LED

Based on your observations you may have realized that the measurements are more stable with repeated measurement (only around +/- 10 lumens differences).
It should also be possible to subtract ambient light (h_umgebung) from the second measurement (h_mitLED) to mathematically eliminate ambient light.
Let's try it.

* ``||variables:Make a Variable...||`` with the name h_unterschied.
* Take the block ``||math:0 - 0||`` and subtract ambient light (h_umgebung) from the second measurement (h_mitLED).
* Show only the difference on the OLED display 🖥️. Remove the display outputs that are no longer needed.
* You can remove the first pause; the second pause is still useful so you can read the values.
* 📥 Press `|Download|` and observe the values on the OLED display 🖥️.
  * Place a Duplo figure between the LED and the light sensor.
  * Observe how the brightness difference changes.

```blocks
// @hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    h_umgebung = messeMax()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    h_mitLED = messeMax()
    // @highlight
    h_unterschied = h_mitLED - h_umgebung
    // @highlight
    smartfeldAktoren.oledWriteNum(h_unterschied)
    basic.pause(200)
})
```

## 👥 Detect and count a figure

* ``||variables:Make a Variable...||`` with the name **anzahlPersonenInWarteschlange** and
set it to 0 at the beginning of the ``forever`` loop.
* At the end of the ``forever`` loop, check whether the brightness difference (h_unterschied) is less than 100 (you can adjust this value if needed). If so, increase **anzahlPersonenInWarteschlange**.
Use ``||logic:if true then||``, ``||logic:0 < 0||``, and
``||variables:change anzahlPersonenInWarteschlange by 1||``.
* Show **anzahlPersonenInWarteschlange** with ``||basic:show number||`` on the micro:bit.
* 📥 Press `|Download|` and check the LED display 🟥.
  * Is the person at the first position counted correctly?
  * Do you already have ideas how to count people at all nine positions? <br />
⬛⬛🟥⬛⬛<br>
⬛🟥🟥⬛⬛<br>
⬛⬛🟥⬛⬛<br>
⬛⬛🟥⬛⬛<br>
⬛🟥🟥🟥⬛<br>

```blocks
// @hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    strip.show()
    h_umgebung = messeMax()
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    strip.show()
    h_mitLED = messeMax()
    h_unterschied = h_mitLED - h_umgebung
    smartfeldAktoren.oledWriteNum(h_unterschied)
    // @highlight
    if (h_unterschied < 100) {
        anzahlPersonenInWarteschlange += 1
    }
    // @highlight
    basic.showNumber(anzahlPersonenInWarteschlange)
    basic.pause(200)
})

```

## Develop an idea for measuring at different positions

If you look at the code in the ``forever`` loop in the tooltip (``|💡|`` bulb in the lower left), you might get the idea to run the yellow marked part nine times (once for each hole in the model), but with a small change: the LED number must increase by one each time. You could do this by copying the code nine times, but a programmer would **wrap a loop around it**.

* As soon as you understand the basic idea, press ``|next|``.

```blocks
// @hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)

basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()

    // @highlight
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.Black))
    // @highlight
    strip.show()
    // @highlight
    h_umgebung = messeMax()
    // @highlight
    strip.setPixelColor(2, neopixel.colors(NeoPixelColors.White))
    // @highlight
    strip.show()
    // @highlight
    h_mitLED = messeMax()
    // @highlight
    h_unterschied = h_mitLED - h_umgebung
    // @highlight
    smartfeldAktoren.oledWriteNum(h_unterschied)
    // @highlight
    if (h_unterschied < 100) {
        anzahlPersonenInWarteschlange += 1
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
    basic.pause(200)
})

```
## Detect and count people at multiple positions

* Use the block ``||loops:for index from 0 to 4||`` to run the previously mentioned blocks nine times. You need to change the 4 to 8.
* Replace the LED number (currently = 2) with ``||math:0 + 0||`` and then change it to ``||variables:index|| + 2``. This is needed so the LED with index 2 lights up first.
* After each pass (at the bottom of the loop) turn off all LEDs with ``||neopixel:strip clear||``.
* Remove all ``||basic:pause (ms)||`` blocks from the code; they are no longer needed.
* Add a line break using ``||SmartfeldAktoren:new line||`` directly after the brightness difference is written on the OLED display 🖥️.
* 📥 Press `|Download|` and test whether the people are correctly counted when you place, for example, three Duplo figures 🦹‍♂️.<br />
🟥🟥🟥🟥🟥<br>
⬛⬛⬛⬛🟥<br>
🟥🟥🟥🟥🟥<br>
⬛⬛⬛⬛🟥<br>
🟥🟥🟥🟥🟥<br>

```blocks
//@hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    //@highlight
    for (let Index = 0; Index <= 8; Index++) {
        //@highlight
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        //@highlight
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        smartfeldAktoren.oledWriteNum(h_unterschied)
        //@highlight
        smartfeldAktoren.oledNewLine()
        if (h_unterschied < 100) {
            anzahlPersonenInWarteschlange += 1
        }
        //@highlight
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})
```

## Cleaner output on the OLED display 🖥️

We can make the OLED display output more useful. For troubleshooting it can help to show the LED position, the measurement value, and the evaluation (person **X** / no person **-**). Example output:

    P0: X :120
    P1: - :5
    P2: - :2
    etc.

* ``||functions:Make a Function...||`` (found in Advanced)
    * Add three parameters: number, number, and text.
    * Name the function **schreibeInfosAufDisplay**.
    * Rename **num** to **position**.
    * Rename **num2** to **wert**.
    * Rename **text** to **symbol** and click **Done**.
* ``||variables:Make a Variable...||`` and name it **zeile**.
We will store the text of one line there (for example P0: X :120).
* Take the block ``||variables:set zeile to 0||`` and place it in the created function.
* Open ``||Advanced||``. Under ``Text`` you will find ``||text:join "Hello" "World" - +||``.
Assign this block to the variable **zeile** (instead of **0**).
* Now join the text **"P"**, the variable **position**, a **": "**, the variable **symbol**, a **" :"** and the variable **wert**.
* Then show the variable ``||variables:zeile||`` on the OLED display 🖥️ using ``||SmartfeldAktoren:write string and new line||``.
* Insert the function ``||functions:schreibeInfosAufDisplay||`` at the correct two spots in the ``forever`` loop as shown in the tooltip (💡 lower left).
* You can now remove the previous OLED display outputs.
* 📥 Press `|Download|` and test whether the OLED display 🖥️ output is correct when placing, for example, three Duplo figures 🦹‍♂️ (your output can vary):<br />

    P0: X :120
    P1: - :5
    P2: - :2
    P3: X :204
    P4: - :4
    P5: - :2
    P6: X :140
    P7: - :2

* If you have another idea for visualizing the OLED display, feel free to program something else!

```blocks
//@highlight
function schreibeInfosAufDisplay (position: number, wert: number, _symbol: string) {
    zeile = "P" + position + ": " + _symbol + " :" + wert
    smartfeldAktoren.oledWriteStrNewLine(zeile)
}
//@hide
function messeMax () {
    ANZAHL_MESSUNGEN = 10
    maximum = 0
    for (let index = 0; index < ANZAHL_MESSUNGEN; index++) {
        if (smartfeldSensoren.getHalfWord_Visible() > maximum) {
            maximum = Math.max(maximum, smartfeldSensoren.getHalfWord_Visible())
        }
    }
    return Math.round(maximum)
}
let h_unterschied = 0
let h_mitLED = 0
let h_umgebung = 0
let anzahlPersonenInWarteschlange = 0
let maximum = 0
let ANZAHL_MESSUNGEN = 0
let zeile = ""
smartfeldSensoren.initSunlight()
smartfeldAktoren.oledInit(128, 64)
let strip = neopixel.create(DigitalPin.P1, 16, NeoPixelMode.RGB)
strip.setBrightness(255)
basic.forever(function () {
    anzahlPersonenInWarteschlange = 0
    smartfeldAktoren.oledClear()
    for (let Index = 0; Index <= 8; Index++) {
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.Black))
        strip.show()
        h_umgebung = messeMax()
        strip.setPixelColor(Index + 2, neopixel.colors(NeoPixelColors.White))
        strip.show()
        h_mitLED = messeMax()
        h_unterschied = h_mitLED - h_umgebung
        if (h_unterschied < 100) {
            //@highlight
            schreibeInfosAufDisplay(Index, h_unterschied, "X")
            anzahlPersonenInWarteschlange += 1
        } else {
            //@highlight
            schreibeInfosAufDisplay(Index, h_unterschied, "-")
        }
        strip.clear()
    }
    basic.showNumber(anzahlPersonenInWarteschlange)
})

```

## Congratulations 🏆 - you have successfully completed Part 1 🚀

* Continue with Part 2: [Part 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/warteschlange-sensorik-part-2)
* If something is not working yet, here is a working version to test: [Solution Part 1](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/warteschlange-sensorik-part-1-solution)
