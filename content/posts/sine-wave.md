---
title: "How to Make a Sine Wave"
date: 2023-08-07T09:07:30+07:00
draft: true
description: Pulse-width modulation, AVR timer programming, and a few ATmega328P registers.
---

- we can output immediately, but it's not smooth
- additionally, direct digital outputs can't do negative values

## The H-Bridge
- H-bridge for switching between +/- values
- PWM for controlling H-bridge
- manipulating PWM into producing sine waves

## Sinusoidal Pulse-Width Modulation
- concept of SPWM
- LC for smoothing

## PWM on the Arduino
- making SPWM with lookup tables
- Arduino PWM

## AVR Timer Programming 
- timer registers

## Yay!
- sine waves!
