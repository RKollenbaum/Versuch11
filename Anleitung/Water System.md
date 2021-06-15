<!--
author:   Robert Kollenbaum
  fn


version:  0.0.9

language: en

narrator: US English Male

comment:  LiaScript template for the AVR8js simulator.

script:   https://cdn.jsdelivr.net/gh/liatemplates/avr8js@0.0.9/dist/index.js

@AVR8js.sketch: @AVR8js.project(@0,sketch.ino)

@AVR8js.project
<script>
let id = "@0"

let name = [
  "@1", "@2", "@3", "@4", "@5", "@6", "@7", "@8", "@9"
  ]
  .map((e) => e.trim())
  .filter((e) => { return (e[0] !== '@' && e !== "") })

let content = [
  `@input(0)`,
  `@input(1)`,
  `@input(2)`,
  `@input(3)`,
  `@input(4)`,
  `@input(5)`,
  `@input(6)`,
  `@input(7)`,
  `@input(8)`,
  `@input(9)`
  ]

let sketch;
let files = []

for(let i=0; i<name.length; i++) {
  if (name[i] == "sketch.ino") {
    sketch = content[i]
  } else {
    files.push({name: name[i], content: content[i]})
  }
}

AVR8js.build(sketch, files)
   .then((e) => {
     if (e.stderr) {
       let msgs = []

       for(let i = 0; i<name.length; i++) {
         msgs.push([])
       }

       let iter = e.stderr.matchAll(/(\w+\.\w+):(\d+):(\d+): ([^:]+):(.+)/g)

       for(let err=iter.next(); !err.done; err=iter.next()) {
         msgs[name.findIndex((e) => e==err.value[1])].push({
           row :    parseInt(err.value[2]) - 1,
           column : parseInt(err.value[3]),
           text :   err.value[5],
           type :   err.value[4]
         })
       }
       send.lia(e.stderr, msgs, false)
       send.lia("LIA: stop")
     }
     else {
       console.debug(e.stdout)

       if (e.hex) {
         let runner = AVR8js.execute(e.hex, console.log, id)

         send.handle("input", (input) => {
            runner.serial(input.slice(0, -1))
         })

         send.lia("LIA: terminal")

         send.handle("stop", e => {
           if(runner) {
             runner.stop()
             runner = null
             console.debug("execution stopped")
           }
         })
       } else {
         send.lia("LIA: stop")
       }
     }
   })
"LIA: wait"
</script>

@end



@AVR8js.asm
<script>
let id = "@0"

AVR8js.buildASM(`@input`)
   .then((e) => {
     if (e.stderr) {
       let msgs = []

       let iter = e.stderr.matchAll(/main\.s:(\d+):(\d+): ([^:]+):(.+)/g)

       for(let err=iter.next(); !err.done; err=iter.next()) {
         msgs.push({
           row :    parseInt(err.value[1]) - 1,
           column : parseInt(err.value[2]),
           text :   err.value[4],
           type :   err.value[3].toLower()
         })
       }
       send.lia(e.stderr, [msgs], false)
       send.lia("LIA: stop")
     }
     else {
       console.debug(e.stdout)

       if (e.hex) {
         let runner = AVR8js.execute(e.hex, console.log, id)

         send.handle("input", (input) => {
            runner.serial(input.slice(0, -1))
         })

         send.lia("LIA: terminal")

         send.handle("stop", e => {
           if(runner) {
             runner.stop()
             runner = null
             console.debug("execution stopped")
           }
         })
       } else {
         send.lia("LIA: stop")
       }
     }
   })
"LIA: wait"
</script>

@end

-->

# Plant Watering System

Kursprototyp: DIY Bewässerungsystem basierend auf:

!?[video](https://www.youtube.com/watch?v=DOaDnYj3vfI)




## Inhaltsverzeichnis
---

Kursablauf

1. **Materialübersicht**
2. **Versuchsaufbau**
3. **Übungsquiz**



### Materialübersicht


| Technik  | Kabel  | Kits | Sonstiges  |
| :--------- | :--------- | :--------- | :--------- |
| Arduino Uno     | Male-to-Male Jumper wires   | Breadboard     | Relay Module   |
| Battery and battery pack | Male-to-Female Jumper wires | 9V snap connector kit | Container filled with water|
| Raspberry Pi 4 | usb C cable | Plant watering kit | Water pump & tubing |


### Versuchsaufbau

1. Connect items to Arduino
-  Relay: connect the vcc to the 5 volt pin of Arduino
-  Sensor: connect A0 to the ground of Arduino
-  Sensor: connect the vcc to the 3.3 volt pin

![Bild](Screens/Ardu.png)


### Versuchsaufbau

---

2. Connect cables to the pump
-  Connect the red wire to the pump
-  connect the other end to the NC on the relay


![Bild](Screens/Pump.png)


## Wiederholung

Was für eine Art Pumpe wird beöntigt?

[[Wasserpumpe]]

Welche Sensoren sind erforderlich?

[[Feuchtigkeitssensor]]

Wie viel Volt benöntigt das Connenctor Kit?

[[9V]]


### Wiederholung

Mit welchem item verbindet man das Relay?

[[X]] Nur mit Arduino
[[ ]] Mit allen anderen items
[[X]] Es wird kein Relay verwendet




Mit welcher Flüssigkeit sollte man Pflanzen gießen:

    [[X]] Wasser
    [[ ]] Sprudelwasser
    [[X]] destilliertes Wasser



## Eingebetter Code, 3D Modell

??[sketchfab](https://sketchfab.com/3d-models/mossy-water-fountain-free-agustin-honnun-28fdbbf5c2784307b47465b86a40eb45)


### Eingebetter Code, Storymap

??[storymaps](https://storymaps.arcgis.com/stories/7a736236fc23415e8b3b08075b98f8d2)


### Simulation LEDs

<div id="example1">
<wokwi-led color="red"   pin="13" label="13"></wokwi-led>
<wokwi-led color="green" pin="12" label="12"></wokwi-led>
<wokwi-led color="blue"  pin="11" label="11"></wokwi-led>
<wokwi-led color="blue"  pin="10" label="10"></wokwi-led>
<span id="simulation-time"></span>
</div>

``` cpp
byte leds[] = {13, 12, 11, 10};
void setup() {
  Serial.begin(115200);
  for (byte i = 0; i < sizeof(leds); i++) {
    pinMode(leds[i], OUTPUT);
  }
}

int i = 0;
void loop() {
  Serial.print("LED: ");
  Serial.println(i);
  digitalWrite(leds[i], HIGH);
  delay(250);
  digitalWrite(leds[i], LOW);
  i = (i + 1) % sizeof(leds);
}
```
@AVR8js.sketch


### Simulation Sensor DHT22

<wokwi-dht22></wokwi-dht22>
