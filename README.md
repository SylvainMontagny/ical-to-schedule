# **Ical-To-Schedule**
Ical-To-Schedules is a **Node-RED based application** that acts as a converter between Ical url and a Schedule object stored on your **Apex Distech controller**.

- [**Ical-To-Schedule**](#Ical-To-Schedule)
    - [**1. What is Ical-To-Schedule ?**](#1-what-is-ical-to-schedule-)
        - [**key features**](#key-features-)
    - [**2. Prerequisites**](#2-prerequisites)
        - [**2.1. Controller**](#21-controller)
        - [**2.2. Node-RED setup**](#22-node-red-setup)
    - [**3. Getting started**](#3-getting-started)
        - [**Ical-To-Schedule Configuration**](#ical-to-schedule-configuration)
    - [**4. Configuration**](#4-configuration)
        - [**4.1. Controller Configuration :**](#41-controller-configuration-)
        - [**4.2. timetable/Schedule Configuration**](#42-timetableschedule-configuration)
        - [**4.3. Schedule values Configuration**](#43-schedule-values-configuration)
        - [**4.4. Room Configuration**](#44-room-configuration)
        - [**4.5. Binding configuration**](#45-link-configuration)
        - [**4.6. Binding configuration**](#46-binding-configuration)
    - [**6. Test With a generated Ical**](#6-test-with-a-generated-ical)


## **1. What is Ical-To-Schedule ?**

**Ical-To-Schedule** is an open source application built on **Node-RED**. It allows you to integrate any timetable inside your controller with their Icalendar url.

### **key features :**

- **Automatic updates**
  * [x] The Schedule value automatically update every hours, it can be configured to take more or less time to do an update
  * [x] Don't need to change the url every time the timetable have been modified

- **Automatic erases**
  * [x] The Schedule that are not used anymore (not in the room list) will be automatically erased
  
- **Can be used with any timetable**
  * [x] Ical-To-Schedule can transform every timetable into Schedule BACnet object as long as the url is in Icalendar format
  * [ ] Cannot use any other format

- **Can be used with LoRaBAC**
  * [x] If you want to link timetable events to LoRaWAN devices you can use the two addons
  * [x] Of course you also need to use the [LoRaBAC](https://github.com/SylvainMontagny/LoRaBAC/tree/main) application

## **2. Prerequisites**

### **2.1. Controller**
**Ical-To-Schedule** use the controller rest API protocol to create and update the schedules, acctually the only implemented API is the **`Distech Controls v2`**.

### **2.2. Node-RED setup**
**Ical-To-Schedule** is a Node-RED flow, so you need a **Node-RED instance** to run it. To work correctly the flow will need the following packages :

- **node-red-contrib-ical-events**
- **@ba47/node-red-queue**

If you install Node-RED from docker you can use the [**montagny/node-red**](https://hub.docker.com/r/montagny/node-red) image where all the packages are installed.

## **3. Getting started**
### **Ical-To-Schedule Configuration**
1. **Import Ical-To-Schedule:**
    * Open Node-RED and go to `Menu > Import`
    * Select the Ical-To-Schedule.json file from your computer or just copy/paste the file content from GitHub

2. **Configure the application :** 
    * In Node-RED locate and open the `TO CONFIGURE` node (at the top left).

## **4. Configuration**
### **4.1. Controller Configuration :**
The controller configuration is almost the most simple part. You need the IP address, the login and the password of the controller.  

```javascript
const controllerIP = "@_ControllerIP";  // Controller IP Address
const controllerLogin = "@_ControllerLogin";  //Apex distech login
const controllerPassword = "@_ControllerPassword";  //Apex distech password
```

### **4.2. Timetable/Schedule Configuration**
For this part you can configure a lot of things :

- **The time zone**
- **the "daily schedule"** : it indicate if all the events of the day need to be regrouped as one or if all the events need to be added to the schedule separately
- **The default time offset before start** :  the default time in minutes that you want to add before every event.
 - **The default time offset before end** :  the default time in minutes that you want to remove before the end every event.
- **The remove content array** : every event that contains a string of this array will be ignored and not added to the schedule
- **preview** : the number of day to come from your timetable that you want to add to the schedule
- **pastview** : it's the same as preview but for the past days
- **scheduleRange** : it's the range in which the schedules can be created. A schedule created outside this range will genertate an error
- **sorted** : it indicate if the events are sorted in chronological order, this helps to reduce algorithm complexity when enabled

```javascript
const timeZone = 'Europe/Paris';
const dailySchedule = false;                 // Group events in 1 event/day
const defaultTimeOffsetBeforeStart = 0;            // in min
const defaultTimeOffsetBeforeEnd = 0;              // in min
const contentRmEvent = ["TO_CONFIGURE"];   

const preview = 7;
const pastview = 0;
const sorted = false;                       


const scheduleRange = [0,100];
```

### **4.3. Schedule default values Configuration**
This configuration set the values of the schedule :

- **defaultTempOccupied** : the default value of the schedule when an event is in progress 
- **defaultTempUnOccupied** : the default value of the schedule when no events are in progress
```javascript
const tempOccupied = 20;
const tempUnOccupied = 17;
```

### **4.4. Room Configuration**
The rooms represent each of your timetables, it's an array of objects, these objects have 4 properties :

- **isActive** : It indicate if the room is active or not, if it's not active the schedule will not be created or even erased if it already exist.
- **ScheduleID** : It's the instance number that will be given to the Schedule BACnet object.
- **name** : It's the name that will be given to the Schedule BACnet object.
- **device** : It's an object of arrays, if you use the link and binding addons you need to specify the number of every devices related to this timetable for each device type 
- **tempOccupied** : the value of the schedule when an event is in progress
- **tempUnOccupied** : the value of the schedule when no events are in progress
- **timeOffsetBeforeStart** :  the time in minutes that you want to add before every event.
- **timeOffsetBeforeEnd** :  the time in minutes that you want to remove before the end every event.
- **url** : It's the link to your timetable
```javascript
let rooms = [
    {
        "isActive": true,
        "ScheduleID": 0,
        "name": "@_ScheduleName",
        "devices": { "@_deviceType1" :[], "@_deviceType2" :[]},
        "tempOccupied": defaultTempOccupied,
        "tempUnOccupied": defaultTempUnOccupied,
        "timeOffsetBeforeStart" : defaultTimeOffsetBeforeStart,
        "timeOffsetBeforeEnd" : defaultTimeOffsetBeforeEnd,
        "url": "@_timetableLink",
    }
];
```
### **4.5. Link Configuration**
If you use the link Addon you will need to configure more things :

- **generateScheduleAnalogValues** : Indicate if you want to create an anlog value related to the Schedule
- **schedulesAnalogValuesOffset** : It's the instance number from which the schedule related analog values storage will start 

```javascript
const createScheduleAV = false; //  If enabled will generate an analog value for each schedule
const schedulesAVOffset = 10000;  //  The starting instance of theses values

```

### **4.6. Binding Configuration**
If you use the link and binding Addons you will need to configure a device list, this list is almost the same as the one used in LoRaBAC :

- **maxDevNum** : The Device maximum number configured in LoRaBAC
- **offsetAV** : The Device Offset for the analog values configured in LoRaBAC
- **instanceRangeAV** : The Instance range of analog values object of the device configured in LoRaBAC
- **instanceNum** : The Instance offset of the device analog values object that you want to bind with the schedule value

```javascript
const deviceList = {
    "@_deviceType1" : {
        "identity": {
            "maxDevNum": 10 <===
        },
        "bacnet": {
            "offsetAV": 0,  <===
            "instanceRangeAV": 10,  <===  
            "objects": {
                "controllerSetpoint": {
                    "instanceNum": 2,   <===
                }
            }
        },
    },
    "@_deviceType2": {
        "identity": {
            "maxDevNum": 10
        },

        etc...

```
## **5. Device list and room list checks**

- [x] Check if each room have all the mandatory properties
- [x] Check if each room have a different schedule instance
- [x] Check if each device have all the mandatory properties
- [x] Check if each device have a different name



## **6. Test With a generated Ical**
To test the application you can use the Ical generator to create a test ical file. To do that you need to :

1. Download the Ical generator from the Github.
2. Generate the file.
3. import the file in any application that provide an ical link (**ex.**Google).
4. Copy the link inside the Ical-To-Schedule configuration
5. Check if everything works as you want 