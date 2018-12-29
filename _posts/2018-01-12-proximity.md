---
title: "Proximity Detection using Bluetooth Low Energy Technology to Identify Seated People"
date: 2018-01-12
tags: [bluetooth low energy, machine learning, online k-means clustering, kalman filter, sensor fusion, web application, postgresql, service oriented architecture]
excerpt: "Non-invasive monitoring system to recognize if a person is seated in a chair using Bluetooth Low Energy (BLE) technology."
---

I presented this project as my master's degree thesis. This page represents an overview, the details of the complete work can be found [here](/images/thesis.pdf).

It is composed of 4 different github repositories:

1. [BLE client](https://github.com/danielRM88/ble-client)
2. [BLE server/beacon](https://github.com/danielRM88/ble-server)
3. [Rails API](https://github.com/danielRM88/proximity-api)
4. [Reactjs frontend](https://github.com/danielRM88/proximity-frontend)

## Table of Contents
- [Introduction](#introduction)
- [Architecture](#architecture)
    - [The signal](#the-signal)
    - [Managing the noise](#managing-the-noise)
    - [Learning the behaviour](#learning-the-behaviour)
    - [Complete system](#complete-system)
- [Description of application](#description-of-application)
    - [Video of the application working](#video-of-the-application-working)

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

### The signal
It is important to note that once the components are set in place, their position does not change. This means that the signal should remain constant as long as there is no noise and no one is seated on the chair.

<figure>
  <img src="/images/beacon_signal.png">
  <figcaption>Figure 1: signal of the beacon</figcaption>
</figure>

The previous image shows the first 200 measurements taken from a beacon placed under the chair. It can be appreciated that the data is noisy and has some really pronounced peaks.

<figure>
  <img src="/images/beacon_signal_distorted_by_seated.png">
  <figcaption>Figure 2: signal of the beacon when someone sits down</figcaption>
</figure>

The previous images hows how the signal gets distorted when someone sits down on the chair. This distortion comes from the presence of a human body placed between the beacon and the client. It also shows that by learning how to identify this distortion, it would be possible to achieve the goal of proximity detection.

It is clear by looking at the previously mentioned figures, that it is important to try to reduce the noise from the signal as much as possible to differentiate between distortion by noise and distortion by human presence.

### Managing the noise
A very noisy signal is difficult to analyze and will cause problems at the time of differentiating between someone being seated or not. As mentioned before, noise reduction is essential for the achievement of the main goal of the project.

For this objective, a Kalman Filter is implemented. Its purpose is to filter out the noise from the signal. 

**NOTE: for a very complete and accessible explanation about filters and the Kalman filter in general please refer to this great [repo](https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python).**

The Kalman filter is an algorithm that uses the state-space approach to filtering. It takes a series of measurements observed over time to produce estimates of unknown variables. It combines the limited knowledge available on how a system behaves with the noisy observations taken by the sensors to produce the best possible estimate of the state of the system. It was developed by Rudolf Emil Kálmán, and it is used in a wide variety of fields. This is due to the fact that it is computationally light and very fast, which makes it a very powerful tool.

As the project moves forward, more beacons will be added to the chair in order to try to get a clearer picture of the signal. A very important feature of the filter is that it allows to merge the different measurements from the different beacons into one single state estimation with better accuracy. This is known as sensor fusion.

**This is a brief overview of the algorithm. For a more detailed description of its variables and its underlying math please feel free to check the following [document](/images/thesis.pdf) (there is a whole section dedicated to this).**

### Learning the behaviour
the key part of the project is to identify or learn the distortion caused by a human's body being present between the beacon(s) and the client. For this task, a machine learning algorithm is implemented.

For this project an unsupervised learning algorithm known as k-means clustering is selected due to its simplicity. It is a rather straightforward application of the algorithm except for the fact that it will be applied over an online system, i.e. new measurements will keep arriving and the algorithm must be able to process one observation at a time. For this, a recursive version is necessary.

This recursive version is known as "Online K-means" and its implementation can be seen in this [post](https://www.researchgate.net/publication/281295652_Lecture_Notes_on_Data_Science_Online_k-Means_Clustering).

### Complete system
A web application programming interface (API) will receive the measurements sent from the BLE client. Every measurement will be stored in a database.

Data visualization has become very important in today's world. This is the reason a web interface is developed to manage and monitor the system and its performance.

The web interface will communicate with the web API and display all the information in an effective way, to allow for proper analysis.

Putting together all the pieces previously described, the final architecture as shown in Figure 3 is composed of:

* 1 or more beacons
* 1 client
* Kalman Filter
* Online K-means Algorithm
* Web API
* Web interface

<figure>
  <img src="/images/final_architecture.png">
  <figcaption>Figure 3: architecture of the system</figcaption>
</figure>

The client, after getting the measurement from the beacon(s), sends it to the Web API, where it is processed by a Kalman filter to reduce the noise as much as possible and merge all the measurements together. The output of the filter is then fed to the machine learning algorithm so that it learns the behaviour of the signal and perform the predictions about whether someone is seated or not. This is all displayed by the web interface.

## Description of application
In order for the project to be scalable and flexible enough to be feasible and practical, a multilayer service oriented architecture was needed to provide the proper support structure for the complete process.

The less computation the micro controllers do, the easier it is to replace them. The hardware solution for this project was developed with the only purpose of analyzing the feasibility of using BLE technology for proximity detection, it is by no means a market ready solution, therefore the system was developed in a way so that the hardware side, i.e. the BLE client and BLE server, could be easily replaced. This is the reason why a web server that stores all the measurements and performs all the computations is created.

For the purposes of this project, the advantages of having a multilayer service oriented architecture out weight those of other types of applications. Among those advantages are: the fact that it can be accessed anywhere in the world, is highly robust, the use of web services and an API makes it highly flexible and resilient to changes in other parts of the system, such as the interface or the hardware solution, and it is easier to scale if the need arises to do so.

There are several reasons why the decision of separating the UI from the API was made. The most important one is the fact that with the separation comes modularity which improves testing, readability and maintainability. Another reason is reusability, which means that the API can be reused by other applications. Also means that it is easier to change the display of the system should the need arise. This is an important feature given that data visualization has become essential to analysis.

For the web server and API, **Ruby and Ruby on Rails** were chosen for the language and framework. With those tools it is possible to setup a working web application quickly, which is exactly what is needed to validate the hypothesis of this project. It is important to note that as a framework, Ruby on Rails, has come a long way in terms of scalability and performance, it is by no means a limitation today, but if the need arises, there is always the possibility to migrate to a more performance oriented framework just as long as the API is implemented in the same way, everything else should fall in place.

For the web interface that will display the information stored in the server, **React.js** is chosen. An important feature of React.js is that it is based on components, each component has its own logic and controls its rendering, and the fact that they can be combined into bigger components makes the code highly reusable. The virtual DOM is another of React.js main features, this is because DOM manipulation is one of the biggest performance bottlenecks in front end applications. React tackles this with a virtual DOM which is basically, a copy of the real DOM that lives in memory. React performs any changes really fast in the virtual DOM and then uses a highly efficient algorithm to determine which changes should be applied to the real DOM providing higher performance and a better user experience.

The architecture of the front end application is heavily based in the one displayed in this great [post](https://medium.com/@rajaraodv/a-guide-for-building-a-react-redux-crud-app-7fe0b8943d0f).

The use case for the system can be observed in Figure 4.

<figure>
  <img src="/images/system_use_case.png">
  <figcaption>Figure 4: system use case</figcaption>
</figure>

The application's data model can be seen in Figure 5. It displays all the models used in the development of the backend of the application.

<figure>
  <img src="/images/erd.png">
  <figcaption>Figure 5: data model</figcaption>
</figure>

The system is a web application that allows the user to manage and monitor all the information that gets generated by the micro controllers. The user can create chairs and beacons and also generate associations between them. By doing so, the system can determine which measurements belong to which chair, and therefore make predictions about whether someone is seated on it or not.

The application has an index page for chairs  that shows all the chairs available in the system. This can be seen in Figure 6, although at this point no chair has been created.

<figure>
  <img src="/images/chairs_index.png">
  <figcaption>Figure 6: index page for chairs</figcaption>
</figure>

By clicking on the 'New Chair' button the user is redirected to the form page to create a chair shown in Figure 7.

<figure>
  <img src="/images/chair_form.png">
  <figcaption>Figure 7: page for the creation of a chair</figcaption>
</figure>

In Figure 7 there is the option to apply the Kalman filter to the chair (Apply filter). This was made optional because the system was developed to be as flexible as possible. There might be situations where the chair with the filter under performs. But it was done mainly to allow for experimentation, to see how the system behaves without the filter as well. After the chair is created the user is redirected to the index page Figure 8.

<figure>
  <img src="/images/chairs_index_1_chair.png">
  <figcaption>Figure 8: index page for chairs</figcaption>
</figure>

The system also contains an index page for beacons that shows all the beacons available in the system. This can be seen in Figure 9, although at this point no beacon has been created.

<figure>
  <img src="/images/beacons_index.png">
  <figcaption>Figure 9: index page for beacons (No beacon created)</figcaption>
</figure>

By clicking on the 'New Beacon' button the user is redirected to the form page to create a beacon shown in Figure 10.

<figure>
  <img src="/images/beacon_form.png">
  <figcaption>Figure 10: page for the creation of beacons</figcaption>
</figure>

Figure 10 shows the fields needed to create a beacon, only two of them are important, these are the mac address and the chair it belongs to. The mac address should be unique to each beacon given that is the field by which the system differentiates them. In order for the beacon to be used by the system, it needs to be associated to a chair. After the beacon is created the user is redirected to the index page Figure 11.

<figure>
  <img src="/images/beacons_index_1_beacon.png">
  <figcaption>Figure 11: index page for beacons</figcaption>
</figure>

A chair can have multiple beacons assigned to it Figure 12.

<figure>
  <img src="/images/beacons_index_2_beacons.png">
  <figcaption>Figure 12: index page for beacons (2 beacons assigned to the same chair)</figcaption>
</figure>

Once the chair and its beacons are set up, in order for the system to start making predictions, the chair must be calibrated. To calibrate a chair, the "Edit" button must be selected from the index page Figure 8. This will take the user to the edit page of the chair shown in Figure 13. In this page, the user can update any information regarding the chair, including add a or remove its filter as well as start the calibration phase. In Figure 13 it can be observed that the chair is not calibrated, there is an indicator to the right side of the screen "Calibrated: false" which will turn into true when the chair is calibrated. After pressing the  "Start Calibration" button, the chair calibration begins, this is shown in Figure 14.

<figure>
  <img src="/images/edit_chair.png">
  <figcaption>Figure 13: edit page for chair</figcaption>
</figure>

<figure>
  <img src="/images/ongoing_calibration.png">
  <figcaption>Figure 14: edit page for chair with ongoing calibration</figcaption>
</figure>

After calibration is complete, the system is ready to start making predictions, this can be seen by pressing on the "Panel" button from the index page Figure 8. After pressing the button the user is redirected to the panel page for the selected chair Figure 15.

<figure>
  <img src="/images/chair_panel.png">
  <figcaption>Figure 15: panel page for chair</figcaption>
</figure>

In this page Figure 15, all the information regarding the chair can be found. Whether someone is seated or not is displayed in the square at the top left corner of the screen (red for no one seated, green otherwise Figure 16. The first plot shows the signals coming from all the beacons assigned to the chair. The plot below it, shows the estate estimation of the filter based on the measurements obtained from both beacons. The last one shows the variance of the estate estimation error, which can be seen, as mentioned in the chapter for the Kalman filter, as how much confidence there is in its estate estimation.

Below the square that shows whether there is someone seated or not in Figure 15, the graph for the online K-means algorithm can be found. The squares represent the position of the centroids of each cluster and the circle represents the last measurement obtained.

<figure>
  <img src="/images/chair_panel_seated.png">
  <figcaption>Figure 16: panel page for chair when someone is seated</figcaption>
</figure>

There is a section to collect the ground truth and evaluate how well the system is performing. This panel shown in Figure 17 where there previously was the k means cluster algorithm, is updated live as the new measurements are being collected. The values for precision, recall, specificity and accuracy are all being updated live so as to get a quick picture of how the system performs.

<figure>
  <img src="/images/chair_panel_gt.png">
  <figcaption>Figure 17: panel page for chair collecting ground truth</figcaption>
</figure>

There is one final panel shown in Figure 18 to adjust the process noise of the filter. This is to allow for experimentation during the test phase.

<figure>
  <img src="/images/chair_panel_filter.png">
  <figcaption>Figure 18: panel page for chair filter panel</figcaption>
</figure>

As mentioned before, the system is flexible enough to allow for a chair to not have a filter assigned to it as can be seen in Figure 19. In this case, if more than one beacon is assigned to the chair, the average of all measurements will be taken since there is no filter to perform sensor fusion. If the chair previously had a filter and it is removed, the chair will need to be calibrated again.

<figure>
  <img src="/images/chair_panel_no_filter.png">
  <figcaption>Figure 19: panel page for chair without a filter</figcaption>
</figure>

**For a more detailed description of the workflow of the system, all the sequence diagrams for each use case can be found in this [document](/images/thesis.pdf).**

### Video of the application working

[Video of application working](https://drive.google.com/file/d/1RquFUS-uHjrSQ0-S-1S3rDpsoxnxkULQ/view?usp=sharing)

