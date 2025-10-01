---
title: "Useless Surpise"
layout: post
categories: [Microcontroller, Building]
description: 
image:
  path: /assets/2025/uselessbox/box.jpg
  alt: Useless Surpise
---

Every year in the holiday season we celebrate the tradition of "Sinterklaas". And for that we make a 'surprise' (_Sur-pree-se_), a personally made gift for someone, usually a creative representation of something hiding the actual gift. This year I wanted to make something more technical than a surprise made of cardboard and duck tape, and decided to create a personal version of the useless box. A useless box is a device with just one task: to turn itself off!

I stumbled upon Bart Venneker’s [Useless Machine met Leeuw](https://www.bartvenneker.nl/Arduino/index.php?art=0023)which featured an energy-saving design that cuts off power when idle. This was a perfect feature since my box would stay wrapped until opened. I also found inspiration from an [escape room puzzle](https://iliketomakestuff.com/make-escape-room-puzzle/) using magnets and reed switches, which I incorporated to add a game element to the gift.

## Parts
To bring this project to life, I gathered the following components:

- **Microcontroller:** Arduino Nano
- **Motors:** SG90 Mini Servo, S3003 Servo
- **Switches:** Toggle Switch (MTS-202), Roller Microswitch, 4 Reed Switches
- **Power:** 9V Battery with clip
- **Extras:** LED, 100uF Capacitor, Resistors (1x 120Ω, 4x 10KΩ), 4 small magnets, Box (tea box from [Action](https://www.action.com/nl-nl/p/2553999/theebox-met-tekst/))

## How It Works
### Wiring
Based on my ideas I created the wiring diagram to start the build:

![Wiring_diagram](/assets/2025/uselessbox/wiring_diagram.png)
_Fig.1 Wiring Diagram_

With this wiring the power of the battery powering the Nano will be cut off if the box is "in rest". 

The Microswitch will be placed under the hendel that opens and closes the lid of the box. By lowering this handle it wil close the microswitch, cutting the power.

![Microswitch](/assets/2025/uselessbox/microswitch.jpg){: w="500"}
_Fig.2 Microswitch_

When the toggle switch is flipped, power flows to the Nano. The program then raises the lid handle just enough to release the Microswitch, keeping power running. The box can than switch the toggle switch to off again, and the Nano will still have power via the Microswitch.

If the box detects no activity (toggle switch isn’t flipped) for a set time, the handle lowers, pressing the Microswitch again and cutting off power.

The capacitor near the Nano’s **VIN** and **GND** pins prevents unwanted motor movements due to power fluctuations, according to the original project.

### Adding the Game: Reed Switches and Magnets
Reed switches are magnetic-sensitive switches and were connected to digital pins (pins 4, 5, 6, and 7). When a magnet triggers the reed switch, the circuit closes, and the pin reads HIGH.

For the reed switched I Googled some more to find how I could connect them to the Nano. They [need one end connected to VCC](https://www.instructables.com/Arduino-Reed-Switch/). The other end of the switch needs two wires, one goes via a 10K resistor to ground, and the second  wire to a port of the Nano. I tested this with one Reed switch to confirmed if they work with this wiring using a simple test sketch:
```c
const int pinSwitch2 = 8; //Pin Reed

int reedSwitch2 = 0;

void setup() {
  Serial.begin(9600);
  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(pinSwitch2, INPUT);
  Serial.println("READY!"); 
}

void loop() {
  reedSwitch2 = digitalRead(pinSwitch2); 
  Serial.println(reedSwitch2); 
  if (pinSwitch2 == 1){
    digitalWrite(LED_BUILTIN, HIGH);
  } else {
     digitalWrite(LED_BUILTIN, LOW); 
  }  
}
```

## Building and Testing
I started by testing the components on a breadboard, ensuring everything worked with the Arduino Nano. Step by step, I soldered the circuit, installed the motors, and tested the system at each stage.

Starting with the original code. I found that this didn't work flawless for my setup. The servo's had a different angle and the I adjusted the range for them.

The original code also contained a routine where where the arm was programmed to flip the switch when the door was closed.

The lid of the box had a windows made of glass. This turned out to be to heavy for the SG90 Mini Servo, so I removed the glass.

## Game
For the game I wanted to use a map of the World. This map would be divided in six squares (as seen in figure 2). Behind four of the six squares I glued one of the Reed switches.

![World_map](/assets/2025/uselessbox/worldmap.png)
_Fig.3 Map of the World_

The switches will react when a magnet is placed close to them. To give these magnets some extra grip I also glued some metal round behind the switches.

![Reed_Switches](/assets/2025/uselessbox/reed_switches.jpg){: w="500"}
_Fig.4 Reed Switches_

The magnets where placed in a small gift wrapper with a piece of cardboard, matching the size of the squares on the map. Attached to these gifts was a small poem, giving a hint to where the gift needed to be placed on the map.

## Program Flow
After reeding, testing and adjusting the code it work perfect for my setup and I added the part for the reed switches.

With this the program flow is:
1. When the toggle switch is flipped, the program checks if **all reed switches** are activated (magnets in correct place).
	1. If successful, the lid opens slowly, and the LED lights up. After a long delay the box will flip the toggle switch to off and close. 
	2. If not, the box will flip the toggle switch to off immediately and close the lid. 
2. After a delay or inactivity, the box shuts off, cutting power.

## Result
Personally, I really like the result. The surprise worked perfectly! The game added a bit of surprise to the 'surpreese'.

![Wiring](/assets/2025/uselessbox/wiring.jpg){: w="500"}
_Fig.5 Wiring_

![inside](/assets/2025/uselessbox/inside.jpg){: w="500"}
_Fig.6 Inside the finished box_

### The Game isn't Solved yet

![not solved](/assets/2025/uselessbox/not_solved.mov)

### The Game is Solved!

![solved](/assets/2025/uselessbox/solved.mov)

## Improvements
The servo lifting the lid of the box couldn't completely open the lid by the position I placed it. And I found out I couldn't rotate the hendel anymore because of the box and the placement of the servo. _If_ I would rebuild this, I would place the servo in such a way that if the puzzle was solved, it would completely open the box.

For now, it’s time to enjoy the holidays and let the box do what it does best: shut itself off!

And the wrapping could use some improvement....

## Code

```c
#include <Servo.h>
#include <EEPROM.h>

const int pinSwitch1 = 7; //Pin Reed
const int pinSwitch2 = 8; //Pin Reed
const int pinSwitch3 = 10; //Pin Reed
const int pinSwitch4 = 11; //Pin Reed

int solved1 = 0;
int solved2 = 0;
int solved3 = 0;
int solved4 = 0;

#define doorClosed 50
#define doorOpen 0
#define doorPeek 20
#define armIn 190
#define armOut 375
#define swtch 2
#define light 12

Servo klepServo;
Servo armServo;

unsigned long timeout = millis();
byte doorPos = doorClosed;
byte armPos = armIn;

void setup() {
  Serial.begin(9600);

  pinMode(pinSwitch1, INPUT);
  pinMode(pinSwitch2, INPUT);
  pinMode(pinSwitch3, INPUT);
  pinMode(pinSwitch4, INPUT);

  pinMode(light, OUTPUT);
  digitalWrite(light, LOW);

  pinMode(swtch, INPUT_PULLUP);

  klepServo.attach(9, 3100, 700);
  armServo.attach(6, 2700, 544);

  delay(100);

  for (int x = 0; x < 10; x++) {
    armServo.write(armIn);
    klepServo.write(doorClosed);
    delay(50);
  }
  klepServo.detach();
  armServo.detach();
}


void loop() {
  delay(500);

  if (digitalRead(pinSwitch1) == HIGH) {
    Serial.println("switch 1 high");
    solved1 = 1;
  }

  if (digitalRead(pinSwitch2) == HIGH) {
    Serial.println("switch 2 high");
    solved2 = 1;
  }

  if (digitalRead(pinSwitch3) == HIGH) {
    Serial.println("switch 3 high");
    solved3 = 1;
  }

  if (digitalRead(pinSwitch4) == HIGH) {
    Serial.println("switch 4 high");
    solved4 = 1;
  }

  // Turn off after 60 seconds of no activity
  if (millis() - timeout > 60000) {
    armServo.detach();
    digitalWrite(light, LOW);
    peek_1();
    // Move the doorServo to lowest position to press the microswitch and cut off the battery power
    Serial.println("lowest position");
    delay(500);
    for (int x = 0; x < 10; x++) {
      klepServo.write(75);
      delay(50);
    }
  }

  if (!digitalRead(swtch)) { // If the switch is in the UP position ..
    // First check if solved
    if (solved()) {
      // no need to do anything, power will be off
    } else {
    // not solved, switch 
      Serial.println("Switch is up");
      timeout = millis();            // reset the timeout value
      //    digitalWrite(light,HIGH);      // light on
      int pn = EEPROM.read(0);       // read the eeprom to get the next routine
      klepServo.attach(9, 3100, 700); // attach the servos
      armServo.attach(6, 2700, 544);

      switch (pn) {
        case 1: delay(500); routine_1(); EEPROM.write(0, 2); break;
        case 2: delay(500); routine_2(); EEPROM.write(0, 3); break;
        case 3: delay(500); routine_3(); EEPROM.write(0, 4); break;
        case 4: delay(500); routine_4(); EEPROM.write(0, 5); break;
        case 5: delay(500); routine_5(); EEPROM.write(0, 6); break;
        case 6: delay(500); routine_6(); EEPROM.write(0, 7); break;
        case 7: delay(500); routine_7(); EEPROM.write(0, 8); break;
        case 8: delay(500); routine_8(); EEPROM.write(0, 9); break;
        case 9: delay(500); routine_9(); EEPROM.write(0, 1); break;
        default: EEPROM.write(0, 1); break;
      }

      klepServo.detach();       // detach the servos
      armServo.detach();
    }
  }
}

void setDoor(byte pos, byte speedDelay) {
  Serial.println("Set Door " + pos);
  if (pos > doorPos) {
    for (int p = doorPos; p < pos ; p++) {
      klepServo.write(p);
      delay(speedDelay);
    }
  }
  else {
    for (int p = doorPos; p > pos ; p--) {
      klepServo.write(p);
      delay(speedDelay);
    }
  }
  doorPos = pos;
}

void setArm (byte pos, byte speedDelay) {
  Serial.println("Set Arm");
  if (pos > armPos) {
    for (int p = armPos; p < pos ;) {
      p = p + 5;
      armServo.write(p);
      delay(speedDelay);
    }
  }
  else {
    for (int p = armPos; p > pos ;) {
      p = p - 5;
      armServo.write(p);
      delay(speedDelay);
    }

  }
  armPos = pos;
}

bool solved() {
  if (solved1 == 1 && solved2 == 1 && solved3 == 1 && solved4 == 1) {
    digitalWrite(light, HIGH);     // light on
    klepServo.attach(9, 3100, 700);

    // Move the doorServo to highest position
    Serial.println("solved, highest position");
    delay(500);
    setDoor(doorOpen, 10);

    delay(30000);
    solved1 = 0;
    solved2 = 0;
    solved3 = 0;
    solved4 = 0;

    if (!digitalRead(swtch)) { // If the switch is in the UP position ..
      armServo.attach(6, 2700, 544);
      flipSwitchSlow(100);
      //      flipSwitchFast();
    }
    delay(500);
    armServo.detach();

    // Move the doorServo to lowest position to press the microswitch and cut off the battery power
    Serial.println("Solved, lowest position");
    digitalWrite(light, LOW);     // light on
    delay(500);
    for (int x = 0; x < 10; x++) {
      klepServo.write(75);
      delay(50);
    }
    return true;
  } else {
    return false;
  }
}

void peek_1() {
  Serial.println("Peek 1");
  klepServo.attach(9, 3100, 700);
  setDoor(doorPeek, 20);
  delay(2000);
  setDoor(doorClosed, 10);
}

void flipSwitchSlow(int spDelay) { // 25 is slow, 50 or 100 is even slower
  Serial.println("flipSwitchSlow");
  byte p = armPos;
  while (!digitalRead(swtch)) { // If the switch is in the UP position ..
    p = p - 5;
    armServo.write(p);
    delay(spDelay);
  }
  armPos = p;
  setArm(armIn, spDelay);
  armPos = armIn;
}

void flipSwitchFast() {
  Serial.println("flipSwitchFast");
  for (byte x = 0; x < 15; x++) {
    armServo.write(1);
    delay(50);
    if (digitalRead(swtch)) break;
  }

  for (byte x = 0; x < 15; x++) {
    armServo.write(armIn);
    delay(50);
  }
  Serial.println("Arm in");
  armPos = armIn;
}

void doorOpenFast() {
  Serial.println("doorOpenFast");
  for (byte x = 0; x < 10; x++) {
    klepServo.write(doorOpen);
    delay(25);
  }
  doorPos = doorOpen;
}

void doorCloseFast() {
  Serial.println("doorCloseFast");
  for (byte x = 0; x < 10; x++) {
    klepServo.write(doorClosed);
    delay(25);
  }
  doorPos = doorClosed;
}

void klepper() {
  Serial.println("klepper");
  for (byte y = 0 ; y < 5 ; y++) {
    for (byte x = 0; x < 10; x++) {
      klepServo.write(doorPeek);
      delay(15);
    }
    for (byte x = 0; x < 10; x++) {
      klepServo.write(doorClosed);
      delay(15);
    }
  }
}

void setArmFast(byte pos, byte speedDelay) {
  Serial.println("setArmFast");
  for (byte x = 0; x < 10; x++) {
    armServo.write(pos);
    delay(speedDelay);
  }
  armPos = pos;
};

void armPause(int d) {
  Serial.println("armPause");;
  armServo.detach();
  delay(d);
  armServo.attach(6, 2700, 544);
  delay(100);
}

void routine_1() {
  // open the door
  Serial.println("Routine 1");
  setDoor(doorOpen, 10);
  flipSwitchSlow(100);
  setDoor(doorClosed, 10);
  armServo.detach();
  delay(2000);
  // check!
  doorOpenFast();
  delay(1000);
  doorCloseFast();
}

void routine_2() {
  Serial.println("Routine 2");
  doorOpenFast();
  flipSwitchFast();
  doorCloseFast();
}

void routine_3() {
  Serial.println("Routine 3");
  setDoor(doorOpen, 10);
  // flip the switch
  setArmFast(armOut, 50);
  // hold the swich down for a moment
  armPause(1000);
  // lift the arm a bit
  for (byte r = 0; r < 2; r++) {
    setArmFast(armOut + 45, 10);
    delay(250);
    setArmFast(armOut, 10);
    delay(800);
  }
  armPause(1000);
  // arm back inside
  setArmFast(armIn, 50);
  doorCloseFast();

}

void routine_4() {
  Serial.println("Routine 4");
  routine_2();
  delay(700);
  check_3();
}

void routine_5() {
  Serial.println("Routine 5");
  routine_2();
  delay(500);
  klepper();
}

void routine_6() {
  Serial.println("Routine 6");
  setDoor(doorOpen, 50);
  setArm(armOut + 30, 50);
  delay(500);
  setArmFast(armOut, 50);
  setArmFast(armOut + 40, 50);
  delay(500);
  setArmFast(armIn, 50);
  doorCloseFast();
  routine_2();
}

void routine_7() {
  Serial.println("Routine 7");
  doorOpenFast();
  delay(500);
  setArmFast(armOut, 50);
  delay(100);
  flipSwitchFast();
  doorCloseFast();
  delay(100);
  armPause(2000); 
  check_2();
}

void routine_8() {
  Serial.println("Routine 8");
  setDoor(doorPeek, 10);
  delay(700);
  doorCloseFast();
  setDoor(doorPeek, 10);
  delay(1500);
  doorOpenFast();
  flipSwitchFast();
  doorCloseFast();
  armServo.detach();
  klepServo.detach();
  delay(1000);
  klepServo.attach(9, 3100, 700);
  delay(100);
  setDoor(doorPeek - 20, 10);
  delay(2000);
  doorCloseFast();
}

void routine_9() {
  Serial.println("Routine 9");
  setDoor(doorOpen, 50);
  flipSwitchSlow(100);
  doorOpenFast();
  setArmFast(armOut, 50); setArmFast(armOut + 110, 50);
  setArmFast(armOut, 50); setArmFast(armOut + 110, 50);
  setArmFast(armOut, 50); setArmFast(armIn, 50);
  doorCloseFast();
}


void check_2() {
  Serial.println("Check 2");
  doorOpenFast();
  setArmFast(armOut + 70, 50);
  armPause(1500);
  setArmFast(armIn, 50);
  delay(100);
  doorCloseFast();
}

void check_3() {
  Serial.println("Check 3");
  doorOpenFast();
  setArmFast(armOut, 50);
  armPause(1500);
  if (!digitalRead(swtch)) {
    routine_2();
    doorOpenFast();
  }
  setArmFast(armOut + 110, 50);
  delay(100);
  setArmFast(armOut, 50);
  armPause(1500);
  if (!digitalRead(swtch)) {
    routine_2();
  }
  setArmFast(armIn, 50);
  klepServo.attach(9, 3100, 700);
  delay(100);
  doorCloseFast();
}
```