# Start SLAM — RTAB-Map external skill

AgenticROS skill that exposes `start_slam`, `stop_slam`, and `save_map` as **external** service capabilities. AgenticROS does **not** launch RTAB-Map — bring it up on the robot, then install this skill.

```bash
npx agenticros skills install chrismatthieu/start-slam
# or: "skillRefs": ["chrismatthieu/start-slam"]
```

## Mission example

```json
{
  "mission": {
    "name": "map the room",
    "steps": [
      { "id": "go", "capability": "start_slam", "inputs": { "request": {} } },
      { "id": "pause", "capability": "stop_slam", "inputs": { "request": {} } }
    ]
  }
}
```

Default service names (`rtabmap/resume`, `rtabmap/pause`, `rtabmap/publish_map`) match common `rtabmap_ros` bringups. If your launch file uses different names, fork the skill or edit `agenticros.capabilities[].implementation.service` in `package.json`.

## Requirements

- `rtabmap_ros` (or compatible) running with the documented services
- Transport with service-call support (local DDS / rosbridge preferred; Zenoh depends on bridge config)

## Operator bringup (example)

```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rgb_topic:=/camera/color/image_raw \
  depth_topic:=/camera/depth/image_rect_raw \
  camera_info_topic:=/camera/color/camera_info
```
