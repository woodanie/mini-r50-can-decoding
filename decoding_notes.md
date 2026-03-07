# Decoding CAN bus frames

Decoding CAN bus frames for MINI Generation 1 (R50, R52, R53)

- [0x316 (790)](#0x316-790)
  - [RPM](#rpm)
- [0x615 (1557)](#0x615-1557)
  - [Bytes](#bytes-1)
  - [Outside Temperature](#outside-temperature)
  - [Handbrake warning light](#handbrake-warning-light)
- [0x61A (1562)](#0x61a-1562)
  - [Bytes](#bytes-2)
  - [Display text](#display-text)
  - [Odometer](#odometer)
  - [Trip odometer](#trip-odometer)
  - [Clock](#clock)
  - [On-board Computer display options](#on-board-computer-display-options)
    - [Temp](#temp)
    - [Range](#range)
    - [Ave. Cons.](#ave-cons)
    - [Cons.](#cons)
    - [Ave. Speed](#ave-speed)
    - [Speed](#speed)
- [0x61F (1567)](#0x61f-1567)
  - [Bytes](#bytes-3)
  - [Instrument cluster LED brightness](#instrument-cluster-led-brightness)
  - [Indicator/warning lights](#indicatorwarning-lights)
    - [Dual gauges (Chrono pack or Navigation system)](#dual-gauges-chrono-pack-or-navigation-system))
    - [Single tachometer gauge](#single-tachometer-gauge)
    - [RPM redline segments](#rpm-redline-segments)

## 0x316 (790)

RPM.

### Bytes

- Byte 0
- Byte 1
- Byte 2: RPM LSB
- Byte 3: RPM MSB
- Byte 4
- Byte 5
- Byte 6
- Byte 7

### RPM

- (0x`B3` + `B2` hex to decimal)/6.4

  e.g.

  ```
              B2 B3                   B3 + B2
  0x316 01 00 80 51 00 00 00 00  # 0x 51 + 80 = 0x5180 → to decimal = 20864 → 20864/6.4 = 3260 RPM
  0x316 01 00 C0 55 00 00 00 00  # 0x 55 + C0 = 0x55C0 → to decimal = 21952 → 21952/6.4 = 3430 RPM
  ```

## 0x615 (1557)

Outside temperature, handbrake warning light.

### Bytes

- Byte 0
- Byte 1
- Byte 2
- Byte 3: Outside temperature in increments of 1°C, including negative temperature.
- Byte 4: Handbrake warning light
- Byte 5
- Byte 6
- Byte 7

### Outside Temperature

Outside temperature in increments of 1°C.

```
                                                         -/+
               B3                                         ↓
0x615 00 00 00 81 02 00 00 00  # 0x81 → to decimal = 129 (10000001) = -1°C (negative temperature)
0x615 00 00 00 01 02 00 00 00  # 0x01 → to decimal = 1   (00000001) =  1°C
0x615 00 00 00 13 00 00 00 00  # 0x13 → to decimal = 19  (00010011) = 19°C
0x615 00 00 00 14 00 00 00 00  # 0x14 → to decimal = 20  (00010100) = 20°C
```

### Handbrake warning light

```
                  B4                         ↓
0x615 00 00 00 13 00 00 00 00  # 0x00 (00000000) = off
0x615 00 00 00 01 02 00 00 00  # 0x02 (00000010) = on
```

## 0x61A (1562)

Odometer, trip odometer, clock, display text

### Bytes

- Byte 0: Odometer LSB
  - Increments of `1`
- Byte 1: Odometer
  - Increments of `256`
- Byte 2: Display text in high nibble, odometer MSB in low nibble.
- Byte 3: Time in minutes, or value of trip odometer in increments of `0.1`
- Byte 4: Time in hours, or value of trip odometer in increments of `25.6`
- Byte 5: Value of on board computer option
- Byte 6: Value of on board computer option
- Byte 7: Trip or Clock selected, and the current on board computer option selected.


### Display text on speedometer

High nibble contains of byte 2 contains display text, including units, inspection, oil service.

- Speedometer units in kilometers or miles
  - Kilometers: displays `km`.
  - Miles: On dual gauge (chrono pack) pre-facelift displays `miles`, facelift displays `mls`.

  ```
        ↓
  0x00 (00000000)  # Kilometers
  0x80 (10000000)  # Miles
  ```

- Oil service

  Displays "oil service"

  ```
         ↓
  0x40 (01000000)
  ```

- Clock icon

  ```
          ↓
  0x20 (00100000)
  ```

- Inspection

  On dual gauge (chrono pack) pre-facelift displays "inspect.", facelift displays "insp."

  ```
           ↓
  0x10 (00010000)
  ```

### Odometer

- Byte 0: Odometer LSB. Increments of `1`
- Byte 1: Odometer. Increments of `256`
- Byte 2: In low nibble, odometer MSB. Increments of `65536`.\
  `0x00` = 0\
  `0x01` = 65536\
  `0x02` = 131072\
  etc.. to\
  `0x0F` = 983040 (max value)

- Low nibble of `B2` + `B1` + `B0` to decimal
- Odometer maximum value: `999999`
- e.g.

  ```
                            (low nibble of B2)
                                      ↓
        B0 B1 B2                     B2   B1   B0
  0x61A 97 F9 01 D7 47 76 07 03  # 0x 1 + F9 + 97 = 0x1F997 → to decimal = 129431
  0x61A 3F 42 0F D7 47 76 07 03  # 0x F + 42 + 3F = 0xF423F → to decimal = 999999 (maximum value)
  ```

### Trip odometer

The trip odometer is only displayed when byte 7 bit 0 is `0`. (`1` is clock)

```
      trip
      ↓
0x01 (00000001): Trip and "Ave. Speed"
0x02 (00000010): Trip and "Temp"
0x03 (00000011): Trip and "Range"
0x04 (00000100): Trip and "Ave. Cons."
0x07 (00000111): Trip and "Speed"
0x09 (00001001): Trip and "Cons."
```

- Minimum value: `0.0`, Maximum value: `999.9`
- (0xB4+B3)/10

  e.g.

  ```
                 B3 B4
  0x61A 97 F8 01 00 00 76 07 03  # 0.0
  0x61A 97 F8 01 01 00 76 07 03  # 0x0001 → to decimal = 1   →   1/10 =  0.1
  0x61A 97 F8 01 00 01 76 07 03  # 0x0100 → to decimal = 256 → 256/10 = 25.6
  0x61A 97 F8 01 01 01 76 07 03  # 0x0101 → to decimal = 257 → 257/10 = 25.7
  ```

### Clock

On facelift cars where clock is in the center speedometer, or speedometer of the chrono pack.

The clock is only displayed when byte 7 bit 0 is `1`. (`0` is trip odometer)

```
    clock
      ↓
0x81 (10000001): Clock and "Ave. Speed"
0x82 (10000010): Clock and "Temp"
0x83 (10000011): Clock and "Range"
0x84 (10000100): Clock and "Ave. Cons."
0x87 (10000111): Clock and "Speed"
0x89 (10001001): Clock and "Cons."
```

- Time
   - B3 sets minutes
   - B4 sets hours
   - e.g.
     ```
                    B3 B4
     0x61A 5B 03 02 A1 50 00 00 87  # 10:21
     0x61A 5B 03 02 A2 50 00 00 87  # 10:22
     0x61A 5B 03 02 B2 53 00 00 87  # 13:32
     0x61A 5B 03 02 D9 56 00 00 87  # 16:59
     ```

- 12h clock

  - AM is set by byte 7 bit 1.

    ```
                                            AM
                                            ↓
    0x61A 5B 03 02 A1 50 00 00 C7  # 0xC7 (11001001) = 10:21 AM
    ```

  - PM is set by byte 7 bit 2.

    ```
                                             PM
                                             ↓
    0x61A 5B 03 02 A1 50 00 00 A7  # 0xA7 (10101001) = 10:21 PM
    ```

- 24h clock when byte 7 bit 1 and bit 2 are set to 0.

  ```
                                          00
                                          ↓↓
  0x61A 5B 03 02 A1 50 00 00 87  # 0x87 (10000111) = 10:21
  ```


- When time is not set:
  - B4 will be `7F`
  - B3 changes between `7E` and `FE` to make `:` flash

  ```
                 B3 B4               ↓
  0x61A 5B 03 02 7E 7F 00 00 87  # -- --
  0x61A 5B 03 02 FE 7F 00 00 87  # --:--
  ```

### On-board Computer display options

- On cars with the On-board Computer the display options are:
  - `Temp`
  - `Range`
  - `Ave. Cons.`
  - `Cons.`
  - `Ave. Speed`
  - `Speed`
- The current option is set by `B7`.
- The value for all options is set by `B5` and `B6` (`(0xB6+B5)/10`)
- When replaying CAN frames, to turn on the display you must send `0x61F` and `0x61A`, such as:
  ```
  0x61F   0xFF 0x3F 0x0B 0x00 0x40 0x00 0x00 0x00
  0x61A   0x77 0xFB 0x01 0xBE 0x47 0xBE 0x00 0x02
  ```

#### Temp

- Temperature is displayed only when byte 7 is either `0x02` ("Temp" and "Trip"), or `0x82` ("Temp" and "Clock")
- Outside temperature in increments of 0.5°C
- `Temp` info option must be selected
  - Byte 7 = `0x02` (Trip and "Temp"), or `0x82` "Clock" and "Temp")
- (0xB6+B5)/10

  e.g.

  Positive temp:

  ```
                       B5 B6 B7      B5
  0x61A 77 FB 01 C6 47 C3 00 02  # 0xC3 → to decimal = 195 → 195/10 = 19.5°C
  0x61A 78 FB 01 D1 47 C8 00 02  # 0xC8 → to decimal = 200 → 200/10 = 20°C
  ```

  Negative temp:

  ```
                       B5 B6 B7
  0x61A 60 54 00 F6 7F FB FF 02  # -0.5°C
  0x61A 60 54 00 F6 7F F6 FF 02  # -1.0°C
  0x61A 60 54 00 F6 7F F1 FF 02  # -1.5°C
  ```

#### Range

- Range is displayed only when byte 7 is either `0x03` ("Trip" and "Range"), or `0x83` ("Clock" and "Range").
- B5: Range LSB
- B6: Range MSB
- (0xB6+B5)/10

  e.g.

  ```
                       B5 B6 B7
  0x61A 77 FB 01 BE 47 BC 07 03  # 198 range
  0x61A 77 FB 01 C6 47 76 07 03  # 191 range
  0x61A 79 FB 01 D4 47 3A 07 03  # 185 range
  ```

#### Ave. Cons.

- Average fuel consumption is displayed only when byte 7 is either `0x04` ("Trip" and "Ave. Cons."), or `0x84` ("Clock" and "Ave. Cons.").
- Byte 5: LSB
- Byte 6: MSB
- (0xB6+B5)/10

  e.g.

  ```
                       B5 B6 B7
  0x61A 79 FB 01 D7 C7 43 00 04  # 0x0043 to decimal = 67 → 67/10 = 6.7 l/100km
  ```

#### Cons.

Current fuel consumption. Displayed only when byte 7 is either `0x09` ("Trip" and "Cons."), or `0x89` ("Clock" and "Cons.").

- Byte 5: Consumption LSB. Increments of `0.1`
- Byte 6: Consumption MSB. Increments of `25.6`
- (0xB6+B5)/10

  e.g.

  ```
                       B5 B6 B7
  0x61A 79 FB 01 DA C7 1C 00 09  # 0x001C → to decimal = 28  →  28/10 =  2.8 l/100km
  0x61A 79 FB 01 D9 C7 1D 00 09  # 0x001D → to decimal = 29  →  29/10 =  2.9 l/100km
  0x61A 79 FB 01 D9 C7 2F 00 09  # 0x002F → to decimal = 47  →  47/10 =  4.7 l/100km
  0x61A 79 FB 01 D8 C7 43 00 09  # 0x0043 → to decimal = 67  →  67/10 =  6.7 l/100km

                       FF 00 09  # 0x00FF → to decimal = 255 → 255/10 = 25.5 l/100km
                       00 01 09  # 0x0100 → to decimal = 256 → 256/10 = 25.6 l/100km
                       01 01 09  # 0x0101 → to decimal = 257 → 257/10 = 25.7 l/100km
                       02 01 09  # 0x0102 → to decimal = 258 → 258/10 = 25.8 l/100km
  ```

- When `--.-` is displayed, B5 will be `FE`, B6 will be `7F`

  ```
                       B5 B6 B7
  0x61A 77 FB 01 BE C7 FE 7F 09  # displays --.-
  ```

#### Ave. Speed

- Average speed is only displayed when byte 7 is either `0x01` ("Trip" and "Ave. Speed"), or `0x81` ("Clock" and "Ave. Speed").
- (0xB6+B5)/10

  e.g.

  ```
                       B5 B6 B7
  0x61A 5B 03 02 6A 41 D2 01 01  # 0x01D2 → to decimal = 466 → 466/10 = 46.6 km/h
  0x61A 78 FB 01 CB 47 FE 01 01  # 0x01FE → to decimal = 510 → 510/10 = 51.0 km/h
  ```

#### Speed

- Speed is displayed only when byte 7 is either `0x07` ("Trip" and "Speed"), or `0x87` ("Clock" and "Speed").
- Byte 5: LSB.
- Byte 6: MSB
- (0xB6+B5)/10

  e.g.

  ```
                       B5 B6 B7
  0x61A 77 FB 01 BF 47 0E 01 07  # 0x010E → to decimal = 270 → 270/10 = 27 km/h
  0x61A 77 FB 01 BF 47 36 01 07  # 0x0136 → to decimal = 310 → 310/10 = 31 km/h
  0x61A 77 FB 01 CF 47 E4 02 07  # 0x02E4 → to decimal = 740 → 740/10 = 74 km/h
  ```

When `---` is displayed, B5 will be `FE`, B6 will be `7F`

```
                     B5 B6
0x61A 68 5B 00 F6 7F FE 7F 07
```

## 0x61F (1567)

Instrument cluster and various indicator/warning lights

Example:

```
      B0 B1 B2 B3 B4 B5 B6 B7
0x61F FF 3F 02 00 40 00 00 00
```

### Bytes

- Byte 0:
- Byte 1: Instrument cluster LED brightness
- Byte 2: Various indicator/warning lights
- Byte 3: Various indicator/warning lights
- Byte 4: Various indicator/warning lights. RPM redline segments.
- Byte 5:
- Byte 6:
- Byte 7:

### Instrument cluster LED brightness

Brighness of center and tachometer gauge.

When lights are off, B1 is `3F` regardless of brightness when lights were on.

0x61F Byte 1
- `00` (`00000000`) = minumum brightness
- `3F` (`00111111`) = maximum brightness, or when lights off.

### Indicator/warning lights

#### Dual gauges (Chrono pack or Navigation system)

- 0x61F, Byte 2. Same on Pre-Facelift and Facelift gauges.
  - `02` (`00000010`) = No indicator/warning lights
  - `42` (`01000010`) = Left indicator
  - `82` (`10000010`) = High beam

- 0x61F Byte 3. Same on Pre-Facelift and Facelift gauges.
  - `00` (`00000000`) = No indicator/warning lights
  - `01` (`00000001`) = Check engine
  - `02` (`00000010`) = Handbrake
  - `04` (`00000100`) = ABS
  - `08` (`00001000`) = Right indicator
  - `10` (`00010000`) = Oil
  - `20` (`00100000`) = EML
  - `40` (`01000000`) = ASC
  - `80` (`10000000`) = Cruise control

- 0x61F Byte 4. Same on Pre-Facelift and Facelift gauges.
  - `40` (`01000000`) = No indicator/warning lights
  - `42` (`01000010`) = Battery

#### Single tachometer gauge

0x61F Byte 3
- `20` (`00100000`) = Cruise control
- `40` (`01000000`) = ASC
- `80` (`10000000`) = Seatbelt

### RPM redline segments

The RPM redline has 11 segments which are used to set a different RPM redline depending on the variant (One D 1.4, or 1.6 One/Cooper/S)

Byte 4:

- `0xFC` (`11111100`) = 5500 RPM redline (1.4 diesel `One D`)
- `0x40` (`01000000`) = 6750 RPM redline (1.6 petrol `One`, `Cooper`, and `Cooper S`)
