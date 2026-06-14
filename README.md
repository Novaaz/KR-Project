# IoT Smart Home OWL Ontology
### Knowledge Representation — University of Verona
**Author:** Leonardo

---

## Overview

This project contains an **OWL ontology for a smart home IoT system**. It models devices placed in rooms, the measurements they produce, the networks they use, and the users who can read or control parts of the system.

The main focus is on two things:
- how measurements are represented and linked to rooms
- how user permissions are modeled in a simple but clear way

---

## Domain

The ontology describes a small smart home environment with:

- rooms and one outdoor space
- sensors for temperature, motion, and humidity
- actuators like lights, locks, and a thermostat
- measurements with values and timestamps
- users with different privileges
- devices connected through WiFi or Zigbee

---

## Ontology Structure

### Main classes

The ontology includes the main branches `Device`, `Room`, `OutdoorSpace`, `User`, `Measurements`, `DeviceStatus`, and `Network`.

Some relevant subclasses are:
- `Sensor`, `Actuator`
- `Thermostat`, `SmartLight`, `SmartLock`
- `TemperatureMeasurements`, `MotionMeasurements`, `HumidityMeasurements`
- `WiFiNetwork`, `ZigbeeNetwork`

### Defined classes

Several classes are defined through logical conditions, for example:

- `TemperatureSensor = Sensor ⊓ ∃generates.TemperatureMeasurements`
- `MotionSensor = Sensor ⊓ ∃generates.MotionMeasurements`
- `HumiditySensor = Sensor ⊓ ∃generates.HumidityMeasurements`
- `MonitoredRoom = Room ⊓ ∃isMonitoredBy.Sensor`
- `DangerousRoom = Room ⊓ ∃hasMeasurements.CriticalTemperatureMeasurements`
- `WellMonitoredRoom = Room ⊓ (≥2 isMonitoredBy Sensor)`
- `InactiveDevice = Device ⊓ hasStatus value Off`
- `AdminUser = User ⊓ ∃controls.Actuator ⊓ ∃reads.Measurements ⊓ hasAdminPrivilege value true`
- `GuestUser = User ⊓ ∃reads.Measurements ⊓ hasAdminPrivilege value false`
- `ConnectedDevice = Device ⊓ ∃connectedTo.Network`

`DeviceStatus` is modeled as a nominal: `{On, Off, Standby}`.

---

## Properties

### Object properties

Main object properties:

- `isLocatedIn` / inverse `contains`
- `generates`
- `monitors` / inverse `isMonitoredBy`
- `controls`
- `reads`
- `hasStatus`
- `connectedTo`
- `hasMeasurements`

`hasStatus` is functional.

`isLocatedIn` is modeled as **transitive** and **asymmetric**.

### Property chain

The ontology uses this property chain:

`contains ∘ generates ⊑ hasMeasurements`

So, if a room contains a sensor and that sensor generates a measurement, the room can be inferred to have that measurement.

This is what allows the reasoner to classify a room like `kitchen` as a `DangerousRoom` when it is linked to a critical temperature measurement.

### Data properties

The ontology uses data properties such as:

- `hasTemperatureValue`
- `hasHumidityValue`
- `hasLightValue`
- `hasMotionIntensity`
- `hasTimeStamp`
- `hasDeviceName`
- `hasIPAddress`
- `hasAdminPrivilege`
- `hasSSID`

`hasAdminPrivilege` is functional.

---

## Disjointness axioms

The ontology includes:

- `DisjointUnion(Device Actuator Sensor)`
- disjointness among top-level branches
- disjointness among sensor subtypes
- disjointness among actuator subtypes
- `AdminUser ⊥ GuestUser`
- `WiFiNetwork ⊥ ZigbeeNetwork`

---

## ABox individuals

The ontology currently contains **33 individuals**.

### Rooms and spaces

- `kitchen`
- `livingroom`
- `corridor`
- `bedroom`
- `entrance`
- `bathroom`
- `garden`

### Sensors

- `sensor_kitchen`
- `sensor_livingroom`
- `sensor_corridor`
- `sensor_entrance`
- `sensor_bedroom`
- `sensor_bathroom`
- `sensor_corridor_humidity`

### Actuators

- `thermostat_corridor`
- `light_livingroom`
- `light_corridor`
- `light_entrance`
- `lock_backdoor`

### Users

- `Admin`
- `Guest`

### Status values

- `On`
- `Off`
- `Standby`

### Networks

- `home_wifi`
- `home_zigbee`

### Measurements

- `measurement_01` to `measurement_07`

---

## Reasoner results

With HermiT, the ontology can infer for example:

- `sensor_kitchen`, `sensor_livingroom`, `sensor_bedroom` as `TemperatureSensor`
- `sensor_corridor`, `sensor_entrance` as `MotionSensor`
- `sensor_bathroom`, `sensor_corridor_humidity` as `HumiditySensor`
- `measurement_03` as `MotionDetectedMeasurements` from `hasMotionIntensity >= 1`
- `kitchen` as `DangerousRoom`
- monitored rooms as `MonitoredRoom`
- `corridor` as `WellMonitoredRoom`
- `light_corridor` and `lock_backdoor` as `InactiveDevice`
- `Admin` as `AdminUser`
- `Guest` as `GuestUser`
- devices connected to a network as `ConnectedDevice`

---

## Notes

A design choice in this ontology is to model measurements as individuals instead of plain values attached directly to rooms. This makes it easier to classify sensors, measurements, and rooms through reasoning.

Another choice is to model user privileges with the boolean data property `hasAdminPrivilege`. This is simpler and more reliable than trying to define users only through negation under the Open World Assumption.

Sensor subtypes are mostly left for the reasoner to infer from the type of measurement each sensor generates.

---

## How to run

1. Open `IOT.owx` in **Protégé 5.x**
2. Start **HermiT** from the Reasoner menu
3. Check the **Inferred** view in the Classes tab
4. You can also inspect inferred types for individuals in the Individuals tab
