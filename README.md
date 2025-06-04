# **Ical-To-Shedule**
Ical-To-Shedules is a **Node-RED based application** that acts as a converter between Ical url and a Schedule object stored on your **Apex Distech controller**.

- [Ical-To-Shedule](#Ical-To-Shedule)
    - [**1. What is Ical-To-Schedule ?**](#1-what-is-ical-to-schedule-)
        - [**key features](#key-features-)
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
        - [**4.5. Binding configuration**](#45-binding-configuration)









## **1. What is Ical-To-Schedule ?**

**Ical-To-Schedule** is an open source application built on **Node-RED**. It allows you to integrate any timetable inside your controller with their Icalendar link.

### **key features :**

- **Automatic updates :**
   - [x] The Schedule value automatically update every hours, it can be configured to take more or less time to do an update.
   - [x] Don't need to change the link every time the timetable have been modified.

- **Can be used with any timetable :**
   - [x] Ical-To-Schedule can transform every timetable into Schedule BACnet object as long as the link is in Icalendar format.
   - [ ] Cannot use any other format.

- **Can be used with LoRaBAC**
   - [x] If you want to link timetable events to LoRaWAN devices you can use the two addons.
   - [x] Of course you also need to use the [LoRaBAC](https://github.com/SylvainMontagny/LoRaBAC/tree/main) application. 

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

### **4.2. timetable/Schedule Configuration**
For this part you can configure a lot of things :

- **The time zone**
- **the "daily schedule"** : it indicate if all the events of the day need to be regrouped as one or if all the events need to be added to the schedule separately
- **The time offset before start** :  the time in minutes that you want to add before every event.
 - **The time offset before end** :  the time in minutes that you want to remove before the end every event.
- **The remove content array** : every event that contains a string of this array will be ignored and not added to the schedule
- **preview** : the number of day to come from your timetable that you want to add to the schedule
- **pastview** : it's the same as preview but for the past days

```javascript
const timeZone = 'Europe/Paris';
const dailySchedule = false;                 // Group events in 1 event/day
const timeOffsetBeforeStart = 0;            // in min
const timeOffsetBeforeEnd = 0;              // in min
const contentRmEvent = ["TO_CONFIGURE"];   

const preview = 7;
const pastview = 0;
const sorted = false;                       // Events are sorted in chronological order.  
                                            // This helps to reduce algorithm complexity 
                                            // when enable

```

### **4.3. Schedule values Configuration**
This configuration set the values of the schedule :

- **tempOccupied** : the value of the schedule when an event is in progress 
- **tempUnOccupied** : the value of the schedule when no events are in progress
```javascript
const tempOccupied = 20;
const tempUnOccupied = 17;
```

### **4.4. Room Configuration**
The rooms represent each of your timetables, it's an array of objects, these objects have 4 properties :

- **ScheduleID** : It's the instance number that will be given to the Schedule BACnet object.
- **name** : It's the name that will be given to the Schedule BACnet object.
- **device** : It's an array where you need to specify the number of every devices related to this timetable if you use the link and binding addons
- **url** : It's the link to your timetable
```javascript
let rooms = [
    {
        "ScheduleID": 0,
        "name": "@_ScheduleName",
        "devices": [],
        "url": "@_timetableLink",
    }
];
```
### **4.5. Binding configuration**
If you use the link and bindings Addons you will need to configure more things :

- **generateScheduleAnalogValues** : Indicate if you want to create an anlog value related to the Schedule
- **schedulesAnalogValuesOffset** : It's the instance number from which the schedule related analog values storage will start 
- **deviceInstanceOffset** : The Device Offset configured in LoRaBAC
- **deviceInstanceRange** : The Instance range of analog values object of the device configured in LoRaBAC
- **deviceObjectInstanceNum** : The Instance offset of the analog values object of the device you want to bind with the schedule configured in LoRaBAC

| parameter                     | can be used with link addon only | can be used with both addons |
|-------------------------------|----------------------------------|------------------------------|
| `generateScheduleAnalogValues`|                  yes             |              yes             |
| `schedulesAnalogValuesOffset` |                  no              |              yes             |
| `deviceInstanceOffset`        |                  no              |              yes             |
| `deviceInstanceRange`         |                  no              |              yes             |
| `deviceObjectInstanceNum`     |                  no              |              yes             |





```javascript
const generateScheduleAnalogValues = false; //  If enabled will generate an analog value for each schedule
const schedulesAnalogValuesOffset = 10000;  //  The starting instance of theses values
const deviceInstanceOffset = 0; // Device Offset configured in LoRaBAC
const deviceInstanceRange = 10; // Instance range of analog values object of the device configured in LoRaBAC
const deviceObjectInstanceNum = 2; // Instance offset of the analog values object of the device you want to bind with the schedule configured in LoRaBAC

```