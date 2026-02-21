# Advanced Assembly Logic and I/O Synchronization

The Stäubli TX60 Assembly sequence contains complex routing capabilities, managing external hardware, tool transformations, and dynamic condition scanning.

## 1. Multi-Tool Transformations
The system relies heavily on dynamic TCP (Tool Center Point) assignments linked to a single physical gripper, differentiating pick configurations based on the payload:
- **`tTool[0]` (Cylinder Operations)**: Optimized configuration for the larger diameter and specific center of mass of the outer Cylinders. Applied when retrieving from the dispenser and placing the fully assembled unit into the final matrix.
- **`tTool[1]` (Piston Operations)**: Offset calculation specific to picking internal pistons from the pallet and carefully inserting them into the cylinders at the assembly station.

## 2. Dynamic Matrix Scanning and Conditional Logic
The piston pallet (`pPistonMatrix`) functions as a dynamic array. To account for potential misses or human intervention removing components, the robotic sequence validates payload presence:
1. Navigates to the expected matrix coordinate (`i`).
2. Lowers `tTool[1]` and checks the `pistonPresent` bit mapped to input `bIn0`.
3. If `pistonPresent == false`, a nested dynamic scan loop engages.
4. The arm iterates over subsequent points (`j = i + 1 to 3`) until the proximity sensor returns `true`.
5. The index is immediately realigned, preventing system starvation.

If all pallets are depleted and no piston is discovered, the sequence aborts safely and returns the arm to `Home`.

## 3. Synchronized Dispenser Actuation
The cylinder dispenser utilizes external pneumatic actuation to feed cylinders. Interaction logic requires strictly synchronized I/O handshakes to prevent mechanical collisions with the feeder arm:
- Controller outputs `cylFeeder` (high) to actuate the piston.
- Halts execution (`wait`) until input `feederExtended` confirms full push stroke.
- Halts execution until input `cylReady` establishes cylinder presence at the exact drop coordinate.
- Drops output `cylFeeder` (low) to retract the feeding arm, proceeding only after `feederContract` triggers.

## 4. Motion Interpolation
While general spatial transit utilizes `movej` for optimal configuration transitions, the system heavily incorporates linear interpolation (`movel`) and precise approach vectors (`appro()`) in the final millimeters of interaction:
- e.g., `movel(appro(target, {0,0,-30,0,0,0}))`. 
- This enforces strict vertical Z-axis approaches over the cylinders and pistons, guaranteeing payloads are not struck horizontally during high-speed routing.
