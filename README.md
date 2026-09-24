# Object Sorting Robotic Arm Using Colour Detection

A colour-based object sorting robotic arm using MATLAB, Arduino Uno, computer vision, and servo motors.

## Project Overview

This project implements a robotic arm that detects the colour of an object using a webcam and MATLAB image processing. Based on the detected colour, the Arduino-controlled robotic arm performs an automated pick-and-place operation.

The system combines computer vision, image processing, and robotic control to demonstrate a simple automated sorting system.

## Features

- Real-time colour detection using MATLAB
- RGB to HSV colour processing
- Webcam/mobile camera input
- Arduino Uno control
- Three-servo robotic arm
- Automatic pick-and-place operation
- Colour-based object sorting

## Hardware

- Arduino Uno
- 3 × SG90 Servo Motors
- Webcam / Mobile Camera
- Laptop/PC
- USB connection
- Robotic arm structure

## Software

- MATLAB
- MATLAB Image Processing Toolbox
- MATLAB Arduino Support
- DroidCam

## System Architecture

```text
Camera / Webcam
       ↓
     MATLAB
       ↓
 Colour Detection
       ↓
    Arduino
       ↓
 Servo Motors
       ↓
 Pick and Place
