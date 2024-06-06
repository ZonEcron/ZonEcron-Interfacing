# ZonEcron Timer Serial Communication Commands

## Description

The ZonEcron timer provides a versatile serial interface for external control and monitoring, designed to offer maximum compatibility with a wide range of systems, despite the fact that serial communication is often considered outdated in modern applications. This interface allows you to send commands and receive data from the timer, ensuring precise control over its operation. Communication should be established at a baud rate of 5600 with 8 data bits, no parity, and 1 stop bit (8N1). These settings are crucial for ensuring that data is accurately transmitted and received between devices.

To facilitate this communication, a special dongle is required: https://www.zonecron.com/ecosistema/visualizacion/mochila . This dongle connects to your computer via USB and is recognized as a serial port by the operating system. Once connected, you can use any standard serial communication software to interact with the timer. The dongle then communicates wirelessly with the timer using a proprietary protocol, ensuring robust and reliable data transmission even in challenging environments.

This dongle will comunicate with the large display, and/or the cells. In case the large display is not available this accesory is the only way to interact with the timer, as the web server enabling other forms of communication resides in the large display. Therefore, using this dongle provides essential connectivity and control with or without the large display unit.


# ZonEcron Timer Serial Communication Commands

## Description

The ZonEcron timer provides a versatile serial interface for external control and monitoring, designed to offer maximum compatibility with a wide range of systems, despite the fact that serial communication is often considered outdated in modern applications. This interface allows you to send commands and receive data from the timer, ensuring precise control over its operation. Communication should be established at a baud rate of 5600 with 8 data bits, no parity, and 1 stop bit (8N1). These settings are crucial for ensuring that data is accurately transmitted and received between devices.

To facilitate this communication, a special dongle is required: [ZonEcron Timer Dongle](https://www.zonecron.com/ecosistema/visualizacion/mochila). This dongle connects to your computer via USB and is recognized as a serial port by the operating system. Once connected, you can use any standard serial communication software to interact with the timer. The dongle then communicates wirelessly with the timer using a proprietary protocol, ensuring robust and reliable data transmission even in challenging environments.

This dongle will communicate with the large display and/or the cells. In case the large display is not available, this accessory is the only way to interact with the timer, as the web server enabling other forms of communication resides in the large display. Therefore, using this dongle provides essential connectivity and control with or without the large display unit.

----------------------------------------------------------------------------------------------------

## Contents
- [1. Connecting Timer and Software](#1-Conecting-timer-and-software)
- [2. Keeping Connection Alive](#2-Keeping-connection-alive)
- [3. Message Diagram](#3-Message-diagram)
- [4. Mode Meanings and Examples](#4-Mode-Meanings-and-Examples)
- [5. Message Exchange](#5-Message-Exchange)
- [6. Timer Accepted Messages](#6-Timer-Accepted-Messages)
- [7. Timer to Software Examples](#7-Timer-to-Software-Examples)
- [8. Software to Timer Examples](#8-Software-to-Timer-Examples)

----------------------------------------------------------------------------------------------------

## 1. Connecting Timer and Software

The ZonEcron timer's serial communication should be established using a baud rate of 56000 with 8 data bits, no parity, and 1 stop bit (8N1). A special dongle is required for communication, which connects to the computer via USB and acts as a serial port. Once connected, any standard serial communication software can be used to interact with the timer. 

----------------------------------------------------------------------------------------------------

## 2. Keeping Connection Alive

The ZonEcron timer ensures robust and reliable data transmission even in challenging environments. The dongle communicates wirelessly with the timer using a proprietary protocol, ensuring that the connection remains stable. There are no special requirements other than ensuring that the software keeps the serial port open and listening.

----------------------------------------------------------------------------------------------------

## 3. Message Diagram

The ZonEcron timer's serial communication messages consist of a command and, in some cases, a parameter. 
- **Command:** A keyword indicating the action to be performed.
- **Parameters:** Additional information required for the command, if any.

Each message must be terminated with a 0x0A - "\n" - line feed character. 
0x0D - "\r" - carriage return characters will be ignored.

Example:
`DATA 2:1:0` : score 2 faults, 1 refusal, and not eliminated.

----------------------------------------------------------------------------------------------------

## 4. Message Meanings and Examples

The ZonEcron timer can switch between different modes using a specific command:

| Command | Parameter  | Optional | Meaning                                                      |
|---------|------------|----------|--------------------------------------------------------------|
| START   |  timestamp |    yes   | Initiates the timer.                                         |
| STOP    |  timestamp |    yes   | Stops the timer.                                             |
| DATA    |   F:R:E    |    no    | Updates the timer with fault, refusal, and elimination data. |
| RESET   |      -     |     -    | Resets the timer.                                            |
| DOWN    |   seconds  |    yes   | Sets the timer in countdown mode.                            |
| WALK    |   seconds  |    yes   | Sets the timer in coursewalk mode.                           |
| FAIL    |      -     |     -    | Notification indicating a cell failure.                      |
| OK      |      -     |     -    | Notification indicating the resolution of a cell failure.    |

Examples:

| Ejemplo       | Descripción                                           | Notas                    |
|---------------|-------------------------------------------------------|--------------------------|
| START         | Arranca el crono de 0                                 | sólo SIN mochila         |
| START 3500    | Arranca el crono marcando 3500ms. = 3,5s.             | sólo SIN mochila         |
| STOP          | Para el crono marcando el tiempo que haya contado     | sólo SIN mochila         |
| STOP 36748    | Para el crono marcando 36748ms. = 36,74s.             | sólo SIN mochila         |
| DATA 2:1:0    | Envía 2 faltas, 1 rehuse y no eliminado               |                          |
| RESET         | Pone tiempo, faltas, rehuses y eliminado a 0 y parado |                          |
| WALK          | Inicia reconocimiento de pista 7 minutos              |                          |
| WALK 360      | Inicia reconocimiento de pista 360s. = 6 minutos      |                          |
| DOWN          | Inicia cuenta 15s. para que el guia inicie la pista   | aún no con Flow Agility  |
| DOWN 20       | Inicia cuenta 20s. para que el guia inicie la pista   | aún no con Flow Agility  |
| DOWN 0        | Desactiva el modo suenta atras para salida            |                          |
| FAIL          | Alguna fotocélula está en alarma                      | desaparece en 5 segundos |
| OK            | Borra estado FAIL = Todas las celulas están ok        |                          |

----------------------------------------------------------------------------------------------------

## 5. Message Exchange

Communication between the software and the ZonEcron timer is facilitated through serial commands. Each command is sent from the software to the timer, which then responds accordingly. 

----------------------------------------------------------------------------------------------------

## 6. Timer Accepted Messages

The ZonEcron timer accepts specific commands and parameters depending on its current mode. These commands include START, STOP, DATA, RESET, DOWN, and WALK. Additionally, the timer may send notifications regarding cell failures and their resolution.

----------------------------------------------------------------------------------------------------

## 7. Timer to Software Examples

Examples of messages sent from the timer to the software include:

- `FAIL`: Notification indicating a cell failure.
- `OK`: Notification indicating the resolution of a cell failure.

----------------------------------------------------------------------------------------------------

## 8. Software to Timer Examples

Examples of messages sent from the software to the timer include:

- `START <initial_time>`: Initiates the timer with an optional initial time.
- `STOP <elapsed_time>`: Stops the timer with an optional elapsed time.
- `DATA <faults>:<refusals>:<eliminated>`: Updates the timer with fault, refusal, and elimination data.
- `RESET`: Resets the timer.
- `DOWN <countdown_time>`: Sets the timer in countdown mode with an optional countdown time.
- `WALK <coursewalk_time>`: Sets the timer in coursewalk mode with an optional coursewalk time.

----------------------------------------------------------------------------------------------------

This manual provides comprehensive guidance on establishing serial communication with the ZonEcron timer, ensuring precise control and monitoring of its operation.
