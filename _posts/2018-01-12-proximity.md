---
title: "Proximity Detection using Bluetooth Low Energy Technology to Identify Seated People"
date: 2018-01-12
tags: [bluetooth low energy, machine learning, kalman filter, sensor fusion, web application]
excerpt: "Non-invasive monitoring system to recognize if a person is seated in a chair using Bluetooth Low Energy (BLE) technology."
---

# Introduction
The focus of this work is to present a non intrusive solution for proximity detection of a human being, more specifically, applied to the case of identifying if a person is seated in a chair using **Bluetooth Low Energy** technology.

The Bluetooth Low Energy **(BLE)** signal is transmitted in the 2.4 GHz radio frequency. This means that the signal may be distorted by interference from specific elements in the environment such as water. It is important to note that a human body is about 60% water, it absorbs signals in the 2.4 GHz band. Taken this into account, by identifying such distortion it is possible to infer human proximity.

With the help of a beacon transmitter that uses BLE technology it is possible to measure the amount of distortion in the signal when a human being is present between the BLE beacon and the BLE client. By using the RSSI (received signal strength indicator) as a measurement value, these changes can be determined. When a person stands between the devices, the signal is not completely shielded, because Bluetooth signals are omnidirectional, however, the RSSI decreases.

A system is designed around the measurements collected from the BLE client and beacon(s). 
The system is composed by a web API that is implemented to store and process all the measurements sent from the devices and a front end application to visualize and manage all the information of the system. 

The BLE signal is noisy, it is affected by several environment variables. A filter is introduced to remove the noise from the signal. More over, to further improve the estimations, more beacons are added, providing more information and a clearer picture of the state of the chair. To reduce the noise and incorporate the measurements of different beacons, a Kalman Filter with sensor fusion is implemented.

Finally a machine learning algorithm is used to analyze and learn the behavior of the signal and make the according predictions about whether someone is seated or not.

The main contribution of this work is to demonstrate that Bluetooth Low Energy technology can be used as a proximity detection solution to identify seated people.

# Architecture
For the roles of beacon and client, the NodeMCU esp32 development board was chosen. The esp32 are small, low cost, low power and very versatile micro controllers. As stated on the offitial website \cite{esp32}:

The esp32 has a hybrid Wi-Fi and Bluetooth Chip, meaning it can perform as a complete standalone system or as slave to a host MCU. They feature ultra low power consumption since they are engineered for mobile devices, wearable electronics and IoT applications. They are also capable of functioning reliably in industrial environments, with temperatures ranging from \(-40^0\)C to \(+125^0\)C.

This is just a brief summary of the capabilities of the micro controller. The important take away from this is that they can perform as a BLE beacon as well as a BLE client, and also connect to Wi-Fi and send the measurements collected as an HTTP request. 

The development boards have USB ports that can be used to power the micro controller. Each of the components used are powered with power banks using this port. They are attached to the chair using simple double sided tape. For the initial setup both beacons are placed one in front of the other underneath the chair. As shown in the following figures:

<figure class="third">
  <img src="/images/chair1.png">
  <img src="/images/client_chair1.png">
  <img src="/images/chair1_1_beacon.png">
  <figcaption>Initial setup</figcaption>
</figure>