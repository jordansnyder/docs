---
word: Examples
title: Examples
order: 3
---

Annotated examples
=======

Here you will find a bunch of examples to get you started with your new Photon!

Blink an LED
===

![One LED illustration](PLACEHOLDER)

Blinking an LED is the ["Hello World"](http://en.wikipedia.org/wiki/Hello_world_program) example of the microcontroller universe. It's a nice way to warm up and start your journey into the land of embedded hardware.

For this example, you will need:
- Your Particle device
- a Breadboard
- an LED
- a Resistor (between 220 and 1000 Ohms recommended, see note)
- a USB to micro-USB cable


**NOTE:** Since there is so much variation in the values of the forward voltage drop of the LEDs depending upon type, size, color, manufacturer, etc., you could successfully use a resistor value from anywhere between 220Ohms to 1K Ohms. In our picture below, we used a 1K resistor (Brown Black Red).


Connect everything together as shown in the picture. The negative (shorter) pin of the LED is connected to ground via a resistor and the positive (longer) pin is connected to D0.

![Fritzing Image](PLACEHOLDER)

Next, we're going to load code onto your core. Copy and paste this code into a new application on http://build.particle.io or on Particle Dev. We've heavily commented this code so that you can see what is going on in each line.

```cpp
// Program to blink an LED connected to pin D0
// of the Spark Core.

// We name pin D0 as led
int led = D0;

// This routine runs only once upon reset
void setup()
{
  // Initialize D0 pin as output
  pinMode(led, OUTPUT);
}

// This routine loops forever
void loop()
{
  digitalWrite(led, HIGH);   // Turn ON the LED
  delay(1000);               // Wait for 1000mS = 1 second
  digitalWrite(led, LOW);    // Turn OFF the LED
  delay(1000);               // Wait for 1 second
}
```

Go ahead and save this application, then flash it to your Core or Photon. You should be able to see that LED blinking away!

When you're ready for the next step, learn to control that LED over the internet!


Control LEDs over the 'net
===

![Fritzing Diagram](PLACEHOLDER)

Now that we know how to blink an LED, how about we control it over the Internet? This is where the fun begins.

Lets hook up two LEDs this time.


Here is the algorithm:
- Set up the pins as outputs that have LEDs connected to them
- Create and register a Spark function ( this gets called automagically when you make an API request to it)
- Parse the incoming command and take appropriate actions

```cpp
// -----------------------------------
// Controlling LEDs over the Internet
// -----------------------------------

// name the pins
int led1 = D0;
int led2 = D1;

// This routine runs only once upon reset
void setup()
{
   //Register our Spark function here
   Spark.function("led", ledControl);

   // Configure the pins to be outputs
   pinMode(led1, OUTPUT);
   pinMode(led2, OUTPUT);

   // Initialize both the LEDs to be OFF
   digitalWrite(led1, LOW);
   digitalWrite(led2, LOW);
}


// This routine loops forever
void loop()
{
   // Nothing to do here
}


// This function gets called whenever there is a matching API request
// the command string format is l<led number>,<state>
// for example: l1,HIGH or l1,LOW
//              l2,HIGH or l2,LOW

int ledControl(String command)
{
   int state = 0;
   //find out the pin number and convert the ascii to integer
   int pinNumber = (command.charAt(1) - '0') - 1;
   //Sanity check to see if the pin numbers are within limits
   if (pinNumber < 0 || pinNumber > 1) return -1;

   // find out the state of the led
   if(command.substring(3,7) == "HIGH") state = 1;
   else if(command.substring(3,6) == "LOW") state = 0;
   else return -1;

   // write to the appropriate pin
   digitalWrite(pinNumber, state);
   return 1;
}
```

---

The API request will look something like this:

```json
POST /v1/devices/{DEVICE_ID}/led

# EXAMPLE REQUEST IN TERMINAL
# Core ID is 0123456789abcdef
# Your access token is 123412341234
curl https://api.spark.io/v1/devices/0123456789abcdef/led \
  -d access_token=123412341234 \
  -d params=l1,HIGH
```

Note that the API endpoint is 'led', not 'ledControl'. This is because the endpoint is defined by the first argument of [Spark.function() PLACEHOLDER], which is a string of characters, rather than the second argument, which is a function.

To better understand the concept of making API calls to your device over the cloud checkout the [Cloud API reference.](#/api)

Measuring the temperature
===

![Temperature Fritzing](PLACEHOLDER)

We have now learned how to send custom commands to our device and control the hardware. But how about reading data back from our device?

In this example, we will hook up a temperature sensor to your device and read the values over the internet with a web browser.

We have used a widely available analog temperature sensor called TMP36 from Analog Devices, and is the temperature sensor that comes with your Spark Maker Kit! You can download the [datasheet here.](http://www.analog.com/static/imported-files/data_sheets/TMP35_36_37.pdf)

Notice how we are powering the sensor from 3.3V\* pin instead of the regular 3.3V. This is because the 3.3V\* pin gives out a (LC) clean filtered  voltage, ideal for analog applications like these. If the readings you get are noisy or inconsistent, add a 0.01uF (10nF) ceramic capacitor between the analog input pin (in this case, A7) and GND as shown in the set up. Ideally, the sensor should be placed away from the Core so that the heat dissipated by the Core does not affect the temperature readings.

```C++
// -----------------
// Read temperature
// -----------------

// Create a variable that will store the temperature value
double temperature = 0.0;

void setup()
{
  // Register a Spark variable here
  Spark.variable("temperature", &temperature, DOUBLE);

  // Connect the temperature sensor to A7 and configure it
  // to be an input
  pinMode(A7, INPUT);
}

void loop()
{
  int reading = 0;
  double voltage = 0.0;
  
  // Keep reading the sensor value so when we make an API
  // call to read its value, we have the latest one
  reading = analogRead(A7);
  
  // The returned value from the Core is going to be in the range from 0 to 4095
  // Calculate the voltage from the sensor reading
  voltage = (reading * 3.3) / 4095;
  
  // Calculate the temperature and update our static variable
  temperature = (voltage - 0.5) * 100;
}
```

---

The API request will look something like this:

```json
GET /v1/devices/{DEVICE_ID}/temperature

# EXAMPLE REQUEST IN TERMINAL
# Core ID is 0123456789abcdef
# Your access token is 123412341234
curl -G https://api.spark.io/v1/devices/0123456789abcdef/temperature \
  -d access_token=123412341234
```