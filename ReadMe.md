# Sensor Data Processing System

## Overview

The **Sensor Data Processing System** is a Python-based framework designed to manage, process, and utilize data from various sensors, specifically focusing on stereo camera data for mapping and pose estimation. The system is modular, scalable, and designed to handle real-time data efficiently while managing memory usage through periodic data dumping.

## Table of Contents

- [Overview](#overview)
- [Class Structure](#class-structure)
  - [Core Components](#core-components)
    - [Sensor](#sensor)
    - [Processor](#processor)
  - [Data Management](#data-management)
    - [SensorData](#sensordata)
    - [DumpSensorData](#dumpsensordata)
  - [Sensor Management](#sensor-management)
    - [SensorManager](#sensormanager)
    - [StereoCamera](#stereocamera)
  - [Data Processing](#data-processing)
    - [StereoDataProcessor](#stereodataprocessor)
  - [Auxiliary Classes](#auxiliary-classes)
    - [MapData](#mapdata)
    - [Pose](#pose)
    - [PoseChange](#posechange)
- [Class Relationships](#class-relationships)
- [Remark](#remark)
- [Usage](#usage)
- [Dependencies](#dependencies)
- [License](#license)

## Class Structure

### Core Components

#### Sensor

- **Description**: An abstract base class representing a generic sensor. It defines the essential interface and attributes that all sensor types must implement.
- **Key Attributes**:
  - `sensor_id`: Unique identifier for the sensor.
  - `sensor_type`: Type/category of the sensor.
  - `_data`: Dictionary to store sensor-specific data.
  - `_timestamp`: Timestamp of the latest data entry.
  - `_frame_id`: Identifier for data frames.
- **Key Methods**:
  - `initialize(sensor_data)`: Abstract method to initialize the sensor with a `SensorData` instance.
  - `get_data()`: Abstract method to retrieve sensor data.
  - `check_connection()`: Abstract method to verify sensor connectivity.
  - `shutdown()`: Abstract method to gracefully shut down the sensor.

#### Processor

- **Description**: An abstract base class for data processors. It serves as a template for processing data from various sensors.
- **Key Attributes**:
  - `sensor_manager`: Instance of `SensorManager` to manage sensors.
- **Key Methods**:
  - `process_data()`: Abstract method to define the data processing workflow.

### Data Management

#### SensorData

- **Description**: Manages live data from all sensors, storing data entries along with their timestamps and frame IDs.
- **Key Attributes**:
  - `__data`: Dictionary mapping sensor types to their data lists.
  - `__timestamp`: Dictionary mapping sensor types to their timestamp lists.
  - `__frame_id`: Dictionary mapping sensor types to their frame ID lists.
- **Key Methods**:
  - `add_sensor_data(sensor_type, sensor_data)`: Adds new data entry for a specific sensor type.
  - `get_data(sensor_type)`: Retrieves data list for a sensor type.
  - `get_timestamp(sensor_type)`: Retrieves timestamp list for a sensor type.
  - `get_frame_id(sensor_type)`: Retrieves frame ID list for a sensor type.
  - `update_data(sensor)`: Updates data by capturing new entries from a sensor.
  - `clear_oldest_entries(sensor_type)`: Removes the oldest data entry to manage memory.

#### DumpSensorData

- **Description**: Periodically clears old data entries from `SensorData` to manage memory usage.
- **Key Attributes**:
  - `sensor_data`: Instance of `SensorData`.
  - `clear_interval`: Time interval for clearing data.
  - `next_clear_time`: Next scheduled time for data clearing.
  - `sensor_type`: Type of sensor whose data is managed.
- **Key Methods**:
  - `get_clear_interval()`: Retrieves the current clear interval.
  - `set_clear_interval(interval, unit)`: Sets a new interval for data clearing.
  - `schedule_clear_once()`: Checks if it's time to clear data and performs the clearing.
  - `clear_data()`: Clears the oldest data entry and schedules the next clearing.

### Sensor Management

#### SensorManager

- **Description**: Manages all sensors and the live `SensorData` storage. It handles sensor initialization, data reading, status checking, and shutdown operations.
- **Key Attributes**:
  - `__sensors`: Dictionary mapping sensor types to sensor instances.
  - `__sensor_status`: Dictionary mapping sensor types to their connection statuses.
  - `__sensor_data`: Instance of `SensorData`.
- **Key Methods**:
  - `get_sensor(sensor_type)`: Retrieves a specific sensor instance.
  - `get_all_sensors()`: Retrieves all sensor instances.
  - `get_sensor_data()`: Retrieves the `SensorData` instance.
  - `initialize_sensors()`: Initializes all sensors.
  - `initialize_specific_sensor(sensor_type)`: Initializes a specific sensor.
  - `read_sensor_data()`: Reads data from all sensors.
  - `read_specific_sensor_data(sensor_type)`: Reads data from a specific sensor.
  - `get_sensor_status()`: Retrieves connection statuses of all sensors.
  - `get_specific_sensor_status(sensor_type)`: Retrieves the connection status of a specific sensor.
  - `shutdown_sensors()`: Shuts down all sensors.
  - `shutdown_specific_sensor(sensor_type)`: Shuts down a specific sensor.

#### StereoCamera

- **Description**: Inherits from `Sensor`, representing a stereo camera system. It captures and processes stereo images to generate point cloud data.
- **Key Attributes**:
  - `__connected`: Connection status of the cameras.
  - `__calibrated`: Calibration status of the cameras.
  - `__left_camera_index` & `__right_camera_index`: Indices for the left and right cameras.
  - `__left_camera` & `__right_camera`: OpenCV `VideoCapture` objects for each camera.
  - `__left_image` & `__right_image`: Latest captured images.
  - `__point_cloud`: Generated point cloud from stereo images.
  - `__previous_images`: Tuple storing the previous pair of images.
  - `__calibration_images`: List of image pairs used for calibration.
  - `__checkerboard_size`: Size of the checkerboard used for calibration.
  - `__Q`: Reprojection matrix for generating point clouds.
- **Key Methods**:
  - `initialize(sensor_data)`: Initializes the camera, checks connection, and performs calibration.
  - `get_data()`: Captures stereo images, computes visual odometry, disparity map, and point cloud.
  - `check_connection()`: Verifies the connection status of both cameras.
  - `shutdown()`: Releases camera resources.
  - `capture_stereo_images()`: Captures synchronized images from both cameras.
  - `compute_visual_odometry()`: Computes visual odometry between consecutive image pairs.
  - `calibrate(calibration_images, checkerboard_size)`: Calibrates the stereo camera system.
  - `compute_disparity_and_point_cloud()`: Generates disparity map and 3D point cloud.

### Data Processing

#### StereoDataProcessor

- **Description**: Inherits from `Processor`, specialized in processing data from the `StereoCamera` sensor. It handles image processing, pose estimation, map updating, and data management.
- **Key Attributes**:
  - `__map_data`: Instance of `MapData` to store point cloud data.
  - `current_pos`: Instance of `Pose` representing the current vehicle position.
  - `sensor_manager`: Instance of `SensorManager` to manage sensors.
  - `dump_stereo_data`: Instance of `DumpSensorData` to manage data clearing.
  - `__run`: Boolean flag to control the processing loop.
- **Key Methods**:
  - `process_stereo_images()`: Processes stereo images to generate a depth map.
  - `estimate_pose_change(current_depth_map)`: Estimates change in position based on the depth map.
  - `update_pose(pose_change)`: Updates the current position based on pose change.
  - `update_map(depth_map)`: Updates the map data with the latest depth map.
  - `process_data()`: Main loop coordinating image processing, pose estimation, map updating, and data dumping.
  - `get_map()`: Retrieves the current map data.

### Auxiliary Classes

#### MapData

- **Description**: Stores and manages point cloud data generated from stereo images.
- **Key Attributes**:
  - `__point_cloud`: NumPy array representing the point cloud.
- **Key Methods**:
  - `get_map()`: Retrieves the point cloud data.
  - `set_map(point_cloud)`: Updates the point cloud data.

#### Pose

- **Description**: Represents the current position and orientation of the vehicle based on sensor data.
- **Key Attributes**:
  - `__translation`: NumPy array representing positional coordinates.
  - `__rotation`: NumPy array representing orientation (e.g., Euler angles or quaternions).
- **Key Methods**:
  - `get_translation()`: Retrieves the translation vector.
  - `get_rotation()`: Retrieves the rotation matrix/vector.
  - `set_translation(translation)`: Updates the translation vector.
  - `set_rotation(rotation)`: Updates the rotation matrix/vector.

#### PoseChange

- **Description**: Represents the change in position and orientation of the vehicle.
- **Key Attributes**:
  - `__delta_translation`: NumPy array representing change in position.
  - `__delta_rotation`: NumPy array representing change in orientation.
- **Key Methods**:
  - `get_delta_translation()`: Retrieves the translation change.
  - `get__delta_rotation()`: Retrieves the rotation change.
  - `set_translation(delta_translation)`: Updates the translation change.
  - `set_rotation(delta_rotation)`: Updates the rotation change.

## Class Relationships

- **SensorManager** manages multiple **Sensor** instances, including **StereoCamera**.
- **SensorData** is utilized by **SensorManager** to store and manage live sensor data.
- **DumpSensorData** interacts with **SensorData** to periodically clear old data entries, ensuring memory efficiency.
- **Processor** serves as a base class for **StereoDataProcessor**, which handles processing tasks specific to **StereoCamera** data.
- **StereoDataProcessor** uses **SensorManager** to access sensor data, processes this data to update **Pose** and **MapData**, and utilizes **DumpSensorData** for memory management.
- **StereoCamera** captures and processes stereo images, generating point cloud data stored in **SensorData** and utilized by **StereoDataProcessor**.
- **PoseChange** represents the incremental changes in vehicle position and orientation, which are used by **StereoDataProcessor** to update **Pose**.

## Remark

The system is thought to be robust but is not yet and there are many aspects which need testing and experimentation to ensure the workflow. This system still requires work to ensure that it is fast and efficient enough to handle real life data. We have yet to also tackle the issue of the creation of `Terminate.txt` file which by existing stops the flow of execution of a sensor. For now we have kept it `Terminate.txt` but will make it more specific to the type of sensor when we deal with more sensors in due time.

## Usage

1. **Initialization**:
   - Instantiate `StereoDataProcessor`, which in turn initializes `SensorManager`, `StereoCamera`, and `DumpSensorData`.
   
2. **Processing Loop**:
   - Run the `process_data()` method of `StereoDataProcessor`.
   - The loop captures stereo images, processes them to generate a point cloud, estimates pose changes, updates the current pose and map, and manages data dumping.
   
3. **Termination**:
   - The processing loop can be terminated by creating a `Terminate.txt` file or if the `StereoCamera` sensor disconnects.
   
4. **Accessing Map Data**:
   - After processing, retrieve the generated map data using the `get_map()` method of `StereoDataProcessor`.

### Example

```python
if __name__ == "__main__":
    processor = StereoDataProcessor()
    processor.process_data()
