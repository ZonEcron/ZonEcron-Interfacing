# ZonEcron Timer Serial Communication Commands

## Description

The ZonEcron timer provides a versatile serial interface for external control and monitoring, designed to offer maximum compatibility with a wide range of systems, despite the fact that serial communication is often considered outdated in modern applications. This interface allows you to send commands and receive data from the timer, ensuring precise control over its operation. Communication should be established at a baud rate of 5600 with 8 data bits, no parity, and 1 stop bit (8N1). These settings are crucial for ensuring that data is accurately transmitted and received between devices.

To facilitate this communication, a special dongle is required: https://www.zonecron.com/ecosistema/visualizacion/mochila . This dongle connects to your computer via USB and is recognized as a serial port by the operating system. Once connected, you can use any standard serial communication software to interact with the timer. The dongle then communicates wirelessly with the timer using a proprietary protocol, ensuring robust and reliable data transmission even in challenging environments.

This dongle will comunicate with the large display, and/or the cells. In case the large display is not available this accesory is the only way to interact with the timer, as the web server enabling other forms of communication resides in the large display. Therefore, using this dongle provides essential connectivity and control with or without the large display unit.

## Commands
- [START](#start)
- [STOP](#stop)
- [DATA](#data)
- [RESET](#reset)
- [DOWN](#down)
- [WALK](#walk)
- [FAIL](#fail)
- [OK](#ok)

## Commands

### START
- **Description:** Initiates the timer.
- **Usage:** `START <optional: initial_time>`
  - The optional `initial_time` parameter specifies the initial time for the timer.
  - Example: `START 30000`

### STOP
- **Description:** Stops the timer.
- **Usage:** `STOP <optional: elapsed_time>`
  - The optional `elapsed_time` parameter specifies the elapsed time for the timer.
  - Example: `STOP 15000`

### DATA
- **Description:** Updates the timer with fault, refusal, and elimination data.
- **Usage:** `DATA <faults>:<refusals>:<eliminated>`
  - Parameters:
    - `<faults>`: Number of faults.
    - `<refusals>`: Number of refusals.
    - `<eliminated>`: Eliminated status (0 or 1).
  - Example: `DATA 2:0:0`

### RESET
- **Description:** Resets the timer.
- **Usage:** `RESET`

### DOWN
- **Description:** Sets the timer in countdown mode.
- **Usage:** `DOWN <optional: countdown_time>`
  - The optional `countdown_time` parameter specifies the countdown time in milliseconds.
  - Example: `DOWN 30000`

### WALK
- **Description:** Sets the timer in coursewalk mode.
- **Usage:** `WALK <optional: coursewalk_time>`
- The optional `coursewalk_time` parameter specifies the coursewalk time in milliseconds.
- Example: `WALK 420000`

### FAIL
- **Description:** Notification indicating a cell failure.
- **Usage:** `FAIL`
- **Notes:** This message is only sent from the dongle to the PC. It is not a command that can be issued to the timer.

### OK
- **Description:** Notification indicating the resolution of a cell failure.
- **Usage:** `OK`
- **Notes:** This message is only sent from the dongle to the PC. It is not a command that can be issued to the timer.

