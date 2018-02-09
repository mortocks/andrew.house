---
layout: post
title: Puppy bin deterrent
tags: proton-particle pet accelerometer
cover: assets/images/april-rubbish.jpg
date: 2018-02-01 04:00:00
author: Author
---

My dog April has taken to pushing the bin over when I'm not home. So lets hook up a deterrent to correct this behaviour when I'm not at home.

<!--more-->

Let me introduce you to April. She's my 11 month old, half french bulldog, half pug. April has been one of the big drivers for my home automation projects because I've become a besotted "doggy daddy" and I like to check in on her while I'm at work and make sure she has everything she needs like  the air-conditioner and fans on hot days. She's a delicate flower you know.

She's my perfect little angel and I adore her...

<amp-img src="{{ site.baseurl }}assets/images/april-rubbish.jpg" width="656" height="400" layout="responsive" alt="" class="mb3"></amp-img>

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

To keep things simple *(and cheap)* I'm going to use [pushover.net](https://pushover.net) to send push notifications to my phone when the buzzer is triggered. This is usually done by a simple ```curl``` request but unfortunately the proton particle doesn't support this via the IDE so I'll have to setup a webhook to do it for me.


### The code

```javascript
/*
 * Wait for accelerometer event then trigger
 * them trigger the buzzer and webhook
 */

int buzzer = D7;  // The buzzers pin on the particle

void setup() {

    // Setup pin as an output
    pinMode(buzzer, OUTPUT);
}

void loop() {

    // Trigger the buzzer for 300ms
    digitalWrite(buzzer, HIGH);
    delay(300);
    digitalWrite(buzzer, LOW);

    // Trigger the webhook to send a push notification
    Particle.publish("trigger", 1, PRIVATE);
}

```


