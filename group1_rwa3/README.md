# RWA3: Behavioral Planning with Behavior Trees

## Overview

This assignment implements a behavioral planner for autonomous driving using Behavior Trees.
The planner decides high-level driving maneuvers (lane keep, follow, lane change) based on
the current driving situation.

## Team Members

| Name              | Student ID | Email            |
|-------------------|------------|------------------|
|Pon Aswin Sankaralingam | 121322517  | aswin03@umd.edu    |
|Siddhant Deshmukh | 121322463 | iamsid@umd.edu   |
|Vaibhav Yuwaraj Shende | 121206817  | svaibhav@umd.edu |
|Venkata Madhav Tadavarthi | 121058768  | vmadhav@umd.edu|


## Files Information

1. **`bt_nodes.py`** - Condition and action nodes for the behavior tree
   - `IsVehicleAhead.update()` - Check if blocking vehicle ahead
   - `IsVehicleSlow.update()` - Check if vehicle ahead is slow
   - `IsLaneChangeSafe.update()` - Check if lane change is safe
   - `SetLaneKeepCommand.update()` - Set lane keeping command
   - `SetFollowCommand.update()` - Set vehicle following command
   - `SetLaneChangeCommand.update()` - Set lane change command

2. **`behavior_tree.py`** - Assemble the behavior tree
   - `BehaviorPlanner._build_tree()` - Build the tree structure

## Provided Files 

- `bt_framework.py` - Behavior tree base classes
- `test_behavior_tree.py` - Unit tests
- `simulator.py` - Highway driving simulator
- `visualizer.py` - Real-time matplotlib visualization

## Dependencies

### System Requirements
- **Ubuntu**
- **Carla**
- **python**: numpy, matplotlib, pygame, carla

## Setup

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# or: venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt
```

## Changes

```bash
bt_nodes.py (Line no.: 439)

# BEFORE
# Test 5: SetFollowCommand
print("\n5. Testing SetFollowCommand:")

env.vehicle_ahead_speed = 22.0
node = SetFollowCommand()
result = node.update()
cmd = blackboard.behavior_command
expected_speed = 20.0  # 22 - 2 buffer
----------------------------------------------
# AFTER
# Test 5: SetFollowCommand
print("\n5. Testing SetFollowCommand:")

env.vehicle_ahead_speed = 22.0
node = SetFollowCommand(speed_buffer = 2.0) # provided argument to the class
result = node.update()
cmd = blackboard.behavior_command
expected_speed = 20.0  # 22 - 2 buffer

```

```bash
carla_interface.py (Line no.: 80)

# BEFORE
# Road parameters (should match bt_nodes.py)
lane_width: float = 3.5
speed_limit: float = 15.0  # m/s (~34 mph) 
----------------------------------------------
# AFTER
# Road parameters (should match bt_nodes.py)
lane_width: float = 3.5
speed_limit: float = 31.0  # m/s (~70 mph) # default: 15

```


## Testing

```bash
cd group1_rwa3
```

```bash
# Run unit tests 
python3 test_behavior_tree.py

# Test individual modules
python3 bt_nodes.py
python3 behavior_tree.py
```
## Implementation Notes

Key decisions:

1. Three driving scenarios were implemented: lane keeping, vehicle following, and overtaking.
2. When the vehicle detects another vehicle ahead, it attempts to change lanes if it is safe; otherwise, it follows the leading vehicle.
3. The vehicle’s default behavior is to maintain its lane.
4. Matplotlib was used for visualization and to verify the correctness of the implementation.
5. The target *d* value is defined relative to the ego vehicle’s current lateral position within the lane.

## Output

Sample output:

```bash

$ python3 test_behavior_tree.py

============================================================
BEHAVIOR TREE UNIT TESTS
============================================================

--- Condition Node Tests ---

IsVehicleAhead:
  [PASS] No vehicle ahead -> FAILURE
  [PASS] Vehicle far (70m) -> FAILURE
  [PASS] Vehicle close and slow -> SUCCESS

IsVehicleSlow:
  [PASS] No vehicle -> FAILURE
  [PASS] Fast vehicle (29 m/s) -> FAILURE
  [PASS] Slow vehicle (22 m/s) -> SUCCESS

IsLaneChangeSafe:
  [PASS] Both clear -> SUCCESS, target=left
  [PASS] Only right clear -> SUCCESS, target=right
  [PASS] No lanes clear -> FAILURE

--- Action Node Tests ---

SetLaneKeepCommand:
  [PASS] behavior=lane_keep, d=0.0, v=31.0

SetFollowCommand:
  [PASS] behavior=follow_vehicle, v=21.0 (expected ~21.0)

SetLaneChangeCommand (left):
  [PASS] behavior=lane_change_left, d=3.5

SetLaneChangeCommand (right):
  [PASS] behavior=lane_change_right, d=-3.5

--- Behavior Planner Integration Tests ---

Empty road scenario:
  [PASS] Empty road -> lane_keep

Slow vehicle, left clear:
  [PASS] Slow vehicle, can pass -> lane_change_left

Vehicle ahead, lanes blocked:
  [PASS] Can't pass -> follow_vehicle

============================================================
RESULTS: 16 passed, 0 failed
============================================================

✅ All tests passed! Your implementation is correct.
```

```bash
$ python3 bt_nodes.py
Testing Behavior Tree Nodes...

1. Testing IsVehicleAhead:
   No vehicle ahead: Status.FAILURE (expected Status.FAILURE) ✓
   Vehicle far (50m): Status.FAILURE (expected Status.FAILURE) ✓
   Vehicle close (20m), slow: Status.SUCCESS (expected Status.SUCCESS) ✓

2. Testing IsVehicleSlow:
   No vehicle ahead: Status.FAILURE (expected Status.FAILURE) ✓
   Vehicle fast (29 m/s): Status.FAILURE (expected Status.FAILURE) ✓
   Vehicle slow (22 m/s): Status.SUCCESS (expected Status.SUCCESS) ✓

3. Testing IsLaneChangeSafe:
   Both lanes clear: Status.SUCCESS, target=left (expected left) ✓
   Only right clear: Status.SUCCESS, target=right (expected right) ✓
   No lanes clear: Status.FAILURE (expected Status.FAILURE) ✓

4. Testing SetLaneKeepCommand:
   Lane keep: lane_keep, d=0.0, v=31.0 ✓

5. Testing SetFollowCommand:
   Follow: follow_vehicle, v=20.0 (expected 20.0) ✓

6. Testing SetLaneChangeCommand:
   Lane change left: lane_change_left, d=3.5 ✓
   Lane change right: lane_change_right, d=-3.5 ✓
```

```bash
$ python3 behavior_tree.py

Testing Behavior Planner...


Behavior Tree Structure:
========================================
Selector(root)
  Sequence(LaneChange)
    IsVehicleAhead(IsVehicleAhead)
    IsVehicleSlow(IsVehicleSlow)
    IsLaneChangeSafe(IsLaneChangeSafe)
    SetLaneChangeCommand(SetLaneChangeCommand)
  Sequence(FollowVehicle)
    IsVehicleAhead(IsVehicleAhead)
    SetFollowCommand(SetFollowCommand)
  Sequence(LaneKeep)
    SetLaneKeepCommand(SetLaneKeepCommand)
========================================

1. Testing empty road (lane keep):
   Behavior: lane_keep (expected lane_keep) ✓
   Target: d=0.0m, v=31.0m/s, T=3.0s

2. Testing vehicle ahead, no passing (follow):
   Behavior: follow_vehicle (expected follow_vehicle) ✓
   Target: d=0.0m, v=21.0m/s, T=5.0s

3. Testing slow vehicle, left clear (lane change left):
   Behavior: lane_change_left (expected lane_change_left) ✓
   Target: d=3.5m, v=31.0m/s, T=4.0s

4. Testing slow vehicle, only right clear (lane change right):
   Behavior: lane_change_right (expected lane_change_right) ✓
   Target: d=-3.5m, v=31.0m/s, T=4.0s

5. Testing fast vehicle ahead (follow, not overtake):
   Behavior: follow_vehicle (expected follow_vehicle) ✓
   Target: d=0.0m, v=28.0m/s, T=5.0s

```


## Visualization (Optional)

Once the implementation passes unit tests, we can visualize it in action:

```bash
# Run with visualization
python3 simulator.py --scenario empty
python3 simulator.py --scenario follow
python3 simulator.py --scenario overtake

# Run in text-only mode
python3 simulator.py --no-viz --scenario overtake --duration 30
```

## Key Concepts

- **Selector**: Tries children until one succeeds (OR logic)
- **Sequence**: Executes children until one fails (AND logic)
- **Status.SUCCESS**: Node completed successfully
- **Status.FAILURE**: Node failed to complete
- **blackboard**: Shared data structure for communication between nodes


## CARLA Simulation

```bash
# Terminal 1
cd /path/to/CARLA_0.9.16
 ./CarlaUE4.sh -RenderOffScreen
```

```bash
# Terminal 2
cd /path/to/carla_simulator

# Empty road - tests lane keeping
python3 carla_simulator.py --scenario empty --duration 60

# Follow vehicle - tests vehicle following
python3 carla_simulator.py --scenario follow --duration 60

# Overtake - tests lane change decisions
python3 carla_simulator.py --scenario overtake --duration 90
```




## Carla Output Video

-  Empty road    : https://drive.google.com/file/d/19d1LWUSoKJ6NlpzE4r0BAv5JvlZzCLNB/view?usp=sharing

- Follow vehicle: https://drive.google.com/file/d/1T3t6_XsRCwhAGIP9qiWB7WeQOITRSTzu/view?usp=sharing

- Overtake:  https://drive.google.com/file/d/1dBVbQJoi3Yzxrl3Bk0T5BkGCu-GWwkQY/view?usp=sharing