```package
iot-cube=github:Smartfeld/pxt-iot-cube#v1.1.2
sensors=github:Smartfeld/pxt-sensorikAktorikSmartfeld
```
### @explicitHints false

# IoT Tutorial Part 1

## 📗 Introduction, Part 1

**Prerequisites**
* micro:bit basics:
    * You can create programs and download them.
    * You know the entry points "on start" and "forever".
    * You know that programs are usually executed step by step (top to bottom). You can also use loops and conditions.
    * You understand that categories contain blocks (for example ``||basic:Basic||``) that can be used in programs.
    * Variables can be created, used, and changed.

**Learning outcome**

In this tutorial you build a program step by step that simulates a soap level and sends it to the internet via 🛜 LoRa. At the end you will have a working program that...

* displays the soap level 🧼.
* reduces or refills the soap level with ``||input:button presses||``:
    * ``||input:button A is pressed||``: the soap level 🧼 is reduced by 20%.
    * ``||input:button B is pressed||``: the soap level 🧼 is set back to 100%.

**Difficulty:** 🔥🔥⚪⚪

Click the 💡 icon if you need extra help and want to check your code.

```blocks
// Great! You found the hint. Use it if you get stuck.
let hinweisGefunden = true;
```

## 👁️ Prerequisites @showdialog
* For Part 1 you only need a micro:bit.
* If you want to use the IoT Cube right away, connect it like this. Watch the red marking:
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-anschliessen-klein.png)
* Set the switches for now:
    * Battery switch: **off**
    * LoRa module: **on**
![Image](https://reifab.github.io/pxt-iot-tutorial-eng/static/tutorials/iot-cube-power-switches-klein.png)
* Check that the micro:bit is connected.

## 🧼 Variable for the soap level
To store the soap level of the dispenser, we use a variable.
* To store the current soap level, we need a variable that shows the level in percent:
``||variables:Make a Variable...||`` and name it **seifenstandInProzent** 🧼.
* The dispenser is full at the beginning. Therefore, set the soap level to 100% in ``||basic:on start||`` using the variable: ``||variables:set seifenstandInProzent to 100||`` 🧼.

```blocks
let seifenstandInProzent = 100
```

## 🧼 Show the soap level
The goal is to show the current soap level on the IoT Cube.
* Get the block ``||led:plot bar graph||``🟥 and place it in the **on start** block directly under the variable **seifenstandInProzent** 🧼.
* Put the variable ``||variables:seifenstandInProzent||`` 🧼 in the first input of **plot bar graph of**.
* Set the range from **seifenstandInProzent** 🧼 to 100.
* 📥 Press `|Download|` and check the LED display:
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥
Are all LEDs on?

```blocks
let seifenstandInProzent = 100
// @highlight
led.plotBarGraph(
seifenstandInProzent,
100
)
```

## ➖ Reduce soap level with button A
The goal is to reduce the soap level by 20% each time button A is pressed.
For this we need a condition that checks whether button A is pressed. If it is pressed, the soap level should be reduced by 20%.
* To add this condition, get the block ``||logic:if true then||`` and place it in the existing ``||basic:forever||`` loop.
* Drag a new block ``||input:button A is pressed||`` into the **true** field.
* Change the variable ``||variables:seifenstandInProzent||`` 🧼 by -20.
* Plot the bar graph again.🟥 Duplicate this part from ``on start``.
* At the end, slow down the ``||basic:forever||`` loop by 150 ms with ``||basic:pause (ms)||``.

```blocks
basic.forever(function () {
    // @highlight
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    // @highlight
    basic.pause(150)
})
```

## 🧼 Prevent the level from going below 0
To avoid the level dropping below 0%, we need another condition that checks whether the soap level is below 0. If it is, set the soap level to 0%.
* Under the block ``||variables:change seifenstandInProzent by -20||``, add another ``||logic:if true then||`` block and check whether ``||seifenstandInProzent < 0||``. If yes, set ``||variables:set seifenstandInProzent to 0||``.
* Set the variable ``||variables:seifenstandInProzent||`` to 0% by setting the soap level 🧼 to 0.
[Here you can find more information about logical operators](https://makecode.microbit.org/blocks/logic/boolean)
* 📥 Press `|Download|` and check the 🟥 LED display. Press button A several times until the soap level would drop below 0%.

⬛⬛🟥⬛⬛<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥<br>
🟥🟥🟥🟥🟥
What happens? Does the display stay at 0?

```blocks
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        // @highlight
        if(seifenstandInProzent<0){
            // @highlight
            seifenstandInProzent = 0
        }
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    basic.pause(150)
})
```

## ➕ Refill the dispenser with button B
Now we want to refill the soap level 🧼 when button B is pressed.
We need a condition that checks whether button B is pressed. If it is, the soap level 🧼 should be set to 100%.
* Get the block ``||logic:if true then||`` and place it in the existing ``||basic:forever||`` loop, above ``||basic:pause (ms)||``.
* Drag the block ``||input:button A is pressed||`` into the **true** field and change button A to button **B**.
* Set the soap level to 100% by setting the variable ``||variables:seifenstandInProzent||`` 🧼 to 100.
* Plot the bar graph again. Copy this part from ``on start``.
* 📥 Press `|Download|` and check the 🟥 LED display.
Does the soap level fill back up to 100%?

```blocks
basic.forever(function () {
    if (input.buttonIsPressed(Button.A)) {
        seifenstandInProzent += -20
        
        if(seifenstandInProzent<0){
           
            seifenstandInProzent = 0
        }
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    // @highlight
    if (input.buttonIsPressed(Button.B)) {
        // @highlight
        seifenstandInProzent = 100
        led.plotBarGraph(
            seifenstandInProzent,
            100
        )
    }
    basic.pause(150)
   
})
```

## Congratulations 🏆 - you have successfully completed Part 1 🚀

* Continue with Part 2: [Part 2](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-2)
* If something is not working yet, here is a working version to test: [Solution Part 1](https://makecode.microbit.org/#tutorial:github:reifab/pxt-iot-tutorial-eng/docs/tutorials/seifenspender-part-1-solution)
