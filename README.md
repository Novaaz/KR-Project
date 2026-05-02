# IoT Smart Home OWL Ontology
### Knowledge Representation — University of Verona
**Author:** Leonardo

---

## Overview

This project implements an **OWL ontology for a Smart Home IoT system**. The ontology models physical devices installed in rooms, the measurements they generate, and the users who interact with the system.

---

## Domain and Motivation

A smart home is modeled from the perspective of **sensor-driven monitoring and access control**. The domain covers:

- **Where** devices are installed: rooms and outdoor spaces
- **What** they measure: temperature, motion, light, humidity
- **What** they produce: typed measurement records with timestamps and numeric values
- **Who** interacts with the system: admin users (full control) and guest users (read-only)
- **How** devices communicate: over WiFi or Zigbee networks

The ontology deliberately excludes automation scenes and hub controllers to focus on the **measurement pipeline** and **user privilege model** as its original contribution.

---

## Ontology Structure

### Primitive Classes (13)

| Class | Description |
|---|---|
| `Device` | Any IoT device installed in the smart home |
| `Sensor` | Device that measures and generates data |
| `Actuator` | Device that performs physical actions |
| `Thermostat`, `SmartLight`, `SmartLock` | Concrete actuator types |
| `Room` | An indoor space in the home |
| `OutdoorSpace` | An outdoor area (e.g. garden) |
| `User` | A person interacting with the system |
| `Measurements` | A data record produced by a sensor |
| `CriticalTemperatureMeasurements` | Temperature reading above a safe threshold |
| `WiFiNetwork`, `ZigbeeNetwork` | Communication protocol subtypes |

### Defined Classes (14)

| Class | Definition | DL Construct |
|---|---|---|
| `TemperatureSensor` | `Sensor ⊓ ∃generates.TemperatureMeasurements` | Existential `∃` |
| `LightSensor` | `Sensor ⊓ ∃generates.LightMeasurements` | Existential `∃` |
| `MotionSensor` | `Sensor ⊓ ∃generates.MotionMeasurements` | Existential `∃` |
| `HumiditySensor` | `Sensor ⊓ ∃generates.HumidityMeasurements` | Existential `∃` |
| `MotionDetectedMeasurements` | `MotionMeasurements ⊓ ∃hasMotionIntensity.[≥1]` | Datatype Facet (OWL 2) |
| `MonitoredRoom` | `Room ⊓ ∃isMonitoredBy.Sensor` | Existential `∃` |
| `DangerousRoom` | `Room ⊓ ∃hasMeasurements.CriticalTemperatureMeasurements` | Existential `∃` |
| `WellMonitoredRoom` | `Room ⊓ (≥2 isMonitoredBy Sensor)` | Min Cardinality `≥n` |
| `SensorOnlyRoom` | `Room ⊓ ∃contains.Sensor ⊓ ∀contains.Sensor` | Universal `∀` + Existential |
| `InactiveDevice` | `Device ⊓ hasStatus value Off` | HasValue |
| `AdminUser` | `User ⊓ ∃controls.Actuator ⊓ ∃reads.Measurements ⊓ hasAdminPrivilege value true` | DataHasValue + `∃` |
| `GuestUser` | `User ⊓ ∃reads.Measurements ⊓ hasAdminPrivilege value false` | DataHasValue |
| `ConnectedDevice` | `Device ⊓ ∃connectedTo.Network` | Existential `∃` |
| `DeviceStatus` | `{On, Off, Standby}` | Nominal `{...}` |

---

### Object Properties (10)

| Property | Domain | Range | Characteristics |
|---|---|---|---|
| `isLocatedIn` | `Device` | `Room` | **Transitive**, **Asymmetric**, Inverse: `contains` |
| `contains` | `Room` | `Device` | Inverse: `isLocatedIn` |
| `generates` | `Sensor` | `Measurements` | — |
| `monitors` | `Sensor` | `Room` | Inverse: `isMonitoredBy` |
| `isMonitoredBy` | `Room` | `Sensor` | Inverse: `monitors` |
| `controls` | `User` | `Actuator` | — |
| `reads` | `User` | `Measurements` | — |
| `hasStatus` | `Device` | `DeviceStatus` | **Functional** |
| `connectedTo` | `Device` | `Network` | — |
| `hasMeasurements` | `Room` | `Measurements` | Inferred via **property chain** |

**Property Chain:** `contains ∘ generates ⊑ hasMeasurements`
If a room *contains* a sensor that *generates* a measurement, the reasoner infers `hasMeasurements` automatically — this is what triggers `DangerousRoom` classification.

---

### Data Properties (9)

| Property | Domain | Range | Characteristics |
|---|---|---|---|
| `hasTemperatureValue` | `TemperatureMeasurements` | `xsd:decimal` | — |
| `hasHumidityValue` | `HumidityMeasurements` | `xsd:decimal` | — |
| `hasLightValue` | `LightMeasurements` | `xsd:decimal` | — |
| `hasMotionIntensity` | `MotionMeasurements` | `xsd:integer` | — |
| `hasTimeStamp` | `Measurements` | `xsd:dateTime` | — |
| `hasDeviceName` | `Device` | `xsd:string` | — |
| `hasIPAddress` | `Device` | `xsd:string` | — |
| `hasAdminPrivilege` | `User` | `xsd:boolean` | **Functional** |
| `hasSSID` | `Network` | `xsd:string` | — |

---

### Disjointness Axioms

- **DisjointUnion** on `Device`: every Device is exactly one of `Sensor` or `Actuator`.
- **DisjointClasses** among all top-level branches: `Device`, `Room`, `OutdoorSpace`, `User`, `Measurements`, `DeviceStatus`, `Network`.
- **DisjointClasses** among all `Sensor` subtypes and among all `Actuator` subtypes.
- **DisjointClasses**: `AdminUser ⊥ GuestUser`, `WiFiNetwork ⊥ ZigbeeNetwork`.

---

## ABox — Individuals (29)

### Rooms and Spaces

| Individual | Type | Notable facts |
|---|---|---|
| `kitchen` | `Room` | Monitored by `sensor_kitchen` |
| `livingroom` | `Room` | Monitored by `sensor_livingroom` |
| `corridor` | `Room` | Monitored by `sensor_corridor` + `sensor_bathroom` (→ `WellMonitoredRoom`) |
| `bedroom` | `Room` | Monitored by `sensor_bedroom` |
| `entrance` | `Room` | Monitored by `sensor_entrance` |
| `garden` | `OutdoorSpace` | Outdoor area |

### Sensors

| Individual | Asserted type | Inferred type | Measurement generated |
|---|---|---|---|
| `sensor_kitchen` | `Sensor` | `TemperatureSensor`  | `measurement_01` (CriticalTemp, 38.5°C) |
| `sensor_livingroom` | `Sensor` | `TemperatureSensor`  | `measurement_02` (Temp, 22.0°C) |
| `sensor_corridor` | `Sensor` | `MotionSensor`  | `measurement_03` (MotionDetected, intensity=1) |
| `sensor_entrance` | `Sensor` | `MotionSensor`  | `measurement_04` (Motion, intensity=0) |
| `sensor_bedroom` | `Sensor` | `TemperatureSensor`  | `measurement_05` (Temp, 19.5°C) |
| `sensor_bathroom` | `Sensor` | `HumiditySensor`  | `measurement_06` (Humidity, 75%) |

### Actuators

| Individual | Type | Status | Network |
|---|---|---|---|
| `thermostat_corridor` | `Thermostat` | `On` | WiFi |
| `light_livingroom` | `SmartLight` | `On` | Zigbee |
| `light_corridor` | `SmartLight` | `Off` → `InactiveDevice`  | Zigbee |
| `light_entrance` | `SmartLight` | `On` | Zigbee |
| `lock_backdoor` | `SmartLock` | `Off` → `InactiveDevice`  | Zigbee |

### Users

| Individual | Asserted type | `hasAdminPrivilege` | Inferred type |
|---|---|---|---|
| `Admin` | `User` | `true` | `AdminUser`  |
| `Guest` | `User` | `false` | `GuestUser`  |

---

## Reasoner Inferences (HermiT)

| Individual | Inferred fact | How |
|---|---|---|
| `sensor_kitchen/livingroom/bedroom` | `TemperatureSensor` | generates `TemperatureMeasurements` |
| `sensor_corridor/entrance` | `MotionSensor` | generates `MotionMeasurements` |
| `sensor_bathroom` | `HumiditySensor` | generates `HumidityMeasurements` |
| `measurement_03` | `MotionDetectedMeasurements` | `hasMotionIntensity ≥ 1` |
| `kitchen` | `DangerousRoom` | contains sensor → CriticalTemp via property chain |
| all monitored rooms | `MonitoredRoom` | `isMonitoredBy some Sensor` |
| `corridor` | `WellMonitoredRoom` | 2 sensors satisfy `isMonitoredBy min 2` |
| `light_corridor`, `lock_backdoor` | `InactiveDevice` | `hasStatus value Off` |
| `Admin` | `AdminUser` | controls + reads + `hasAdminPrivilege true` |
| `Guest` | `GuestUser` | reads + `hasAdminPrivilege false` |
| all devices | `ConnectedDevice` | `connectedTo some Network` |

---

## Design Notes

**Measurements as first-class objects.** Each measurement is an ABox individual, not a bare data value. This lets the reasoner classify rooms and sensors based on measurement content via the property chain.

**User privilege via data property.** `hasAdminPrivilege` is used instead of negation (`not controls some Actuator`) because OWL's Open World Assumption makes negation-based definitions unreliable with HermiT.

**Sensor type inferred, not asserted.** All sensors are typed only as `Sensor` in the ABox — their specific subtype is inferred entirely by the reasoner based on what they generate.

---

## How to Run

1. Open `IOT_complete.owx` in **Protégé 5.x**
2. Go to **Reasoner → HermiT → Start Reasoner**
3. Switch to the **Inferred** view in the Classes tab to verify automatic classifications
