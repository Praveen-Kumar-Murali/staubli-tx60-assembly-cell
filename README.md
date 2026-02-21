# Stäubli TX60 Assembly Cell (VAL3)

## Overview
Implementation of a highly advanced, multi-component assembly sequence utilizing a Stäubli TX60 industrial robot programmed in VAL3. The robotic cell integrates precision picking, dynamic missing-component scanning, external sensor I/O synchronization, and dual-tool kinematic transformations to assemble a cylinder-piston mechanism.

## System Architecture
* **Manipulator**: Stäubli TX60 (6-axis articulated arm)
* **Controller**: CS8 Series Controller
* **Software Environment**: VAL3 via Stäubli Robotics Suite (SRS)
* **Peripheral Integration**: Synchronized tooling, pneumatic cylinder feeder, and proximity matrices.

## Operational Workflow
The primary assembly engine processes 4 distinct units sequentially through the following pipeline:
1. **Feeder Handshake**: Synchronized I/O sequence actuates external pneumatics to dispense a cylinder, parsing sensor feedback to confirm structural alignment.
2. **Cylinder Transfer**: Linear approach and pick via TCP `tTool[0]`, transporting the unit to the assembly jig.
3. **Piston Acquisition**: Traverses the `pPistonMatrix` pallet. Utilizes TCP `tTool[1]` to scan proximity endpoints. If an empty slot is detected, the arm recursively scans remaining matrices dynamically.
4. **Assembly**: A precise linear insertion (`movel` + `appro()`) mates the piston down the Z-axis into the cylinder.
5. **Final Storage**: The composite unit is gripped using the primary tool parameters and stacked in the `pFinalMatrix`.
6. **Recovery**: A built-in exception loop immediately aborts the sequence to the safe `Home` state upon detecting matrix depletion.

## Key Technical Concepts
* **Dynamic Array Scanning**: Real-time evaluation of `pistonPresent` proximity logic directly modifying Cartesian loop routes.
* **Hybrid Motion Planning**: Macroscopic multi-axis orientation transitions via `movej` coupled with micro-scale linear insertion interpolations (`movel`).
* **Tool Coordinate Frames (TCP)**: Dynamic switching between `tTool[0]` and `tTool[1]` kinematics handling variant part diameters and grips.
* **I/O Handshaking**: Robust external process execution mapping `dioLink` to binary terminals (`bIn0`, `bOut10`, etc.).

## Applications
This architecture serves as a foundational blueprint for complex industrial scenarios:
* Automated component mating and joining
* Palletizing and depalletizing arrays
* Sensor-driven error recovery configurations

## Demonstration

### Simulation Run
Video: [Insert Link to Simulation Drive / YouTube]
*(See `simulation/simulation_video_link.txt`)*

*(Note: Appropriate thumbnails and GIFs can be placed in `report/setup_images/` for visual reference)*
