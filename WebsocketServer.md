# ZONECRON TIMER AS WEBSOCKET SERVER

## Description

The timer can work as a WebSocket server. Therefore, the software will need to act as websocket client.

Working as websocket server, the messages used, are basically the same used when the timer is a websocket client but with some details that must be taken into account. We recommend to read the documentation carefully

----------------------------------------------------------------------------------------------------

## Contents
- [1. Connecting Timer and Software](#1-Connecting-Timer-and-Software)
- [2. Keeping Connection Alive](#2-Keeping-connection-alive)
- [3. Message Diagram](#3-Message-Diagram)
- [4. Mode Meanings and Examples](#4-Mode-Meanings-and-Examples)
- [5. Message Exchange](#5-Message-Exchange)
- [6. Timer Accepted Messages](#6-Timer-Accepted-Messages)
- [7. Timer to Software Examples](#7-Timer-to-Software-Examples)
- [8. Software to Timer Examples](#8-Software-to-Timer-Examples)

----------------------------------------------------------------------------------------------------

## 1. Connecting Timer and Software

The timer operates a WebSocket server on port 81. This connection is not encrypted as it is intended for local network communication, where network security is ensured and encryption is not deemed necessary. This setup is ideal for applications requiring low latency and fast communication within the same network.

Once the connection is established, the software should send a message with "d0" to request the timer's status update. The timer will respond with the current status message. Regardless of whether a "d0" message is sent or not, the timer will send a new message whenever an event occurs (start, stop, coursewalk runout, etc.).

Below is an example of JavaScript code that can be run from an HTML file to connect to the timer, assuming it has the IP address 192.168.1.100:

```javascript
const socket = new WebSocket('ws://192.168.1.100:81');    // Create a new instance of WebSocket

socket.onopen = function(event) {                         // Event when the connection opens
    console.log('Connected to timer.');
    socket.send('d0');                                    // Request timer status update
};

socket.onmessage = function(event) {                      // Event when a message is received
    console.log('Received: ', event.data);
};
```

----------------------------------------------------------------------------------------------------

## 2. Keeping Connection Alive

The timer will use heartbeats to track whether the client is alive according to the RFC 6455 specifications (see [RFC 6455](https://www.rfc-editor.org/rfc/rfc6455) sections 5.5.2 and 5.5.3).  
Additionally, the timer will send a message containing "__ping__" every 5 seconds for browsers to keep track of the server's status. Use it as needed.


----------------------------------------------------------------------------------------------------

## 3. Message Diagram

All messages begin with a letter indicating the type of information and have a fixed length of 11 characters except for the "d0" update request:

- 1 letter: "g", "i", "s", "o", "p", or "q"
- 1 digit: number of faults
- 1 digit: number of refusals
- 1 digit: elimination status: not eliminated (0) or eliminated (1)
- 7 digits: elapsed time in milliseconds (the maximum elapsed time is 9999 seconds).

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

Both the timer and the software can exchange messages with each other. Confirmation messages are not required, as communication relies on WebSocket acknowledge protocols. When the software sends a message to the timer, it will broadcast back its updated status to all clients, which can include other software or web clients in browsers, as the timer can be operated through a web server.

For instance, if the timer is running ("i" mode), when the software sends information about faults, refusals, or eliminations to the timer, the timer will broadcast back the same faults, refusals, and eliminations, but it will ignore any received time and send the message with the current time.

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

## 7. Timer to Software Examples 
Messages that can be sent by the timer:
  - `i0000002000` - Timer running. Actual elapsed time is 2000 ms -> 2 seconds.
  - `p2100028654` - Timer stopped. Score is 2 faults, 1 refusal, not eliminated, and 28.654 seconds.
  - `p0000000000` - Reset (e.g., after course walk countdown runs out).
  - `__ping__` - Ping message to all clients every 5 seconds.

----------------------------------------------------------------------------------------------------

## 8. Software to Timer Examples
Messages that can be received by the timer:
  - `d0` - Status request. Response should be prefixed by the `#` symbol. For example, if the timer is running, the response should be `#i2100036597`.
  - `i2100000000` - If the timer is running, 2 faults and 1 refusal will be scored on display (time is ignored).
  - `p0010000000` - If the timer is stopped, elimination will be scored on display (time is ignored).
  - `o0004200000` - Set or pause course walk time.
  - `g0004200000` - Start or resume course walk time.
  - `p0000000000` - Reset. 
