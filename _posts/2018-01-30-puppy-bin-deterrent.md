---
layout: post
title: Puppy bin deterrent
description: My dog April has taken to pushing the bin over when I’m not home. So lets hook up a device to correct this behaviour when I’m not at home.
tags: proton-particle pet accelerometer
cover: /assets/images/april-rubbish.jpg
date: 2018-01-30 04:00:00
author: Andrew Morton
---

Let me introduce you to April. She's my 11 month old, half french bulldog, half pug. April has been one of the big drivers for my home automation projects because I've become a besotted "doggy daddy" and I like to check in on her while I'm at work and make sure she has everything she needs like  the air-conditioner and fans on hot days. She's a delicate flower you know.

She's my perfect little angel and I adore her...

![april in rubish](/assets/images/april-rubbish.jpg)

....most of the time.


## The problem

Recently April (like many dogs) has taken to knocking over the bins and spreading its contents all over the apartment. There's no point in getting angry when I get home because she won't understand what she did wrong but I still need to stop this behaviour while I'm not around.

I figured the best way to discourage her without physical barriers or causing her any harm is noise, so the plan is to trigger a short, but loud sound whenever she interacts with the bin. Hopefully after a few weeks with this rig attached to the bin, it will correct the behaviour and I'll be able to remove the device 🤞.

Ideally this device would be wireless (so April has less chance of causing more damage if she pushes the bin over) and provide some kind of notification to me so I know when she's touching the bin.

## The Setup

### Hardware

- [Particle proton](https://www.amazon.com/ZIYUN-Particle-Photon/dp/B01EX9LUT8/ref=sr_1_11?s=industrial&ie=UTF8&qid=1518176225&sr=1-11&keywords=particle+photon) will be the brains of the system
- [Arduino accelerometer](https://www.amazon.com/ADXL335-Accelerometer-Angular-Transducer-Arduino/dp/B076PZGMB2/ref=sr_1_5?ie=UTF8&qid=1518176099&sr=8-5&keywords=3-Axis+Accelerometer+Module+for+Arduino) to monitor if the bin moves
- [Dual Sound Piezo Buzzer 1-13V](https://www.jaycar.com.au/dual-sound-piezo-buzzer-1-13v/p/AB3456) to make a loud noise
- [Power 1000](http://todo.com) and [Battery](http://todo.com) for power and wireless operation


### Notifications

To keep things simple *(and cheap)* I'm going to use [pushover.net](https://pushover.net) to send push notifications to my phone when the buzzer is triggered. This is usually done by a simple ```curl``` request but unfortunately the proton doesn't support this via the IDE so I'll have to setup a webhook to do it for me.

<amp-img src="{{ site.baseurl }}assets/images/dog-bin-webhook.png" width="1330" height="1552" layout="responsive" alt="" class="mb3"></amp-img>

Now I have an event called ```motionDetected``` that I can call via ```Particle.publish()``` to send the push notification.


### The code

The code is pretty straight forward. I flashed the particle with the following code. 

```javascript
/*
  Bin Movement

  Reads an Analog Devices ADXL3xx accelerometer and if the movement
  is above a thrshold then play a buzzer and send a push notification

  The circuit:
  
    Accelerometer
    - analog 1: z-axis
    - analog 2: y-axis
    - analog 3: x-axis
    - analog 4: ground
    - analog 5: vcc
  
    Buzzer
    - GND: buzzer ground
    - D1: buzzer positive

  created 15 Feb 2018 
  by Andrew Morton

  This  code is in the public domain.

  http://andrew.house
*/

// Variable for logging
char publishString[40];

// these constants describe the pins. They won't change:

// Accelerometer pins
const int groundpin = A5;             // analog input pin 5 -- ground
const int powerpin = A4;              // analog input pin 4 -- voltage
const int xpin = A3;                  // x-axis of the accelerometer
const int ypin = A2;                  // y-axis of the accelerometer
const int zpin = A1;                  // z-axis of the accelerometer

// Buzzer pins
const int buzzer = D1;

// Stored properties of last movement for comparrison
int lastX = 0;
int lastY = 0;
int lastZ = 0;

// Amount of movement
int deltaX = 0;
int deltaY = 0;
int deltaZ = 0;

// Amount of movement required to trigger the buzzer
int threshhold = 15;

// Only trigger notifications every 10 seconds
const int notificationDelay = 10000;
int notificationCounter = 0;

void setup() {
    
  // initialize the serial communications:
  Serial.begin(9600);

  pinMode(groundpin, OUTPUT);
  pinMode(powerpin, OUTPUT);
  digitalWrite(groundpin, LOW);
  digitalWrite(powerpin, HIGH);

  pinMode(buzzer, OUTPUT);
}

void loop() {

  // Update the movement delta
  deltaX = abs(lastX - analogRead(xpin));
  deltaY = abs(lastY - analogRead(ypin));
  deltaZ = abs(lastZ - analogRead(zpin));

  // Reset values
  lastX = analogRead(xpin);
  lastY = analogRead(ypin);
  lastZ = analogRead(zpin);

  // Combine deltas for a "total movement" delta
  int delta = deltaX + deltaY + deltaZ;
    
  // If there's a big enough movement, trigger an event
  if (delta > threshhold) {
    
    // Log data every 10 seconds
    if (notificationCounter <  notificationDelay) {
      sprintf(publishString, "%d %d %d %d", deltaX, deltaY, deltaZ, delta);
      Particle.publish("motionDetected", publishString, PRIVATE);
    }
    
    notificationCounter = 0;

    // Start buzzer
    digitalWrite(buzzer, HIGH);
    
    // Small delay so buzzer won't imdialty stop
    delay(100);
  } 
  
  else {
      
    // Stop buzzer
    digitalWrite(buzzer, LOW);
    notificationCounter = notificationCounter++;
    
  }


}

```

## The Result

TODO


