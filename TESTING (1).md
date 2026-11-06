# Test Plan and Results Template

Use this document to record only tests you personally performed. The historical project notes confirm that an I²C scanner found APDS9960 at `0x39`, the chip ID was read as `0x9E`, and the sensor had intermittent detection before a damaged sensor was replaced. Other results below are a test plan, not claimed completed results.

| ID | Test | Procedure | Expected result | Actual result / date |
|---|---|---|---|---|
| T01 | I²C scan | Run scanner with SDA 21 / SCL 22 | Device at 0x39 | Fill in |
| T02 | Chip ID | Read sensor ID | 0x9E | Fill in |
| T03 | Gesture UP | Move hand upward | UP event printed | Fill in |
| T04 | Gesture DOWN | Move hand downward | DOWN event printed | Fill in |
| T05 | Gesture LEFT | Move hand left | LEFT event printed | Fill in |
| T06 | Gesture RIGHT | Move hand right | RIGHT event printed | Fill in |
| T07 | Pi health | GET /health | status=ok | Fill in |
| T08 | HTTP command | POST a valid gesture | 2xx and state response | Fill in |
| T09 | Invalid command | POST an unknown gesture | HTTP 400 | Fill in |
| T10 | End-to-end | Gesture in front of sensor | Mapped Pi output changes | Fill in |

## Historical troubleshooting observation

During earlier debugging, the sensor was not consistently detected. The I²C scanner reported address `0x39`; the chip ID was `0x9E`. Power, ground, SDA/SCL, pull-ups and the sensor itself were investigated. A previously used sensor had been damaged and was replaced.

Do not describe the issue as fully resolved by one specific electrical change unless your own test notes establish that cause.
