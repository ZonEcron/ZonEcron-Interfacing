# ZonEcron Timer Serial Communication Commands

## Description

The ZonEcron timer provides a versatile serial interface for external control and monitoring, designed to offer maximum compatibility with a wide range of systems, despite the fact that serial communication is often considered outdated in modern applications. This interface allows you to send commands and receive data from the timer, ensuring precise control over its operation. Communication should be established at a baud rate of 5600 with 8 data bits, no parity, and 1 stop bit (8N1). These settings are crucial for ensuring that data is accurately transmitted and received between devices.

To facilitate this communication, a [ZonEcron dongle](https://www.zonecron.com/ecosistema/visualizacion/mochila) is required. This dongle connects to your computer via USB and is recognized as a serial port by the operating system. Once connected, you can use any standard serial communication software to interact with the timer. The dongle then communicates wirelessly with the timer using a proprietary protocol, ensuring robust and reliable data transmission even in challenging environments.

This dongle will communicate with the large display and/or the cells directly. In case the large display is not available, this accessory is the only way to interact with the timer, as the web server enabling other forms of communication resides in the large display. Therefore, using this dongle provides basic connectivity and control with or without the large display unit.

---

## Contents
- [1. Connecting Timer and Software](#1-connecting-timer-and-software)
- [2. Keeping Connection Alive](#2-keeping-connection-alive)
- [3. Message Diagram](#3-message-diagram)
- [4. Mode Meanings and Examples](#4-mode-meanings-and-examples)
- [5. Message Exchange](#5-message-exchange)
- [6. Timer Accepted Messages](#6-timer-accepted-messages)

---

## 1. Connecting Timer and Software

The ZonEcron timer's serial communication should be established using a baud rate of 56000 in older versions or 38400 from version 2.7.0 and newer. Use 8 data bits, no parity, and 1 stop bit (8N1). Once the dongle is connected via USB and recognized as a serial port by the operating system, any serial communication software can be used to interact with the timer.

---

## 2. Keeping Connection Alive

The ZonEcron dongle ensures robust and reliable data transmission even in challenging environments. The dongle communicates wirelessly with the timer using a proprietary protocol, ensuring that the connection remains stable. There are no special requirements other than ensuring that the software keeps the serial port open and listening.

---

## 3. Message Diagram

The ZonEcron timer's serial communication messages consist of a command and, in some cases, a parameter. If a parameter is sent, the command and parameter must be separated by one space character: 
- **Command:** A keyword indicating the action to be performed.
- **Parameters:** Additional information required for the command, if any.

Each message must be terminated with a 0x0A - "\n" - line feed character. 
0x0D - "\r" - carriage return characters and spaces after the command or parameter, if present, will be ignored.

For readability purposes, the terminating line feed character will not be shown in the examples in this manual.

Example of command with parameter:
`DATA 2:1:0`: Scores 2 faults, 1 refusal, and not eliminated.

---

## 4. Message Meanings and Examples

The ZonEcron timer can switch between different modes using specific commands:

| Command | Parameter | Optional | Meaning                                                      |
|---------|-----------|----------|--------------------------------------------------------------|
| STATUS  |     -     |    -     | Request to update the current state of the timer.            |
| START   |   time    |    no    | Starts the timer with the specified elapsed time.            |
| STOP    |   time    |    no    | Stops the timer with the specified elapsed time.             |
| DATA    |  F:R:E    |    no    | Scores faults, refusals, and elimination flag.               |
| RESET   |     -     |    -     | Resets the timer.                                            |
| WALK    |  seconds  |    yes   | Sets the timer in coursewalk mode.                           |
| DOWN    |  seconds  |    yes   | Sets the timer in countdown mode.                            |
| FAIL    |     -     |    -     | Notification indicating a cell failure.                      |
| OK      |     -     |    -     | Notification indicating the resolution of a cell failure.    |

Examples:

- `START 3500`: Starts the timer with an elapsed time of 3500ms (3.5s).
- `STOP 36748`: Stops the timer with an elapsed time of 36748ms (36.74s).
- `DATA 2:1:0`: Sends 2 faults, 1 refusal, and not eliminated.
- `RESET`: Sets time, faults, refusals, and elimination to 0 and stops the timer.
- `WALK`: Initiates course walk for 7 minutes.
- `WALK 360`: Initiates course walk for 360 seconds (6 minutes).
- `DOWN`: Initiates a 15-second countdown for the guide to start the course.
- `DOWN 20`: Initiates a 20-second countdown for the guide to start the course.
- `FAIL`: Indicates that a photocell is in alarm. This message will repeat every 5 seconds until the alarm is corrected.
- `OK`: Clears the FAIL state, indicating that all cells are okay.

---

## 5. Message Exchange

In the ZonEcron timer system, communication between the timer and the software involves exchanging specific commands. These commands control the timer's functions and provide status updates. Below are the messages that can be sent and received:

Timer can send this commands to software: `START`, `STOP`, `DATA`, `RESET`, `WALK`, `DOWN`, `FAIL` and `OK`.

Software can send this commands to timer: `STATUS`, `DATA`, `RESET`, `WALK` and `DOWN`.

---

## 6. Timer Accepted Messages

Depending on the timer's current mode, it may not always accept scores or elapsed times. For example:
- If the timer is started (`START`), score commands (`DATA`) are accepted but not course walk (`WALK`) commands. It is not possible to modify the counting time (`START`) or stop the timer with a specific time (`STOP`).
- If the timer is in course walk mode (`WALK`), any values for faults, refusals, or eliminations (`DATA`) will be discarded. Only the elapsed time modifications with (`WALK`) are taken into account.

Allowed messages and changes in the timer's current mode are as follows:
- A status request and reset command are allowed in any mode. The reset (`RESET`) command stops the timer and sets the faults, refusals, eliminations, and time to zero.
- The timer cannot be started or stopped with commands (except for reset); it relies on its own photocells.
- For each current mode, the timer accepts incoming command information according to the following table, other commands are rejected:

| ACTUAL | INCOMING | ADMITTED | CONDITION | ACTIONS                                  |
|---|---|---|---|---|
| RESET  |  DATA    |  F-R-E   |    r>0    | timer starts and ignores first detection |
|        |  WALK    |  time    |           | modify remaining coursewalk time         |
|        |  DOWN    |  time    |           | start countdown with indicated time      |
|        |          |          |           |                                          |
| START  |  DATA    |  F-R-E   |           | score F-R-E                              |
|        |          |          |           |                                          |
| STOP   |  DATA    |  F-R-E   |           | score F-R-E                              |
|        |  DOWN    |  time    |           | start countdown with indicated time      |
|        |          |          |           |                                          |
| WALK   |  WALK    |  time    |           | modify remaining coursewalk time         |
|        |          |          |           |                                          |
| DOWN   |  DATA    |  F-R-E   |    r>0    | timer starts and ignores first detection |

---

