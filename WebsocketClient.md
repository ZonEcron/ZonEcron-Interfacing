# ZONECRON TIMER AS WEBSOCKET CLIENT

## Description

The timer can work as a WebSocket client. Therefore, the software will need to set up a WebSocket server.

Working as websocket client, the messages used, are basically the same used when the timer is a websocket server but with some details that must be taken into account. We recommend to read the documentation carefully

This variation is due to the feature being designed to work in conjunction with the [Flow Agility](https://www.flowagility.com/) software. You can check [their repository](https://github.com/flowagility/timer), where you will find information similar to what is provided here.

----------------------------------------------------------------------------------------------------

## Contents
- [1. Connecting Timer and Software](#1-Connecting-Timer-and-Software)
- [2. Keeping Connection Alive](#2-Keeping-connection-alive)
- [3. Message diagram](#3-Message-Diagram)
- [4. Mode Meanings and Examples](#4-Mode-Meanings-and-Examples)
- [5. Message Exchange](#5-Message-Exchange)
- [6. Timer Accepted Messages](#6-Timer-Accepted-Messages)
- [7. Timer to Software Examples](#7-Timer-to-Software-Examples)
- [8. Software to Timer Examples](#8-Software-to-Timer-Examples)

----------------------------------------------------------------------------------------------------

## 1. Connecting Timer and Software

To connect the timer to the software that will act as the WebSocket server, you need to configure the timer through its web interface. In that interface adrres, path and port can be configured as well as type of server

To connect the timer to a software, the software should be set into a listening mode, by providing the Mac Address of a timer. This can be done inside of the run views. After the timer Mac Address is entered, the software is waiting for the timer to connect. Timer should then initiate connection to a specific URL (https://flowagility.com/ws/timer/mac_address - where mac_address is a Mac Address of the timer). 

Timer should try to connect to the software with series of 5, 10, 30 and 60 seconds intervals (3-5 attempts in a series).

As soon as timer is connected, the software is ready to receive signals from the timer.

Eveytime the user is loading the page which is able to track timer connection, the software is sending a status request to the timer and is awaiting for the response with a current state. 

----------------------------------------------------------------------------------------------------

## 2. Keeping Connection Alive

The timer is capable of responding to heartbeats from the server in accordance with RFC 6455 specifications (sections 5.5.2 and 5.5.3), allowing the software to monitor the status of the timer client's connection. 
Additionally, the timer will reply to messages containing 'ping' with a corresponding 'pong' message, enabling the software to implement custom heartbeat management if necessary.

----------------------------------------------------------------------------------------------------

## 3. Message Diagram

All messages begin with a letter indicating the type of information and have a fixed length of 11 characters except for the "d0" update request:

- 1 letter: "g", "i", "s", "o", "p", or "q"
- 1 digit: number of faults
- 1 digit: number of refusals
- 1 digit: elimination status: not eliminated (0) or eliminated (1)
- 7 digits: elapsed time in milliseconds (so the maximum elapsed time will be 9999 seconds).

```text
i 1 2 0 0031841
│ │ │ │ └──┬──┘
│ │ │ │    └────> 31841 milliseconds
│ │ │ └─────────> not eliminated
│ │ └───────────> 2 refusal
│ └─────────────> 1 fault
└───────────────> timer running
```

----------------------------------------------------------------------------------------------------

## 4. Mode Meanings and Examples

These are the modes the timer can be in, or can be asked to switch to:

- "g": Course walk countdown running.
- "i": Timer running.
- "p": Timer stopped.
- "s": Course start countdown running.
- "o": Course walk countdown stopped.
- "q": Course start countdown stopped.

"p" and "i" are the normal modes, which the timer is in most of the time.

"o" and "g" are course walk modes. Course walk is the time allocated for handlers to inspect the course layout and plan their strategy. The default time is 7 minutes (420 seconds).

"s" and "q" are course start modes. Course start is the maximum time for handlers to begin the course once the judge signals. The default time is 15 seconds. This is not commonly used, typically reserved for international competitions like the EO (European Open) or the AWC (Agility World Championship).

Below, there is an example of a message for each mode along with its meaning:

| Letter  |   Example   | Meaning                                                               |
|---------|-------------|-----------------------------------------------------------------------|
|    d    |     d0      | Status update request                                                 |
|    g    | g0000312045 | Course walk countdown running, time left: 312 s. = 5 min. 12 s.       |
|    i    | i2100023218 | Timer running with 2 faults, 1 refusal, not Eliminated, and 23.218 s. |
|    s    | s0000008451 | Course start countdown running, time left: 8.451 s.                   |
|    o    | o0000420000 | Course walk countdown stopped, ready to start with 420 s. -> 7 min.   |
|    p    | p0000000000 | Timer stopped, 0 faults, 0 refusals, 0 elim, and 0 time -> reset      |
|    q    | q0000015000 | Course start countdown stopped, ready to start with 15 seconds        |

----------------------------------------------------------------------------------------------------

## 5. Message Exchange

Both the timer and the software can exchange messages with each other. Upon receiving a message, the recipient must respond with the same message, prefixed with "#" if it has been accepted, or with a message indicating its current mode, prefixed with "!".

For example, when the software sends information about faults, refusals, or eliminations to the timer, the timer should respond with the same F+R+E, followed by the current time in milliseconds, prefixed with "#" if it has been accepted, or with a message indicating its current mode, prefixed with "!".

----------------------------------------------------------------------------------------------------

## 6. Timer Accepted Messages

Depending on the timer's current mode, it may not always accept scores or elapsed times. For example:
- In "time running" mode "i", it will accept scores but not elapsed time. It is not possible to modify the counting time or stop the timer with a specific time.
- In "coursewalk running" mode "g", any values for faults, refusals, or eliminations are discarded and just elapsed time is taken into account.

Allowed messages and changes in the timer's current mode are as follows:
- A reset is allowed in any mode. The reset command is "p0000000000", which stops the timer and resets the faults, refusals, eliminations, and time to zero.
- From a reset state, the timer can transition to "g", "s", "o", and "q". (The timer cannot be started or stopped with commands; it relies on its own photocells.)
- For each current mode, the timer accepts incoming telegram information according to the following table, other information or telegrams are rejected:

| ACTUAL | INCOMING | ADMITTED | CONDITION | ACTIONS                                  |
|--------|----------|----------|-----------|------------------------------------------|
|   g    |     g    |   time   |           | modify remaining coursewalk time         |
|        |     o    |   time   |           | pause coursewalk with indicated time     |
|        |          |          |           |                                          |
|   i    |     i    |  F-R-E   |           | score F-R-E                              |
|        |          |          |           |                                          |
|   s    |     s    |  F-R-E   |           | score F-R-E                              |
|        |          |          |    r>0    | timer starts and ignores first detection |
|        |          |          |           |                                          |
|   o    |     g    |   time   |           | start coursewalk with indicated time     |
|        |     o    |   time   |           | modify coursewalk time                   |
|        |          |          |           |                                          |
|   p    |     q    |   time   |           | ready to countdown with indicated time   |
|        |     p    |  F-R-E   |           | score F-R-E                              |
|        |          |          | t=0 & r>0 | timer starts and ignores first detection |
|        |          |          |           |                                          |
|   q    |     s    |   mode   |           | start countdown                          |
|        |     q    |   time   |           | modify countdown time                    |
|        |     q    |  F-R-E   |           | score F-R-E                              |
|        |          |          |    r>0    | timer starts and ignores first detection |

----------------------------------------------------------------------------------------------------

## 7. Timer to software examples 
Messages that can be sent by the timer:
  - `i0000002000` - timer running. Actual elapsed time is 2000 ms. -> 2 s. 
  - `#i2100036597` - response with infromation shown on the running timer at the moment of the request
  - `p2100028654` - timer stoped. Score is 2 faults, 1 refusals, not eliminated and 28.654 s.
  - `ping` - ping message to a software, that should receive `pong` as a response
  
----------------------------------------------------------------------------------------------------

## 8. Software to timer examples
Messages that can be received by the timer:
  - `d0` - status request. Response should be pre-pended by `#` symbol. For example, if the timer is running, then response should be `#i2100036597`
  - `i2100000000` - If timer is running, 2 faults and 1 refusal will be scored on display (time is ignored)
  - `#i2100000000` - response with infromation shown on the running timer at the moment of the request
  - `p0010000000` - If timer is stopped, elimination will be scored on display (time is ignored)
  - `o0004200000` - Set or pause coursewalk time
  - `g0004200000` - Start or resume coursewalk time 
  - `p0000000000` - Reset
  - `pong` - response to `ping` command from Timer
 
