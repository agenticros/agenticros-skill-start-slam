# Start SLAM — RTAB-Map external skill

AgenticROS skill that exposes `start_slam`, `stop_slam`, `save_map`, `load_map`, `set_mapping_mode`, and `set_localization_mode` as **external** service capabilities. AgenticROS does **not** launch RTAB-Map — bring it up on the robot, then install this skill.

This skill does **not** drive the base. Pair it with [`@agenticros/explore`](https://github.com/agenticros/agenticros-skill-explore) to cover a room, then [`@agenticros/navigate-to`](https://github.com/agenticros/agenticros-skill-navigate-to) to go places on the saved map.

```bash
npx agenticros skills install @agenticros/start-slam
# or: npx agenticros skills install chrismatthieu/start-slam
# or: "skillRefs": ["@agenticros/start-slam"]
```

## Capabilities

| Id | Service | Type | What it does |
|---|---|---|---|
| `start_slam` | `rtabmap/resume` | `std_srvs/srv/Empty` | Resume mapping |
| `stop_slam` | `rtabmap/pause` | `std_srvs/srv/Empty` | Pause mapping |
| `save_map` | `rtabmap/backup` | `std_srvs/srv/Empty` | Copy the live database to `<database_path>.back` (default `~/.ros/rtabmap.db.back`) |
| `load_map` | `rtabmap/load_database` | `rtabmap_msgs/srv/LoadDatabase` | Load a `.db`; inputs `{ database_path, clear? }` |
| `set_mapping_mode` | `rtabmap/set_mode_mapping` | `std_srvs/srv/Empty` | Continue expanding the map |
| `set_localization_mode` | `rtabmap/set_mode_localization` | `std_srvs/srv/Empty` | Localize only (Nav2-friendly after a mapping session) |

RTAB-Map already publishes `/map` (`nav_msgs/OccupancyGrid`) while it runs. `save_map` persists the **graph/database**, not a PGM. To also dump a Nav2 occupancy file:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/maps/room
```

## Mission example — map, then localize

```json
{
  "mission": {
    "name": "map the room",
    "steps": [
      { "id": "slam", "capability": "start_slam", "inputs": { "request": {} } },
      { "id": "cover", "capability": "explore", "inputs": { "timeout_s": 180 } },
      { "id": "save", "capability": "save_map", "inputs": { "request": {} } },
      { "id": "localize", "capability": "set_localization_mode", "inputs": { "request": {} } }
    ]
  }
}
```

Reload a previous database:

```json
{
  "id": "load",
  "capability": "load_map",
  "inputs": {
    "database_path": "/home/user/.ros/rtabmap.db",
    "clear": true
  }
}
```

Default service names match common `rtabmap_ros` / `rtabmap_slam` bringups. If your launch file uses different names, fork the skill or edit `agenticros.capabilities[].implementation.service` in `package.json`.

## Requirements

- `rtabmap_ros` (or compatible) running with the documented services
- Transport with service-call support (local DDS / rosbridge preferred; Zenoh depends on bridge config)
- RGB-D (RealSense) or lidar topics already remapped in your RTAB-Map launch

## Operator bringup

See **[Mapping a room](https://github.com/agenticros/agenticros/blob/main/docs/mapping.md)** in the AgenticROS repo (RealSense + RTAB-Map + Nav2, no AMCL).

Short form:

```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rgb_topic:=/camera/camera/color/image_raw \
  depth_topic:=/camera/camera/aligned_depth_to_color/image_raw \
  camera_info_topic:=/camera/camera/color/camera_info \
  frame_id:=base_link \
  approx_sync:=true
```
