# Intellihorn-bluetooth

A bluetooth controlled car which would detect and lower the horn sound in front of hospitals.

We used the [Nordic Semiconductor's Bluetooth app for Android](https://play.google.com/store/apps/details?id=no.nordicsemi.android.nrftoolbox&hl=en) and configured the control buttons in the UART section.
We configured our Intel Curie/Genuino 101 with a relay and the L293D Motor Driver which gets the data and turns on the motor and the tires start moving. The few other Pins were connected with the Buzzer to demonstrate how the sound is lowered when the IR Sensor fixed infront of the chassis would sense the white coloured roads infront of the schools and the hospitals to numb down the horn voice.

> **Note:** We used the Intel Curie/Genuino 101 as our controller which has Bluetooth in-built but you can use the HC-05 as the Bluetooth if you're using the Arduino Uno or Mega.

**Code: **

```
   #include <CurieBLE.h> 


BLEPeripheral blePeripheral;  // BLE Peripheral Device (the board you're programming)
// Nordic's UART Characteristics:    https://devzone.nordicsemi.com/documentation/nrf51/6.0.0/s110/html/a00066.html
BLEService UARTService ("6E400001-B5A3-F393-E0A9-E50E24DCCA9E"); // Nordic UART Service

// BLE UART Switch Characteristic - custom 128-bit UUID, read and writable by central
BLEUnsignedCharCharacteristic RXCharacteristic ("6E400002-B5A3-F393-E0A9-E50E24DCCA9E", BLEWrite);  // The Nordic Semiconductor UART App sends chars over this RXCharacteristic.
BLEUnsignedCharCharacteristic TXCharacteristic ("6E400003-B5A3-F393-E0A9-E50E24DCCA9E", BLERead);


int  timer=0; // Timer Value
int ir = A0; // IR Sensor Analog Read
int buzzer = 9; // Buzzer pin

int m11 = 3;
int m12 = 4;
int m21 = 5;
int m22 = 6;


void setup() {
  // put your setup code here, to run once:
  Serial.begin (9600);

  // set advertised local name and service UUID:
  blePeripheral.setLocalName ("UART");
  blePeripheral.setAdvertisedServiceUuid (UARTService.uuid());

  // add service and characteristic:
  blePeripheral.addAttribute (UARTService);
  blePeripheral.addAttribute (RXCharacteristic);
  blePeripheral.addAttribute (TXCharacteristic);

  // set the initial value for the characeristic:
  RXCharacteristic.setValue (0);

  // begin advertising BLE service:
  blePeripheral.begin ();

  Serial.println ("BLE UART Peripheral");

  pinMode(3, OUTPUT);
  pinMode(4, OUTPUT);
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(11, OUTPUT);
  pinMode(A0, INPUT);


timer=millis()/1000;

}

void loop() {
  // put your main code here, to run repeatedly:

  if(analogRead(A0)>500)
  timer=millis()/1000;

  // listen for BLE peripherals to connect:
  BLECentral central = blePeripheral.central ();

  // if a central is connected to peripheral:
  if (central) {
    Serial.print ("Connected to central: ");
    // print the central's MAC address:
    Serial.println (central.address ());

    // while the central is still connected to peripheral:
    while (central.connected ()) {
      
    if(analogRead(A0)>500)
    timer=millis()/1000;
      // if the remote device wrote to the characteristic,
      if (RXCharacteristic.written()) {
        Serial.print ("Received: ");
        Serial.print (RXCharacteristic.value ());  // RX Characteristic
        Serial.print (", ");
        Serial.println (TXCharacteristic.value ());  // TX Characteristic

        car(RXCharacteristic.value ());
      }

      // https://github.com/sandeepmistry/arduino-BLEPeripheral/blob/master/BLETypedCharacteristic.h
      RXCharacteristic.setValue (0);

    }
  }


}

void car(char c) // Following are the ASCII values of the data sent by the user from the Bluetooth app.
{
  if (c == 48)
  { // ASCII '1'
    forward ();
    
  }
  if (c == 52)
  { // a 0 value
    backward ();
  }
  if (c == 49)
  { //
    left ();
  }
  if (c == 51)
  { //
    right ();
  }
  if (c == 50)
  {
    horn();
  }

  if (c == 53)
  {
    brake(); 
  }
}


// Following are the motor controls
void forward () // forward
{
  digitalWrite (m11, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m12, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m21, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m22, LOW);  //Reverse motor direction, 3 low, 4 high
  Serial.println ("forward");
}

void backward () // backward
{
  digitalWrite (m12, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m11, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m22, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m21, LOW);  //Reverse motor direction, 3 low, 4 high
  Serial.println ("backward");
}

void left () 
{
  digitalWrite (m11, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m12, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m21, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m22, LOW);  //Reverse motor direction, 3 low, 4 high
  Serial.println ("left");
}

void right () 
{
  digitalWrite (m11, HIGH);  //Reverse motor direction, 1 high, 2 low
  digitalWrite (m12, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m21, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m22, HIGH);  //Reverse motor direction, 1 high, 2 low
  Serial.println ("right");
}


void brake () // backward
{
  digitalWrite (m11, LOW); //Reverse motor direction, 1 high, 2 low
  digitalWrite (m12, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m21, LOW);  //Reverse motor direction, 3 low, 4 high
  digitalWrite (m22, LOW);  //Reverse motor direction, 1 high, 2 low
  Serial.println ("brake");
}

void horn()
{
  if ((millis()/1000)-timer<2)
  {

    analogWrite(buzzer,50);
    Serial.println("low horn");
  } // Horn buzzer when the IR detector detects white

  else
  {
    Serial.println("high horn");
    analogWrite(buzzer,255);

  } // Horn buzzer when the IR detector detects otherwise.
  delay(500); // 0.5 second delay in Read
  analogWrite(9,0); // Analrong will write on pin 9
}

```

Hope it helps! :smile:

Cheers! :tada:
