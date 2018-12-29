---
title: "Proximity Detection using Bluetooth Low Energy Technology to Identify Seated People"
date: 2018-01-12
tags: [bluetooth low energy, machine learning, kalman filter, sensor fusion, web application]
excerpt: "Non-invasive monitoring system to recognize if a person is seated in a chair using Bluetooth Low Energy (BLE) technology."
---

## Introduction
The focus of this work is to present a non intrusive solution for proximity detection of a human being, more specifically, applied to the case of identifying if a person is seated in a chair using **Bluetooth Low Energy** technology.

The Bluetooth Low Energy **(BLE)** signal is transmitted in the 2.4 GHz radio frequency. This means that the signal may be distorted by interference from specific elements in the environment such as water. It is important to note that a human body is about 60% water, it absorbs signals in the 2.4 GHz band. Taken this into account, by identifying such distortion it is possible to infer human proximity.

With the help of a beacon transmitter that uses BLE technology it is possible to measure the amount of distortion in the signal when a human being is present between the BLE beacon and the BLE client. By using the RSSI (received signal strength indicator) as a measurement value, these changes can be determined. When a person stands between the devices, the signal is not completely shielded, because Bluetooth signals are omnidirectional, however, the RSSI decreases.

A system is designed around the measurements collected from the BLE client and beacon(s). 
The system is composed by a web API that is implemented to store and process all the measurements sent from the devices and a front end application to visualize and manage all the information of the system. 

The BLE signal is noisy, it is affected by several environment variables. A filter is introduced to remove the noise from the signal. More over, to further improve the estimations, more beacons are added, providing more information and a clearer picture of the state of the chair. To reduce the noise and incorporate the measurements of different beacons, a Kalman Filter with sensor fusion is implemented.

Finally a machine learning algorithm is used to analyze and learn the behavior of the signal and make the according predictions about whether someone is seated or not.

The main contribution of this work is to demonstrate that Bluetooth Low Energy technology can be used as a proximity detection solution to identify seated people.

## Architecture
For the roles of beacon and client, the NodeMCU esp32 development board was chosen. The esp32 are small, low cost, low power and very versatile micro controllers. As stated on the offitial website \cite{esp32}:

The esp32 has a hybrid Wi-Fi and Bluetooth Chip, meaning it can perform as a complete standalone system or as slave to a host MCU. They feature ultra low power consumption since they are engineered for mobile devices, wearable electronics and IoT applications. They are also capable of functioning reliably in industrial environments, with temperatures ranging from -40<sup>0</sup>C to +125<sup>0</sup>C.

This is just a brief summary of the capabilities of the micro controller. The important take away from this is that they can perform as a BLE beacon as well as a BLE client, and also connect to Wi-Fi and send the measurements collected as an HTTP request. 

The development boards have USB ports that can be used to power the micro controller. Each of the components used are powered with power banks using this port. They are attached to the chair using simple double sided tape. For the initial setup both beacons are placed one in front of the other underneath the chair. As shown in the following figures:

<figure class="half">
  <img src="/images/chair1.png">
  <img src="/images/chair1_1_beacon.png">
  <figcaption>Initial setup</figcaption>
</figure>

The RSSI can be measured with the Friis transmission equation:

<img src="http://latex.codecogs.com/svg.latex?P_r%3DP_t%2A%28G_t%2AG_r%29%2Ac%5E2%2F%284+%5Cpi+Rf%29%5E2">

In this setup, the beacon sends periodically to the client the RSSI between them, calculated with the Friis transmission equation. The client in turn sends the measurement to a web server via an HTTP request.

### The Signal
It is important to note that once the components are set in place, their position does not change. This means that the signal should remain constant as long as there is no noise and no one is seated on the chair.

<figure>
  <img src="/images/beacon_signal.png">
  <figcaption>Signal of the beacon</figcaption>
</figure>

The previous image shows the first 200 measurements taken from a beacon placed under the chair. It can be appreciated that the data is noisy and has some really pronounced peaks.

<figure>
  <img src="/images/beacon_signal_distorted_by_seated.png">
  <figcaption>Signal of the beacon when someone sits down</figcaption>
</figure>

The previous images hows how the signal gets distorted when someone sits down on the chair. This distortion comes from the presence of a human body placed between the beacon and the client. It also shows that by learning how to identify this distortion, it would be possible to achieve the goal of proximity detection.

It is clear by looking at the previously mentioned figures, that it is important to try to reduce the noise from the signal as much as possible to differentiate between distortion by noise and distortion by human presence.

### Managing the noise
A very noisy signal is difficult to analyze and will cause problems at the time of differentiating between someone being seated or not. As mentioned before, noise reduction is essential for the achievement of the main goal of the project.

For this objective, a Kalman Filter is implemented. Its purpose is to filter out the noise from the signal. 

The Kalman filter is an algorithm that uses the state-space approach to filtering

<figure class="third">
  <img src="/images/state-space.png">
</figure>

It takes a series of measurements observed over time to produce estimates of unknown variables. It combines the limited knowledge available on how a system behaves with the noisy observations taken by the sensors to produce the best possible estimate of the state of the system. It was developed by Rudolf Emil Kálmán, and it is used in a wide variety of fields. This is due to the fact that it is computationally light and very fast, which makes it a very powerful tool.

It is important to keep in mind that the filter operates with the assumptions that all the inputs come from a Gaussian distribution.

After applying complex math derivations, the filter equation to calculate the state estimation of the system at time t looks as follows:

<figure class="third">
  <img src="/images/filter-eq.png">
</figure>

Where K<sub>0</sub> is the Kalman gain and e(t) is the innovation carried by the observation at time t.

**This is a brief overview of the algorithm, its variables and the math involved in deriving them. For more details please feel free to check the complete [thesis](/images/thesis.pdf) (there is a whole chapter dedicated to this).**


