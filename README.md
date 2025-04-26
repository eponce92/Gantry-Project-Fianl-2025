# Gantry Control System - Pen Plotter

This is a draft implementation for a 2-axis gantry control system with a pen actuator. It includes a simple state machine to handle transitions on the entire system, status and command tags to allow for controlling the gantry and simulation mode for virtual commissioning.

I added a simple HMI using TwinCAT web visualizations to test it.

![Demo Video](HMI_Video.mp4)

For final integration with any OPC UA based HMI program I exposed the relevant tags.

## Challenge Details

This project implements a pen plotter gantry that:

- Takes a point (x,y) from the HMI
- Identifies if the point is inside any of 4 pre-defined rectangular parts
- Draws the complete outline of the identified part using the gantry

The gantry has:

- X-axis servo for horizontal movement
- Y-axis servo for vertical movement
- Z-axis actuator with pen (up/down)
- Working area of 1000mm × 1000mm
- Home position at (10mm, 10mm)

## State Machine

The state machine for the gantry and servo are the same right now, separated them so future development is not constrained, both have these states and transitions:

- DISABLED: All motors off
- ENABLING: Powering up motors
- HOME_NEEDED: On but position unknown
- HOMING: Finding home position
- READY: Ready for commands
- MOVING: Going to target position
- STOPPING: Controlled stop
- ERRORED: Error state
- RESETTING: Clearing errors

Simulation mode uses timers to fake motion, homing, and other operations without real hardware. The status structure includes a has_error flag for simplified error detection. All real hardware integration points are marked with TODO comments for future implementation.

## Drawing Process

When a part is selected and drawing starts:

1. Move to first corner with pen up
2. Lower pen
3. Draw the rectangle by moving to each corner in sequence
4. Return to first corner to close the shape
5. Lift pen
6. Return to idle state

## Core Files

Here are the key files if you want to dig into the code:

### Main Blocks

- [FB_Gantry](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/FB_Gantry.TcPOU) - Controls the whole gantry
- [FB_Servo](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/FB_Servo.TcPOU) - Handles each axis

### Data Structures

- [E_Gantry_State](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/E_Gantry_State.TcDUT) - Gantry states
- [E_Servo_State](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/E_Servo_State.TcDUT) - Servo states
- [ST_Gantry_Status](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/ST_Gantry_Status.TcDUT) - Gantry status
- [ST_Servo_Status](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/ST_Servo_Status.TcDUT) - Servo status
- [ST_Position](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/ST_Position.TcDUT) - XYZ coordinates
- [ST_Move_Description](Gantry/TwinCAT%20Project1/HMI-PLC/Gantry/DUTs/ST_Move_Description.TcDUT) - Motion settings
