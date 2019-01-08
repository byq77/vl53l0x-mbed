# VL53L0X library for mbed

This is Pololu's VL53L0X library adapted for mbed framework. 

## Getting started

### Hardware

A [VL53L0X carrier](https://www.pololu.com/product/2490) can be purchased from Pololu's website.  Before continuing, careful reading of the [product page](https://www.pololu.com/product/2490) as well as the VL53L0X datasheet is recommended.

## ST's VL53L0X API and this library

Most of the functionality of this library is based on the [VL53L0X API](http://www.st.com/content/st_com/en/products/embedded-software/proximity-sensors-software/stsw-img005.html) provided by ST (STSW-IMG005), and some of the explanatory comments in the code are quoted or paraphrased from the API source code, API user manual (UM2039), and the VL53L0X datasheet. For more explanation about the library code and how it was derived from the API, see the comments in VL53L0X.cpp.

This library is intended to provide a quicker and easier way to get started using the VL53L0X with an Arduino-compatible controller, in contrast to customizing and compiling ST's API for the Arduino. The library has a more streamlined interface, as well as smaller storage and memory footprints. However, it does not implement some of the more advanced functionality available in the API (for example, calibrating the sensor to work well under a cover glass), and it has less robust error checking. For advanced applications, especially when storage and memory are less of an issue, consider using the VL53L0X API directly.

## Examples
Target: `CORE2`
### Single
```cpp
/*
 * This example shows how to get single-shot range
 * measurements from the VL53L0X. The sensor can optionally be
 * configured with different ranging profiles, as described in
 * the VL53L0X API user manual, to get better performance for
 * a certain application.
 */
#include <mbed.h>
#include <VL53L0X.h>

DigitalOut hsens_power(SENS_POWER_ON,1); // power on hSens terminals on CORE2
VL53L0X sensor(SENS1_PIN4, SENS1_PIN3);

// Uncomment this line to use long range mode. This
// increases the sensitivity of the sensor and extends its
// potential range, but increases the likelihood of getting
// an inaccurate reading because of reflections from objects
// other than the intended target. It works best in dark
// conditions.

//#define LONG_RANGE


// Uncomment ONE of these two lines to get
// - higher speed at the cost of lower accuracy OR
// - higher accuracy at the cost of lower speed

//#define HIGH_SPEED
//#define HIGH_ACCURACY

int main()
{
    printf("Start...\r\n");
    sensor.init();
    printf("Initialisation completed!\r\n");
    sensor.setTimeout(500);
#if defined LONG_RANGE
  // lower the return signal rate limit (default is 0.25 MCPS)
  sensor.setSignalRateLimit(0.1);
  // increase laser pulse periods (defaults are 14 and 10 PCLKs)
  sensor.setVcselPulsePeriod(VL53L0X::VcselPeriodPreRange, 18);
  sensor.setVcselPulsePeriod(VL53L0X::VcselPeriodFinalRange, 14);
#endif

#if defined HIGH_SPEED
  // reduce timing budget to 20 ms (default is about 33 ms)
  sensor.setMeasurementTimingBudget(20000);
#elif defined HIGH_ACCURACY
  // increase timing budget to 200 ms
  sensor.setMeasurementTimingBudget(200000);
#endif
    while (1)
    {
        printf("%u\r\n", sensor.readRangeSingleMillimeters());
        if (sensor.timeoutOccurred())
        {
            printf("TIMEOUT!\r\n");
        }
    }
}
```

### Continuos
```cpp
/*
 * This example shows how to use continuous mode to take
 * range measurements with the VL53L0X.
 */
#include <mbed.h>
#include <VL53L0X.h>

DigitalOut hsens_power(SENS_POWER_ON,1); // power on hSens terminals on CORE2
VL53L0X sensor(SENS1_PIN4, SENS1_PIN3);

int main()
{
    printf("Start...\r\n");
    sensor.init();
    printf("Initialisation completed!\r\n");
    sensor.setTimeout(500);
    // Start continuous back-to-back mode (take readings as
    // fast as possible).  To use continuous timed mode
    // instead, provide a desired inter-measurement period in
    // ms (e.g. sensor.startContinuous(100)).
    sensor.startContinuous(0);
    while (1)
    {
        printf("%u\r\n", sensor.readRangeContinuousMillimeters());
        if (sensor.timeoutOccurred())
        {
            printf("TIMEOUT!\r\n");
        }
    }
}
```
## Library reference

* `uint8_t last_status`<br>
  The status of the last I&sup2;C write transmission.

* `VL53L0X(PinName sda_pin, PinName scl_pin)`<br>
  Constructor.

* `void setAddress(uint8_t new_addr)`<br>
  Changes the I&sup2;C slave device address of the VL53L0X to the given value (7-bit).

* `uint8_t getAddress(void)`<br>
  Returns the current I&sup2;C address.

* `bool init(bool io_2v8 = true)`<br>
  Iniitializes and configures the sensor. If the optional argument `io_2v8` is true (the default if not specified), the sensor is configured for 2V8 mode (2.8 V I/O); if false, the sensor is left in 1V8 mode. The return value is a bool indicating whether the initialization completed successfully. 

* `void writeReg(uint8_t reg, uint8_t value)`<br>
  Writes an 8-bit sensor register with the given value.

  Register address constants are defined by the regAddr enumeration type in VL53L0X.h.<br>
  Example use: `sensor.writeReg(VL53L0X::SYSRANGE_START, 0x01);`

* `void writeReg16Bit(uint8_t reg, uint16_t value)`<br>
  Writes a 16-bit sensor register with the given value.

* `void writeReg32Bit(uint8_t reg, uint32_t value)`<br>
  Writes a 32-bit sensor register with the given value.

* `uint8_t readReg(uint8_t reg)`<br>
  Reads an 8-bit sensor register and returns the value read.

* `uint16_t readReg16Bit(uint8_t reg)`<br>
  Reads a 16-bit sensor register and returns the value read.

* `uint32_t readReg32Bit(uint8_t reg)`<br>
  Reads a 32-bit sensor register and returns the value read.

* `void writeMulti(uint8_t reg, uint8_t const * src, uint8_t count)`<br>
  Writes an arbitrary number of bytes from the given array to the sensor, starting at the given register.

* `void readMulti(uint8_t reg, uint8_t * dst, uint8_t count)`<br>
  Reads an arbitrary number of bytes from the sensor, starting at the given register, into the given array.

* `bool setSignalRateLimit(float limit_Mcps)`<br>
  Sets the return signal rate limit to the given value in units of MCPS (mega counts per second). This is the minimum amplitude of the signal reflected from the target and received by the sensor necessary for it to report a valid reading. Setting a lower limit increases the potential range of the sensor but also increases the likelihood of getting an inaccurate reading because of reflections from objects other than the intended target. This limit is initialized to 0.25 MCPS by default. The return value is a bool indicating whether the requested limit was valid.

* `float getSignalRateLimit(void)`<br>
  Returns the current return signal rate limit in MCPS.

* `bool setMeasurementTimingBudget(uint32_t budget_us)`<br>
  Sets the measurement timing budget to the given value in microseconds. This is the time allowed for one range measurement; a longer timing budget allows for more accurate measurements. The default budget is about 33000 microseconds, or 33 ms; the minimum is 20 ms. The return value is a bool indicating whether the requested budget was valid.

* `uint32_t getMeasurementTimingBudget(void)`<br>
  Returns the current measurement timing budget in microseconds.

* `bool setVcselPulsePeriod(vcselPeriodType type, uint8_t period_pclks)`
  Sets the VCSEL (vertical cavity surface emitting laser) pulse period for the given period type (`VL53L0X::VcselPeriodPreRange` or `VL53L0X::VcselPeriodFinalRange`) to the given value (in PCLKs). Longer periods increase the potential range of the sensor. Valid values are (even numbers only):

  Pre: 12 to 18 (initialized to 14 by default)<br>
  Final: 8 to 14 (initialized to 10 by default)

  The return value is a bool indicating whether the requested period was valid.

* `uint8_t getVcselPulsePeriod(vcselPeriodType type)`<br>
  Returns the current VCSEL pulse period for the given period type.

* `void startContinuous(uint32_t period_ms = 0)`<br>
  Starts continuous ranging measurements. If the optional argument `period_ms` is 0 (the default if not specified), continuous back-to-back mode is used (the sensor takes measurements as often as possible); if it is nonzero, continuous timed mode is used, with the specified inter-measurement period in milliseconds determining how often the sensor takes a measurement.

* `void stopContinuous(void)`<br>
  Stops continuous mode.

* `uint16_t readRangeContinuousMillimeters(void)`<br>
  Returns a range reading in millimeters when continuous mode is active.

* `uint16_t readRangeSingleMillimeters(void)`<br>
  Performs a single-shot ranging measurement and returns the reading in millimeters.

* `void setTimeout(uint16_t timeout)`<br>
  Sets a timeout period in milliseconds after which read operations will abort if the sensor is not ready. A value of 0 disables the timeout.

* `uint16_t getTimeout(void)`<br>
  Returns the current timeout period setting.

* `bool timeoutOccurred(void)`<br>
  Indicates whether a read timeout has occurred since the last call to `timeoutOccurred()`.

