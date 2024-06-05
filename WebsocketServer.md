# ZONECRON TIMER INTERFACING

This document describes the message formats sent over websockets to interface a ZonEcron timer or ZonEcron APP.
 - Visit us https://www.zonecron.com
 - Insult us https://facebook.com/zonecron
 - Write us zonecron@zonecron.com

------------------------------------------------------------------------------------------------------------------------

## Contents
- [Context and introduction](#Context-and-introduction)
- [Conecting timer and software](#Conecting-timer-and-software)
- [Message diagram](#Message-diagram)
- [Messages sumary table](#Messages-sumary-table)
- [Admissible messages](#Admissible-messages)
- [Timer to Software Examples](#Timer-to-Software-Examples)
- [Software to Timer Examples](#Software-to-Timer-Examples)
- [Keeping Connection Alive](#keeping-connection-alive)

------------------------------------------------------------------------------------------------------------------------

## Context and introduction
Since its inception in 2019, ZonEcron Timer has featured a WebSocket-based web interface for configuring and operating the timer. As new competition management platforms were developed, the need arose to eliminate human error in transcribing timing results. Additionally, it became imperative to provide the helper team with remote control of the timer integrated with the platform, thus avoiding the need to manage multiple screens simultaneously. In this context, we decided to make public our telegram system between the timer and its web interface, therefor allowing platforms to interface directly with the timer.

------------------------------------------------------------------------------------------------------------------------

## Conecting timer and software

This document will describe the protocol to use when Timer acts as websocket server (running on timer's port 81). 
The timer can also act as a websocket client. In that case, the protocol is slightly different since it was designed to work in conjunction with the [Flow Agility](https://www.flowagility.com/) platform. You can check the details of that protocol in it's repository: https://github.com/flowagility/timer

Once the connection is established, the software should send a message with "d0" to request the update of the timer state.
The timer will respond with a actual status message.
Regardless of whether a "d0" message is sent or not, the timer will send a new message every time an event occurs (start, stop, ...).

------------------------------------------------------------------------------------------------------------------------

## Message diagram
All messages begin with a letter indicating the type of information and have a fixed length of 11 characters except the "d0" update request:

- 1 letter: "g", "i", "s", "o", "p" or "q"
- 1 digit: number of faults
- 1 digit: number of refusals
- 1 digit: eliminated status: not eliminated (0) or eliminated (1)
- 7 digits: elapsed time in milliseconds (so max elapsed will be 9999 s.)

```
i 2 1 0 0036597
│ │ │ │ └──┬──┘
│ │ │ │    └────> 36597 milliseconds
│ │ │ └─────────> not eliminated
│ │ └───────────> 1 refusal
│ └─────────────> 2 faults
└───────────────> timer running
```

------------------------------------------------------------------------------------------------------------------------

## Messages sumary table

| Letter  |   Example   | Meaning                                                               |
|---------|-------------|-----------------------------------------------------------------------|
|    d    |     d0      | Status update request                                                 |
|    g    | g0000312045 | coursewalk running (time left 312 s. -> 5 min. and 12 s.)             |
|    i    | i2100036597 | time running with F+R+E + elapsed                                     |
|    s    | s0000014851 | countdown running (15s. countdown)                                    |
|    o    | o0000420000 | coursewalk stopped (timer showing 7 min ready to start coursewalk)    |
|    p    | p0000000000 | time stopped with F+R+E + elapsed (or reset if all digits == 0 )      |
|    q    | q0000015000 | countdown stopped (timer showing 15s. ready yo start countdown)       |

------------------------------------------------------------------------------------------------------------------------

## Admissible messages 

Depending on the current mode, the timer will not always accept scores or elapsed times. 
For example, in time running ("i") mode, it will accept scores but not elapsed time, and vice versa in coursewalk running ("g") mode.

Depending on the mode in which the timer is, it admits or rejects certain mode changes. 
Examples:
- In time running mode it is not possible to modify the time it is counting or stop it with a specific time, it can only be reseted.
- In coursewalk running mode, any value of faults, refusals or eliminated is discarded.

The reset command is "p0000000000", that is, stopped and with FRE (Faults Refusals Eliminated) and time at zero.

From each mode the following changes are supported: 
- A reset is allowed in any mode.
- From reseted timer, it can go to "g", "s", "o" and "q". (Timer can not be started or stopped with commands, just with its own photocells)
- From each current mode, information from the incoming telegram is admitted according to the following table, the rest of the information or telegrams are discarded: 

| ACTUAL | INCOMING | ADMITTED | CONDITION | ACTIONS                                  |
|--------|----------|----------|-----------|------------------------------------------|
|   g    |     g    |   time   |           | modify remaining coursewalk time         |
|        |     o    |   time   |           | pause coursewalk with indicated time       |
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

------------------------------------------------------------------------------------------------------------------------

## Timer to software examples 
Messages that can be sent by the timer:
  - `i0000002000` - timer running. Actual elapsed time is 2000 ms. -> 2 s. 
  - `p2100028654` - timer stoped. Score is 2 faults, 1 refusals, not eliminated and 28.654 s.
  
------------------------------------------------------------------------------------------------------------------------

## Software to timer examples
Messages that can be received by the timer:
  - `d0` - status request. 
  - `i2100000000` - If timer is running, 2 faults and 1 refusal will be scored on display (time is ignored)
  - `p0010000000` - If timer is stopped, elimination will be scored on display (time is ignored)
  - `o0004200000` - Set or pause coursewalk time
  - `g0004200000` - Start or resume coursewalk time 
  - `p0000000000` - Reset

------------------------------------------------------------------------------------------------------------------------

## Keeping Connection Alive

The timer will use heartbeats to track whether the client is alive according to the RFC 6455 specifications https://www.rfc-editor.org/rfc/rfc6455 (section 5.5.2 and 5.5.3), so the software must comply with these.
The timer will also send a message containing "__ping__" every 5s. for browsers to keep track of server alive. Use it at convenience.
