---
layout: post
title:  "Indoor Location using WiFi signal and SVM"
date:   2015-09-16 22:00:00 -0300
categories: Programming osx docker
---

Indoor location have been used in a wide variety of systems, to track users in a store, track employees in a big warehouse etc. The use of Bluetooth 4.0 beacons (e.g. iBeacons) is quite popular now, but they require a special kind of hardware (the beacons) to work. These devices require maintenance, battery replacement and are not that cheap. If you want more precision you need to install a few of them in the same area.

In this post I’ll show how to use the existing WiFi signal to locate users in a indoor environment. It can be a shopping mall, a big store with a big wifi infrastructure or it can be your house or apartment where you and your neighbors have a simple access point each.

The system has two stages, train and classify. The first stage maps the area with an Android application that collects the access points BSSIDs, their signal strength and the user position (like bedroom, living room, 2nd floor bedroom etc) and trains a machine learning classifier. In this example I’ll use the Support Vector Machine (SVM) classifier. In the second stage, with the SVM trained, we can use the Android app to capture the BSSID’s and signal strength and classify this new data as one of the house parts. 

To simplify the implementation, I’ll use the Weka software to do the machine learning stuff. The Android app generate a comma separated file with:


	//ROOM, UUID OF SAMPLE, BSSID, SSID, SIGNAL
	edroom1,7b9cb38d-b67e-4c3b-9310-00fc876e5b5a,2c:e4:12:85:18:8b,Matrix,-50
	bedroom1,7b9cb38d-b67e-4c3b-9310-00fc876e5b5a,54:e6:fc:c8:3c:1e,Nano,-86
	bedroom1,0abcf392-24a1-49df-9bae-9baf3798d428,2c:e4:12:85:18:8b,Matrix,-46
	bedroom1,0abcf392-24a1-49df-9bae-9baf3798d428,54:e6:fc:c8:3c:1e,Nano,-83
	bedroom1,468fecf8-3511-4915-bcca-14d9be99a5fb,2c:e4:12:85:18:8b,Matrix,-55
	bedroom1,468fecf8-3511-4915-bcca-14d9be99a5fb,54:e6:fc:c8:3c:1e,Nano,-89
	bedroom1,468fecf8-3511-4915-bcca-14d9be99a5fb,00:15:6d:dc:31:15,wireless,-93
	livingroom,4aacab86-6652-4592-b83d-fa38e1afc27c,2c:e4:12:85:18:8b,Matrix,-53
	livingroom,4aacab86-6652-4592-b83d-fa38e1afc27c,00:4f:62:10:ce:be,PLASBRASIL,-89
	livingroom,4aacab86-6652-4592-b83d-fa38e1afc27c,54:e6:fc:c8:3c:1e,Nano,-88
	livingroom,4aacab86-6652-4592-b83d-fa38e1afc27c,00:15:6d:dc:31:15,wireless,-93
	livingroom,6571c7a6-9887-492a-a004-d82955be4700,2c:e4:12:85:18:8b,Matrix,-57
	livingroom,6571c7a6-9887-492a-a004-d82955be4700,00:4f:62:10:ce:be,PLASBRASIL,-89
	livingroom,6571c7a6-9887-492a-a004-d82955be4700,54:e6:fc:c8:3c:1e,Nano,-88
	livingroom,6571c7a6-9887-492a-a004-d82955be4700,00:15:6d:dc:31:15,wireless,-93
	livingroom,792e9923-6aa6-41df-b1a6-03ebf9186816,2c:e4:12:85:18:8b,Matrix,-58
	livingroom,792e9923-6aa6-41df-b1a6-03ebf9186816,54:e6:fc:c8:3c:1e,Nano,-88
	livingroom,792e9923-6aa6-41df-b1a6-03ebf9186816,30:f3:1d:32:0b:8f,VIVO Wi-Fi,-90
	livingroom,792e9923-6aa6-41df-b1a6-03ebf9186816,2c:e4:12:85:18:8b,Matrix,-62
	livingroom,792e9923-6aa6-41df-b1a6-03ebf9186816,30:f3:1d:32:0b:8f,VIVO Wi-Fi,-88


Each line represents an access point, and each sample collected is grouped by an UUID.

This text file is then transformed to an arff format required by Weka.

To demonstrate this technique we are going to use one Android smartphone to collect the signals strength and map the area. The Android records a wifi_data.txt file with the data separated by comma. The file is saved in the SD Card.

The results in my house using 1200 samples:

# Results

	Correctly Classified Instances     200   96.6184 %
	Incorrectly Classified Instances   7      3.3816 %
	Kappa statistic                             0.96 
	Mean absolute error                       0.0085
	Root mean squared error                   0.0919
	Relative absolute error                   3.9995 %
	Root relative squared error              28.2959 %
	Coverage of cases (0.95 level)           96.6184 %
	Mean rel. region size (0.95 level)       12.5    %
	Total Number of Instances                 207    

	================================================

	Confusion Matrix

	  a  b  c  d  e  f  g  h   <-- classified as
	 42  0  1  0  1  1  0  0 |  a = 1
	  0 28  0  0  0  0  0  0 |  b = 4
	  0  1 35  0  0  0  0  0 |  c = 5
	  0  0  0 23  0  0  1  0 |  d = 3
	  1  0  0  0 15  0  0  0 |  e = 6
	  0  0  0  0  0 35  0  0 |  f = 7
	  0  0  0  1  0  0 22  0 |  g = 2
	  0  0  0  0  0  0  0  0 |  h = none


Here are the results of 4000 collected points in 8 rooms in my University.

	Results
	
	Correctly Classified Instances  240     99.5851 %
	Incorrectly Classified Instances  1      0.4149 %
	Kappa statistic                          0.995
	Mean absolute error                      0.001
	Root mean squared error                  0.0322
	Relative absolute error                  0.4997 %
	Root relative squared error             10.0052 %
	Coverage of cases (0.95 level)          99.5851 %
	Mean rel. region size (0.95 level)      12.5    %
	Total Number of Instances              241 

	Confusion Matrix

	  a  b  c  d  e  f  g  h   <-- classified as
	 21  0  0  0  0  0  0  0 |  a = 7
	  0 37  0  0  0  0  0  0 |  b = 5
	  0  0 64  0  0  0  0  0 |  c = 6
	  0  0  0 38  0  0  0  0 |  d = 3
	  0  0  0  0 12  0  0  0 |  e = 4
	  0  0  0  0  0 27  1  0 |  f = 2
	  0  0  0  0  0  0 41  0 |  g = 1
	  0  0  0  0  0  0  0  0 |  h = none


As the results show, using just Wifi intensity signal can be a good solution for indoor position estimation. 

# Code
The Android app to collect wifi data is [available here](https://github.com/guidefreitas/wifi_locator_android)  

The Java application to convert, train and run cross validation is [available here](https://github.com/guidefreitas/wifi_locator)

To execute the java application use (use Java 1.8):
	
    mvn clean
	mvn compile
	mvn exec:java

# The good
- This technique doesn’t use any special hardware (iBeacons), just an Android smartphone and the existing WiFi infrastructure.
- Simple technique, use just raw Wifi signals, no crazy normalization.

# The bad
- Works only with Android. Apple doesn’t have any public API to get a list of the WiFi networks the iOS device can reach.
- The area need to be mapped upfront. I think this can be achieved with some sort of crowd based information if applied in a environment where there are a lot of people with smartphones (e.g. in a mall).
- Privacy is also a big concern. If and Android app has permition to scan the networks it can use this information to scan networks everywhere, so an app that uses this to provide you useful tips in a store can also collect wifi data when you are at home or work.
