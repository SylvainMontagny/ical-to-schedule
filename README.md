# **Ical-To-Schedule** : control your building temperature
`ical-To-Schedule` is a **Node-RED based application** used to control temperature heating system in a Building Management System. It works with Distech-Controls controllers.
1. it gets an ical file from any ical server : outlook, google calendar, ADE (used by USMB)...
2. it creates BACnet schedule objects based on the content of the ical file
3. it connects the output of the schedule object to a batch of analog value, usually the setpoint of thermostatic valves to be controlled. 



- [**Ical-To-Schedule** : control your building temperature](#ical-to-schedule--control-your-building-temperature)
  - [**1. Prerequisites**](#1-prerequisites)
    - [**1.1. Controller**](#11-controller)
    - [**1.2. Node-RED setup**](#12-node-red-setup)
    - [**1.3. LoRaBAC**](#13-lorabac)
  - [**2. Getting started**](#2-getting-started)
    - [**2.1. Ical-To-Schedule Import**](#21-ical-to-schedule-import)
    - [**2.2. Application configuration**](#22-application-configuration)
      - [**2.2.1. General configuration**](#221-general-configuration)
      - [**2.2.2. Room configuration:**](#222-room-configuration)
  - [**3. Test With a generated Ical**](#3-test-with-a-generated-ical)



## **1. Prerequisites**
### **1.1. Controller**
**Ical-To-Schedule** uses the **`Distech Controls v2`** HTTP Rest API. It only works with a Distech-Controls controller.

### **1.2. Node-RED setup**
**Ical-To-Schedule** is a Node-RED flow, which means it requires a Node-RED instance to operate. For proper functionality, the flow depends on the following additional Node-RED packages:
- **node-red-contrib-ical-events**
If you install Node-RED from docker you can use the [**montagny/node-red**](https://hub.docker.com/r/montagny/node-red) image where all the packages are installed.

### **1.3. LoRaBAC**
This application needs to get information from the LoRaBAC application, so you need a [LoRaBAC](https://github.com/SylvainMontagny/LoRaBAC) configuration ready and running in your Node-RED global environment.


## **2. Getting started**
### **2.1. Ical-To-Schedule Import**
* Open Node-RED and go to `Menu > Import`
* Select the `Ical-To-Schedule.json` file from this repository.

### **2.2. Application configuration**
#### **2.2.1. General configuration**
The following elements can be configured :
* **printMissingDevice** : Print on the debug panel all missing devices based on the name of the LoRaWAN devices. [true, false]
* **defaultControl** : defines the default control strategy for the thermostatic valves. ["controlledByEventFromIcal", "controlledBySchedule", "controlledByConstant", "inactive"]

Configuration to create BACnet schedule :
* **timeBetweenScheduleUpdate**: time (in seconds) between two updates of BACnet schedule object from the ical file.
* **scheduleInstanceRange**: schedule instance range where schedule objects will be stored. ie : [0, 50];
* **groupEvents**: choose whether we want to group events during the day to make only one. [true, false]
* **timeZone**: Time zone. ie : 'Europe/Paris'
* **eventsToDiscard**: Events that needs to be discarded based on their description. ie: ["SOBRIETE ENERGETIQUE"]
* **defaultTempOccupied**: Occupied default temperature used if not mentionned in the room configuration.
* **defaultTempUnoccupied**: Unoccupied default temperature used if not mentionned in the room configuration.
* **defaultTimeOffsetBeforeStart**: Heating anticipation start time if not mentionned in the room configuration. (mins)
* **defaultTimeOffsetBeforeEnd**: Heating anticipation stop time if not mentionned in the room configuration. (mins)
* **defaultNbrDaysPreview**:  Number of days preloaded with events in schedule BACnet object if not mentionned in the room configuration.
* **defaultAddSuffixToAdeURL**: use this option when using ADE calendar [true, false]
* **minimumSlotDuration**: Event below this time will be discarded.
* **defaultWeekly**: Weekly schedule applied by default if not mentionned in the room configuration.

Configuration to create Analog value for each schedule
* **scheduleAVInstanceOffset**: Starting instance number for the analog value associated with each schedule.
* **timeBetweenScheduleAVUpdate**: time (in seconds) between two updates from the schedule present value and the Analog Value associated with this schedule.

#### **2.2.2. Room configuration:**
In the TO CONFIGURE node, you will see a table named room that represents the room configuration.
```javascript
let rooms = [];
```
For each room, you need to provide mandatory and optionnal configuration:
```javascript
let rooms = [
{
    // Mandatory //
    "scheduleName": "roomName0",
    "devices": { "device-1": { "deviceNums": [1, 2, 3], "AVToBindName": "controllerSetpoint" } },
    "url": 'https://ade-usmb-ro.grenet.fr/jsp/custom/modules/plannings/direct_cal.jsp?data=b5cfb898a9c27be94975c12c6eb30e9233bdfae22c1b32e2cd88eb944acf5364c69e3e5921f4a6ebe36e93ea9658a08f,1&resources=1480&projectId=5&calType=ical&lastDate=2042-08-14',
    
    // Optionnal, if not specified the default value will be applied //
    "control": defaultControl,
    "nbrDaysPreview" : defaultNbrDaysPreview,
    "tempOccupied": defaultTempOccupied,
    "tempUnoccupied": defaultTempUnoccupied,
    "timeOffsetBeforeStart": defaultTimeOffsetBeforeStart,
    "timeOffsetBeforeEnd": defaultTimeOffsetBeforeEnd,
    "addSuffixToAdeURL" : defaultAddSuffixToAdeURL,
}
];
```

## **3. Test With a generated Ical**
To test the application you can use the [Ical generator](https://github.com/SylvainMontagny/ical-to-schedule/tree/main/Ical_Generator) from this repository to create a test ical file.