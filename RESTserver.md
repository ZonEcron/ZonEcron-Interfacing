# ZONECRON TIMER AS REST SERVER

The ZonEcron web server provides a comprehensive suite of Application Programming Interfaces (APIs) designed to facilitate seamless interaction and real-time monitoring of its functionality. Within the context of this guide, we'll assume the timer is configured with the IP address 192.168.1.100. Through these APIs, users can remotely access and manipulate various aspects of the timer's behavior, such as initiating countdowns, scoring faults and refusals, or resetting the timer entirely. These APIs empower users to streamline competition management processes and enhance overall efficiency by offering direct control and access to critical timer functions.

It's important to note that, to maintain optimal server performance, it's recommended to limit API requests to **no more than one per second**. This ensures that the server remains responsive and prevents potential saturation. Through the examples outlined below, users can gain insights into the versatility and utility of these APIs, leveraging them to tailor the timer's operation to their specific requirements and workflows.

## Contents
- [1. Timer Actions](#1-timer-actions)
- [2. GET XML](#2-get-xml)
- [3. GET JSON](#3-get-json)

---

## 1. Timer Actions

1. **Update Timer with Faults, Refusals, and Eliminations:**
   - Send an HTTP PUT request to `http://192.168.1.100/action` with parameters `faults`, `refusals`, and `eliminated` to update the timer's score. For example:
     ```
     PUT /action?faults=2&refusals=1&eliminated=0 HTTP/1.1
     ```

2. **Start the Countdown Mode:**
   - Send an HTTP PUT request to `http://192.168.1.100/action` with the `countdown` parameter to initiate the countdown mode of the timer. The value of this parameter represents the duration of the countdown in milliseconds. If no value is specified or the value is 0, the default countdown will be 15000 milliseconds (15 seconds). For example, to start a countdown of 20 seconds:
     ```
     PUT /action?countdown=20000 HTTP/1.1
     ```

3. **Start the Coursewalk Mode:**
   - Send an HTTP PUT request to `http://192.168.1.100/action` with the `coursewalk` parameter to initiate the coursewalk mode of the timer. The value of this parameter represents the duration of the coursewalk in milliseconds. If no value is specified or the value is 0, the default coursewalk will be 420000 milliseconds (420 seconds or 7 minutes). For example, to start a coursewalk of 8 minutes:
     ```
     PUT /action?coursewalk=48000 HTTP/1.1
     ```

4. **Reset the Timer:**
   - Send an HTTP PUT request to `http://192.168.1.100/action` with the `reset` parameter to reset the timer. This will reset all values of the timer, including time, faults, refusals, eliminations, etc. For example:
     ```
     PUT /action?reset=1 HTTP/1.1
     ```

---

## 2. GET XML

You can send an HTTP GET request to http://192.168.1.100/xml to retrieve the current state of the timer in XML format.
You will receive a response containing the following elements:

- `timerID`: The unique identifier of the timer.
- `time`: The current elapsed time.
- `running`: Indicates whether the timer is running (`1`) or stopped (`0`).
- `countdown`: Indicates whether the timer is in countdown mode (`1`) or not (`0`).
- `coursewalk`: Indicates whether the timer is in coursewalk mode (`1`) or not (`0`).
- `faults`: The number of scored faults.
- `refusals`: The number of scored refusals.
- `eliminated`: Indicates whether the team has been eliminated (`1`) or not (`0`).

Example of response of timer running with elapsed time 21.5 secconds and 2 faults:
```xml
<?xml version='1.0' encoding='UTF-8'?>
<xml>
    <timerID>00:1A:22:XX:XX:XX</timerID>
    <time>21500</time>
    <running>1</running>
    <countdown>0</countdown>
    <coursewalk>0</coursewalk>
    <faults>2</faults>
    <refusals>0</refusals>
    <eliminated>0</eliminated>
</xml>
```

---

## 3. GET JSON

You can send an HTTP GET request to http://192.168.1.100/json to retrieve the current state of the timer in JSON format.
You will receive a response containing the following fields:

- `timerID`: The unique identifier MAC address of the timer.
- `time`: The current elapsed time.
- `running`: Indicates whether the timer is running (`true`) or stopped (`false`).
- `countdown`: Indicates whether the timer is in countdown mode (`true`) or not (`false`).
- `coursewalk`: Indicates whether the timer is in coursewalk mode (`true`) or not (`false`).
- `faults`: The number of scored faults.
- `refusals`: The number of scored refusals.
- `eliminated`: Indicates whether the team has been eliminated (`true`) or not (`false`).


Example of response of timer running with eladsed time 21.5 secconds and 2 faults:
```json
{
    "timerID": "00:1A:22:XX:XX:XX",
    "time": 21500,
    "running": true,
    "countdown": false,
    "coursewalk": false,
    "faults": 2,
    "refusals": 0,
    "eliminated": false
}
```