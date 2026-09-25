# DrumStick — Project Pause / Handoff

**Project:** DrumStick  
**Status:** PAUSED  
**Pause date:** September 25, 2026  
**Expected pause:** Approximately 3 months to 1 year  

---

# 1. Project Goal

DrumStick is a low-latency wireless air-drum system.

The complete system consists of:

- 2 motion-sensing wireless drumsticks
- 1 pressure-sensitive wireless kick pad
- 1 custom USB-C receiver
- 1 PC drum application

The sticks detect drum-playing motions using IMUs.

The kick pad detects foot pressure.

All three wireless devices communicate with one USB receiver using proprietary Nordic 2.4 GHz radio.

The receiver sends drum events to the PC through USB HID.

The PC application plays the actual drum sounds.

The goal is to allow a performer to play drums without a physical drum kit and move around the stage while performing.

---

# 2. Overall Architecture

```text
LEFT STICK  ──┐
              │
RIGHT STICK ──┼── nRF proprietary 2.4 GHz ──> USB RECEIVER ──> USB HID ──> PC APP
              │
KICK PAD ─────┘
```

Bluetooth is NOT planned for the main instrument-to-PC connection.

The dedicated receiver exists to reduce latency and jitter compared with BLE-to-PC or webcam tracking.

---

# 3. Firmware / Software Languages

Embedded firmware:

```text
Stick Firmware     -> C
Kick Firmware      -> C
Receiver Firmware  -> C
```

PC application:

```text
TypeScript
```

The embedded projects should share common protocol/radio definitions wherever practical.

---

# 4. Current GitHub Repository Structure

```text
DrumStick/
├── firmware/
│   ├── common/
│   │   ├── config.h
│   │   ├── device_id.h
│   │   ├── protocol.c
│   │   ├── protocol.h
│   │   ├── radio.c
│   │   └── radio.h
│   │
│   ├── stick/
│   │   ├── main.c
│   │   ├── imu.c
│   │   ├── imu.h
│   │   ├── hit_detect.c
│   │   ├── hit_detect.h
│   │   ├── velocity.c
│   │   ├── velocity.h
│   │   ├── battery.c
│   │   ├── battery.h
│   │   ├── power.c
│   │   ├── power.h
│   │   ├── led.c
│   │   └── led.h
│   │
│   ├── kick/
│   │   ├── main.c
│   │   ├── fsr.c
│   │   ├── fsr.h
│   │   ├── kick_detect.c
│   │   ├── kick_detect.h
│   │   ├── velocity.c
│   │   ├── velocity.h
│   │   ├── battery.c
│   │   ├── battery.h
│   │   ├── power.c
│   │   ├── power.h
│   │   ├── led.c
│   │   └── led.h
│   │
│   └── receiver/
│       ├── main.c
│       ├── receiver.c
│       ├── receiver.h
│       ├── usb_hid.c
│       ├── usb_hid.h
│       ├── packet_queue.c
│       ├── packet_queue.h
│       ├── led.c
│       └── led.h
│
├── hardware/
│   ├── stick/
│   ├── kick/
│   └── receiver/
│
├── cad/
│   ├── stick/
│   └── kick/
│
├── pc-app/
│   ├── src/
│   └── assets/
│       └── sounds/
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── PROTOCOL.md
│   └── TEST_PLAN.md
│
├── images/
├── JOURNAL.md
├── README.md
└── PROJECT_STATUS.md
```

Most firmware files currently exist mainly as structure/placeholders.

The firmware itself has NOT been fully implemented.

---

# 5. Smart Stick Hardware

Quantity:

```text
2 sticks
```

## MCU / Wireless

**Raytac MDBT50Q-1MV2**

Based on Nordic nRF52840.

Integrated antenna module.

Approximate module size:

```text
10.5 x 15.5 x 2.05 mm
```

Main wireless link:

```text
Proprietary Nordic 2.4 GHz
```

NOT normal BLE-to-PC.

The exact Raytac antenna keepout must be followed from the manufacturer datasheet.

Do not guess the antenna keepout.

---

# 6. Stick IMU

**TDK ICM-42688-P**

6-axis IMU:

- Accelerometer
- Gyroscope

Approximate package:

```text
2.5 x 3.0 x 0.91 mm
```

Planned communication:

```text
SPI
```

Capabilities include approximately:

```text
Accelerometer: ±16 g
Gyroscope:     ±2000 dps
```

The IMU should be rigidly mounted.

Its physical X/Y/Z orientation should correspond logically with the drumstick orientation.

Do not place it where the PCB can flex heavily.

---

# 7. Stick Power Supply

## LDO

**TI TPS7A0230PDBVR**

Fixed output:

```text
3.0 V
```

Package:

```text
SOT-23-5 / DBV
```

Current capability:

```text
200 mA
```

3.0 V was selected rather than 3.3 V so the single-cell LiPo can discharge farther before regulator dropout becomes a problem.

Exact input/output capacitors must still be verified using the TI datasheet.

---

# 8. Stick Charger

**MCP73831**

Initial target stick charge current:

```text
approximately 100 mA
```

Approximate initial RPROG concept:

```text
around 10 kΩ
```

BUT:

The exact MCP73831 variant and programming resistor formula MUST be checked again before fabrication.

---

# 9. Stick USB-C Charging Connector

Current intended family:

**GCT USB4505**

Likely exact part previously selected:

```text
USB4505-03-0-A
```

Purpose:

```text
Charging only
```

USB D+ / D- are not required on the sticks.

For a USB-C charging receptacle, CC1 and CC2 require correct sink configuration.

Current concept:

```text
CC1 -> 5.1 kΩ -> GND
CC2 -> 5.1 kΩ -> GND
```

Verify the exact connector footprint and manufacturer land pattern before PCB fabrication.

---

# 10. Stick Power Button

**Panasonic EVQ-P7A01P**

Momentary tactile push button.

Side/right-angle actuation was selected to allow access through the drumstick shell.

A slide switch was intentionally rejected.

---

# 11. Stick Power Controller

**MAX16150BUT+T**

Used as the smart push-button ON/OFF controller concept.

IMPORTANT:

MAX16150 should NOT automatically be assumed to carry the entire load current.

The final topology still needs to be verified.

Possible design options include:

- controlling regulator enable
- controlling a MOSFET
- controlling a load switch

Do NOT fabricate until this power topology has been reviewed against the datasheet.

---

# 12. Stick Status LED

**Lite-On LTST-C190KRKT**

Approximate characteristics:

```text
0603
Red
~631 nm
```

Exact LED resistor and control method are not finalized.

---

# 13. Stick Debug / Programming

Use SWD pads rather than a large permanent connector.

Pads required:

```text
SWDIO
SWCLK
GND
3V0
RESET
```

Programming can use pogo pins / test pads with a Nordic debugger, J-Link, CMSIS-DAP, or compatible board.

---

# 14. Stick Battery

Initial target concept:

```text
350926
1S LiPo
3.7 V
target around 200 mAh
```

IMPORTANT:

THIS BATTERY IS NOT FINAL.

"350926" is primarily a physical dimension code.

Capacity is NOT standardized.

Many commercially available 350926 batteries appear to have much lower capacity than 200 mAh.

Before purchasing:

- verify manufacturer
- verify dimensions
- verify actual capacity
- verify discharge rating
- verify protection circuit
- verify connector/wire arrangement

Prefer a protected cell.

---

# 15. Stick PCB Strategy

Use components on BOTH sides of the PCB.

This was intentionally chosen to keep the board shorter and narrower.

General concept:

```text
SIDE A
├── MDBT50Q-1MV2
└── ICM-42688-P

SIDE B
├── charger
├── power circuitry
├── LDO
└── passives
```

USB-C and push button positions should be determined together with CAD.

Important rules:

- Follow exact Raytac antenna keepout
- No prohibited copper under antenna
- No prohibited vias under antenna
- No prohibited components opposite antenna
- IMU must be rigid
- IMU axes should align logically with stick
- Avoid unnecessary PCB width
- PCB and CAD should be co-designed

---

# 16. Kick Pad Hardware

Quantity:

```text
1
```

MCU:

**Raytac MDBT42V-512KV2**

Based on:

```text
Nordic nRF52832
```

Approximate memory:

```text
512 KB Flash
64 KB RAM
```

Integrated antenna.

IMPORTANT:

Do not confuse:

```text
MDBT42V-512KV2
```

with antenna/package variants such as:

```text
MDBT42V-P512KV2
```

Check exact ordering code before purchase.

---

# 17. Kick Pressure Sensor

**Interlink FSR UX 402**

Chosen instead of a smaller/basic FSR because the intended force range is more suitable for a kick input.

Approximate force capability previously considered:

```text
roughly 0.5 N to 150 N range
```

Initial divider concept:

```text
3.0 V
  |
 FSR
  |
  +---- ADC
  |
 ~10 kΩ
  |
 GND
```

10 kΩ is NOT final.

The divider resistance should be tuned after testing the real physical kick pad.

---

# 18. Kick Mechanical Stack

Concept:

```text
Foot
 ↓
Rubber / silicone top
 ↓
Pressure distribution plate
 ↓
Force-concentrating puck
 ↓
FSR
 ↓
Rigid base
```

Important:

Do not allow full human body weight to be directly concentrated onto the FSR.

Mechanical design must protect the sensor while still transferring kick force reliably.

Non-slip material should be used on the bottom.

---

# 19. Kick Battery

Current planned battery:

**Adafruit Product 1578**

Approximate specification:

```text
3.7 V
500 mAh
```

Protected LiPo.

Approximate size previously considered:

```text
29 x 36 x 4.75 mm
```

IMPORTANT:

Check the current manufacturer's recommended charge current again before selecting MCP73831 RPROG.

Do not automatically use the originally discussed 200 mA charge value.

---

# 20. Kick Other Electronics

Planned:

```text
MCP73831 charger
TPS7A0230PDBVR 3.0 V LDO
USB-C charge connector
EVQ-P7A01P button
MAX16150 power controller
SWD pads
```

The same MAX16150 power-path warning applies to the kick unit.

---

# 21. Receiver Hardware

Quantity:

```text
1
```

MCU:

**Raytac MDBT50Q-1MV2**

Based on nRF52840.

Purpose:

- receive packets from both sticks
- receive packet from kick
- send events to PC through USB HID

---

# 22. Receiver Power

Concept:

```text
USB-C VBUS 5 V
      ↓
TLV75533
      ↓
3.3 V
      ↓
MDBT50Q-1MV2
```

Current intended regulator family:

**TI TLV75533**

Likely orderable part previously discussed:

```text
TLV75533PDBVR
```

Verify exact suffix/package before ordering.

---

# 23. Receiver USB ESD

Previously selected:

**USBLC6-2SC6**

Purpose:

Protect USB D+ / D-.

Place close to USB connector.

Keep USB differential traces short and clean.

A later conversation also mentioned another VBUS protection device, but that addition was NOT clearly finalized.

Do not assume extra TVS/protection components are finalized without checking the current schematic.

---

# 24. Receiver USB-C Male Connector

Earlier known intended part:

**Würth 629712010214**

USB Type-C male plug.

USB 2.0.

IMPORTANT:

There was later uncertainty in discussion about the exact receiver USB connector part number.

Do NOT order the receiver USB connector until the current schematic and manufacturer datasheet are checked again.

Also:

USB-C male plug/device CC wiring is NOT necessarily identical to the receptacle wiring used on the charging boards.

Do not blindly copy the stick/kick 5.1 kΩ CC arrangement.

Verify the exact plug pinout and required USB Type-C device configuration.

---

# 25. Receiver Enclosure

Current plan:

```text
No enclosure
```

Receiver will likely be used as the bare custom USB dongle PCB.

Therefore there is currently no receiver CAD folder required.

---

# 26. Wireless Topology

```text
RIGHT STICK ──┐
              │
LEFT STICK ───┼── proprietary nRF 2.4 GHz ──> RECEIVER
              │
KICK PAD ─────┘
```

There are:

```text
3 transmitters
1 receiver
```

---

# 27. Wireless Packet Concept

Initial conceptual event packet:

```text
Byte 0 : Device ID
Byte 1 : Event
Byte 2 : Zone
Byte 3 : Velocity
Byte 4 : Sequence
```

Approximate total:

```text
5 bytes
```

This is a concept, not yet a final protocol.

Still unresolved:

- RF channel
- addresses
- network IDs
- ACK strategy
- retry strategy
- collision avoidance
- simultaneous hits
- time slots
- sequence handling
- packet loss behavior
- resynchronization

The system should prioritize low latency and stable timing.

Avoid excessive retries that create unpredictable latency.

---

# 28. USB Communication

Receiver to computer:

```text
USB HID
```

Goal:

```text
No custom PC driver
```

The HID report may be designed to closely reflect the wireless event packet.

USB HID descriptor is NOT finalized.

---

# 29. Stick Firmware Architecture

Planned flow:

```text
ICM-42688-P
      ↓
    imu.c
      ↓
 hit_detect.c
      ↓
  velocity.c
      ↓
  protocol.c
      ↓
   radio.c
```

Responsibilities:

## imu.c

Only sensor driver responsibilities such as:

```text
initialize IMU
read accelerometer
read gyroscope
sensor configuration
SPI communication
```

Do NOT bury the entire hit algorithm inside imu.c.

## hit_detect.c

Responsible for:

```text
hit detection
peak detection
direction checks
cooldown
false-hit rejection
```

## velocity.c

Responsible for mapping measured motion strength into approximately:

```text
1 - 127
```

## protocol.c

Packet creation / parsing.

## radio.c

Wireless transmission/reception interface.

---

# 30. Stick Sensor Tuning

Exact hit thresholds are intentionally NOT finalized.

Do NOT try to perfectly calculate these values before physical hardware exists.

Initial firmware only needs approximate values.

Example tunable values later:

```text
HIT_THRESHOLD
HIT_COOLDOWN_MS
VELOCITY_MIN
VELOCITY_MAX
```

These should eventually be kept in a central configuration file.

After physical hardware exists, test:

```text
soft hit
medium hit
hard hit
fast repeated hits
slow movements
walking
normal arm swing
stick rotation
intentional drum strike
```

Use real sensor logs to tune the algorithm.

---

# 31. False-Hit Rejection

This is important because the performance concept includes walking around the stage.

Normal movement should NOT accidentally trigger drums.

Potential future detection may use combinations of:

```text
acceleration peak
gyroscope movement
direction
timing
state machine
cooldown
velocity profile
```

Do not over-design this before real sensor data exists.

---

# 32. Kick Firmware Architecture

Planned flow:

```text
FSR
 ↓
fsr.c
 ↓
kick_detect.c
 ↓
velocity.c
 ↓
protocol.c
 ↓
radio.c
```

Responsibilities:

## fsr.c

```text
ADC initialization
raw ADC reading
basic filtering
baseline handling
```

## kick_detect.c

```text
kick threshold
peak detection
debounce
cooldown
```

## velocity.c

Convert measured force into approximately:

```text
velocity 1 - 127
```

Exact thresholds must be tuned using the finished physical pad.

---

# 33. Receiver Firmware Architecture

Planned flow:

```text
radio.c
   ↓
receiver.c
   ↓
packet_queue.c
   ↓
usb_hid.c
   ↓
PC
```

Receiver priorities:

```text
low latency
stable reception
minimal buffering
simultaneous-hit handling
sequence checking
USB HID output
```

Avoid creating a large queue that adds noticeable latency.

---

# 34. PC Application

Language:

```text
TypeScript
```

Responsibilities:

```text
USB HID input
drum event parsing
sample playback
velocity-based volume
drum mapping
settings
```

Possible later UI:

```text
device connection status
left stick status
right stick status
kick status
velocity monitor
sensitivity
drum kit selection
master volume
debug/log screen
```

PC application has NOT been implemented yet.

---

# 35. CAD Status

CAD is NOT completed.

CAD should be completed before deep firmware development resumes.

---

# 36. Stick CAD Concept

General shape:

```text
thin
long
cylindrical / drumstick-like
```

Internal components:

```text
PCB
battery
USB-C port
push button
```

Important considerations:

- electronics must not move during fast swings
- IMU must be rigidly fixed
- battery must be secured
- USB-C must be accessible
- button must be accessible
- antenna region should not be blocked
- stick balance matters
- grip should remain usable
- PCB width should fit comfortably

The stick does NOT require complicated decorative CAD.

The major CAD challenge is fitting electronics while preserving a usable drumstick shape.

---

# 37. Kick CAD Concept

General shape:

```text
thin rectangular / square pad
```

The external enclosure itself is simple.

The difficult part is the internal mechanical force-transfer system.

Main concerns:

```text
FSR protection
force distribution
force concentration
non-slip base
PCB mounting
battery mounting
```

---

# 38. Receiver CAD

No enclosure currently planned.

Therefore:

```text
receiver CAD not required for V1
```

---

# 39. Development Environment

Firmware development:

```text
GitHub Codespaces
VS Code
C
```

Possible embedded SDK direction:

```text
Nordic nRF Connect SDK
Zephyr
```

This has NOT been fully configured yet.

Do not create final CMake/prj.conf/west configuration until the SDK and board strategy are confirmed.

---

# 40. Current Firmware Repository Philosophy

Keep modules separated by responsibility.

Examples:

```text
imu.c          -> sensor driver
hit_detect.c   -> hit algorithm
velocity.c     -> velocity mapping
radio.c        -> wireless transport
protocol.c     -> packet format
fsr.c          -> FSR/ADC
usb_hid.c      -> USB HID
battery.c      -> battery monitoring
power.c        -> power management
led.c          -> status indication
```

Do not split tiny functions into dozens of unnecessary files unless the project grows enough to require it.

---

# 41. Original Prototype History

The project originally began as a browser/webcam air-drum prototype.

Technologies included:

```text
HTML
CSS
JavaScript
MediaPipe Hands
Web Audio
```

The webcam system successfully detected hands and could trigger virtual drum sounds.

However:

```text
webcam latency
hand tracking latency
browser processing
fast motion tracking
```

were not good enough for a serious low-latency drum instrument.

This led to the custom hardware design.

---

# 42. Why Dedicated Hardware Was Chosen

New approach:

```text
IMU stick
+
IMU stick
+
FSR kick
+
dedicated RF receiver
+
USB HID
```

Advantages expected:

```text
lower latency
lower jitter
more reliable fast-motion detection
independence from webcam lighting
more consistent stage performance
```

---

# 43. Performance Goal

The long-term goal is to use DrumStick for a school/stage performance.

The visual concept is unusual because:

```text
The audience hears drums,
but there is no physical drum kit.
```

The performer may be able to walk around the stage while playing the sticks.

The kick pad still requires the performer to be near the pad during sections that use the kick.

---

# 44. Shoe Kick Sensor Idea

A future shoe-mounted kick sensor was discussed.

Possible concept:

```text
shoe-mounted IMU
or
IMU + pressure sensor
```

This could theoretically remove the fixed kick pad.

Decision:

```text
DO NOT add this to V1.
```

It would create unnecessary scope and require more false-trigger detection.

Keep the physical kick pad for V1.

---

# 45. Future LED Stage Project

A separate wireless LED stage system was discussed.

Possible future name:

```text
DrumLight
```

Possible architecture:

```text
PC / Phone
   ↓
Wireless LED controller
   ↓
LED strips
```

Potential modes:

```text
music reactive
phone BLE control
PC control
DrumStick event reactive
```

Decision:

```text
NOT part of DrumStick V1.
```

Do not expand the current project into stage lighting before DrumStick works.

---

# 46. Important Hardware Items To Re-Verify

Before ordering final hardware, verify ALL of these again:

```text
1. Exact stick LiPo battery
2. Actual battery capacity
3. Battery protection circuit
4. Exact receiver USB-C male connector
5. Receiver USB-C CC wiring
6. USB4505 exact footprint
7. MAX16150 implementation
8. Whether an additional MOSFET/load switch is required
9. MCP73831 exact variant
10. MCP73831 RPROG values
11. Kick battery allowed charge current
12. TPS7A02 capacitor requirements
13. TLV75533 exact orderable part
14. Raytac antenna keepout
15. ICM-42688-P footprint
16. ICM-42688-P orientation
17. USB ESD placement
18. SWD pad layout
19. Receiver power protection
```

Do NOT trust old distributor stock or old prices after a long pause.

Re-check availability and exact manufacturer part numbers when the project resumes.

---

# 47. PCB Workflow When Resuming

Recommended PCB process:

```text
1. Open current schematic
2. Verify schematic is the latest version
3. Check unresolved power issues
4. Assign footprints
5. Check every footprint against datasheet
6. Confirm CAD dimensions
7. Confirm board outline
8. Lock USB-C position
9. Lock button position
10. Apply antenna keepout
11. Lock IMU orientation
12. Place remaining components
13. Route
14. Ground plane review
15. DRC
16. 3D inspection
17. Final manufacturing review
```

---

# 48. CAD / PCB Relationship

Do not finish one without checking the other.

Correct approach:

```text
PCB dimensions
   ↕
CAD dimensions
   ↕
USB-C opening
   ↕
button opening
   ↕
battery space
   ↕
mounting
```

PCB and CAD should converge together.

---

# 49. Firmware Development Order

After hardware/CAD is stable:

```text
1. Common protocol
2. Common radio layer
3. Stick basic firmware
4. Kick basic firmware
5. Receiver firmware
6. USB HID
7. PC program
8. Hardware testing
9. Sensor logging
10. Algorithm tuning
```

---

# 50. First Firmware Goal

Do NOT try to create the perfect drum algorithm first.

First milestone:

```text
Stick IMU moves
     ↓
temporary threshold detects hit
     ↓
wireless packet sent
     ↓
receiver receives
     ↓
USB HID event sent
     ↓
PC plays a sound
```

For kick:

```text
Foot presses FSR
     ↓
temporary threshold detects kick
     ↓
wireless packet sent
     ↓
receiver receives
     ↓
USB HID event sent
     ↓
PC plays kick sound
```

Only after the complete pipeline works should detailed tuning begin.

---

# 51. Real Hardware Tuning

After PCB assembly:

Record actual sensor data.

Test:

```text
soft strikes
hard strikes
fast strikes
double strikes
walking
arm swinging
rotating sticks
idle movement
intentional hits
kick taps
hard kicks
foot resting on pad
```

Then adjust:

```text
threshold
peak detection
cooldown
velocity curve
direction filter
baseline
false-trigger rejection
```

---

# 52. Wireless Testing

Test at minimum:

```text
left stick only
right stick only
kick only
left + right simultaneously
stick + kick simultaneously
all three simultaneously
rapid rolls
packet loss
distance
stage movement
USB stability
```

The final system is intended for performance use, so reliability matters.

---

# 53. Forge / Documentation

The project is also associated with Hack Club Forge.

Forge journal entries must remain the creator's own work and honest time records.

Do not inflate development time.

Keep:

```text
photos
PCB screenshots
CAD screenshots
test videos
prototype results
design decisions
failures
revisions
```

These can later support documentation and project history.

---

# 54. Current Project State

At pause time:

```text
Overall concept       -> decided
System architecture   -> decided
Major components      -> mostly selected
Repository structure  -> created
Firmware file layout  -> created
PCB work               -> partially completed / needs final verification
CAD                    -> not completed
Firmware               -> not implemented
PC application         -> not implemented
Physical assembly      -> not completed
Sensor tuning          -> not started
Final RF protocol      -> not finalized
USB HID descriptor     -> not finalized
```

---

# 55. Reason For Pause

The project is intentionally being paused because other urgent work needs attention.

Expected pause:

```text
approximately 3 months to 1 year
```

Do not feel pressure to restart early.

The project has enough documentation to resume later.

---

# 56. EXACT RESUME POINT

When returning to DrumStick:

DO NOT redesign the entire project.

Start here:

```text
1. Read this PROJECT_STATUS.md
2. Inspect current GitHub repository
3. Open latest schematics/PCB files
4. Verify exact component part numbers and current datasheets
5. Resolve remaining power/USB/battery questions
6. Finish Stick CAD
7. Finish Kick CAD
8. Match CAD with PCB dimensions
9. Finalize PCB layouts
10. Run DRC and manufacturing review
11. Order PCB/components
12. Begin firmware while parts are shipping
```

Then:

```text
common firmware
      ↓
stick firmware
      ↓
kick firmware
      ↓
receiver firmware
      ↓
PC application
      ↓
assembly
      ↓
testing
      ↓
sensor tuning
      ↓
wireless optimization
      ↓
performance testing
```

---

# 57. Most Important Reminder

DO NOT restart by adding more features.

The architecture is already sufficiently ambitious.

Finish V1 first:

```text
2 wireless sticks
1 wireless kick pad
1 USB receiver
1 PC app
```

No shoe sensor.

No stage LED system.

No unnecessary extra features until the basic instrument works.

---

# 58. Short Project Description

DrumStick is a low-latency wireless air drum system with motion-sensing drumsticks, a pressure-sensitive kick pad, and a custom USB receiver. It lets you play drums in the air without a physical drum kit.

---

# 59. One-Line Reminder For Future Me

```text
Don't redesign it. Finish CAD, verify the hardware, build it, then tune it with real sensor data.
```

