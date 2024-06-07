# ZONECRON TIMER INTERFACING

- Visit us at [https://www.zonecron.com](https://www.zonecron.com)
- Follow us on [Facebook](https://facebook.com/zonecron)
- Contact us at [zonecron@gmail.com](mailto:zonecron@gmail.com)

This document describes the ways to interface with a ZonEcron timer.

## Contents
- [1. Introduction and Context](#1-introduction-and-context)
- [2. Information Flow](#2-information-flow)
- [3. Four Ways to Interface with a ZonEcron Timer](#3-four-ways-to-interface-with-a-zonecron-timer)

---

## 1. Introduction and Context

Agility is a very young sport and is still in a state of rapid evolution. Besides changes in rules, course design, obstacles, and handling techniques, significant evolution is also occurring off the ring. In recent years, we have witnessed the proliferation of numerous new platforms for competition management.

During the development of a competition, we identified two main needs that we knew we could meet:
- Eliminate human error in transcribing timing results.
- Provide the helper team with remote control of the timer integrated with the platform, thus avoiding the need for multiple screens.

Since its inception in 2018, ZonEcron Timer has offered several ways to configure and operate it. We decided to make our communication system public, allowing platforms to interface directly with the timer.

---

## 2. Information Flow

The following outlines the flow of information within the system, detailing the interactions between the software and the timer.

- The timer sends messages when a dog crosses the start gates and the finish gates.
- The timer also sends information when the time for a course walk countdown or course start countdown expires.
- The software can send information on faults, refusals, and eliminations.
- The software can initiate a course walk countdown or a course start countdown.
- The software can reset the timer at any moment.

---

## 3. Four Ways to Interface with a ZonEcron Timer

Here you have four available methods to interface with a ZonEcron timer for seamless integration and efficient management. Whether you prefer WebSocket communication, RESTful APIs, or serial connection, we provide comprehensive guides for each approach. Choose the method that best fits your needs and effortlessly integrate your ZonEcron timer into your software.

- [1. ZonEcron as WebSocket Server](WebsocketServer.md)
- [2. ZonEcron as WebSocket Client](WebsocketClient.md)
- [3. ZonEcron REST Server](RESTserver.md)
- [4. ZonEcron over Serial](Serial.md)
