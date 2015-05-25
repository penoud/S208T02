# S208T02
An arduino PWM controller for the grove SSD relay base on the S208T02 chip planned to be used with GrovePi+. Based on Timer.h. This is fo 50Hz operations.

Example :


#include <Wire.h>
#include "Timer.h"
#include "S208T02.h"


S208T02 fan2;
S208T02 fan3;
S208T02 fan4;
S208T02 fan5;
S208T02 fan6;
S208T02 fan7;
S208T02 fan8;
Timer t;

#define SLAVE_ADDRESS 0x04

int cmd[5];
int index=0;
int flag=0;
int tevery=0;
byte val=0,b[9];
int aRead=0;
uint8_t fan;
int pin;

void setup()
{
    Serial.begin(9600);         // start serial for output
    Wire.begin(SLAVE_ADDRESS);
    Wire.onReceive(receiveData);
    Wire.onRequest(sendData);
    fan2.begin();
    fan2.setPin(2);
    fan3.begin();
    fan3.setPin(3);
    fan4.begin();
    fan4.setPin(4);
    fan5.begin();
    fan5.setPin(5);
    fan6.begin();
    fan6.setPin(6);        
    fan7.begin();
    fan7.setPin(7);        
    fan8.begin();
    fan8.setPin(8);        
    tevery = t.every(20, FanStatus);
    Serial.println("Ready!");
}

void FanStatus()
{
  fan2.checkFanStatus();
  fan3.checkFanStatus();
  fan4.checkFanStatus();
  fan5.checkFanStatus();
  fan6.checkFanStatus();
  fan7.checkFanStatus();
  fan8.checkFanStatus();
}

void loop()
{
  t.update();

  if(index==4 && flag==0)
  {
    flag=1;
    //Digital Read
    if(cmd[0]==1)
      val=digitalRead(cmd[1]);
    //Digital Write
    if(cmd[0]==2)
      digitalWrite(cmd[1],cmd[2]);
    //Analog Read
    if(cmd[0]==3)
    {
      aRead=analogRead(cmd[1]);
      b[1]=aRead/256;
      b[2]=aRead%256;
    }
    //Set up Analog Write
    if(cmd[0]==4)
      analogWrite(cmd[1],cmd[2]);
    //Set up pinMode
    if(cmd[0]==5)
      pinMode(cmd[1],cmd[2]);
    //Firmware version
    if(cmd[0]==8)
    {
      b[1] = 1;
      b[2] = 2;
      b[3] = 3;
    }
    
    //SSD Relay
    if(cmd[0]==21)
    {
      t.stop(tevery);
      if(cmd[1]==2){
        fan2.controlFanSpeed(cmd[2]);
        fan2.setFanControllerState(cmd[3]);
      }else if(cmd[1]==3){
        fan3.controlFanSpeed(cmd[2]);
        fan3.setFanControllerState(cmd[3]);
      }else if(cmd[1]==4){
        fan4.controlFanSpeed(cmd[2]);
        fan4.setFanControllerState(cmd[3]);
      }else if(cmd[1]==5){
        fan5.controlFanSpeed(cmd[2]);
        fan5.setFanControllerState(cmd[3]);
      }else if(cmd[1]==6){
        fan6.controlFanSpeed(cmd[2]);
        fan6.setFanControllerState(cmd[3]);
      }else if(cmd[1]==7){
        fan7.controlFanSpeed(cmd[2]);
        fan7.setFanControllerState(cmd[3]);
      }else if(cmd[1]==8){
        fan8.controlFanSpeed(cmd[2]);
        fan8.setFanControllerState(cmd[3]);
      }
      tevery = t.every(20, FanStatus);
    }
  }
}

void receiveData(int byteCount)
{
    while(Wire.available())
    {
      if(Wire.available()==4)
      {
        flag=0;
        index=0;
      }
        cmd[index++] = Wire.read();
    }
}

// callback for sending data
void sendData()
{
  if(cmd[0] == 1)
    Wire.write(val);
  if(cmd[0] == 3 || cmd[0] == 7 || cmd[0] == 56)
    Wire.write(b, 3);
  if(cmd[0] == 8 || cmd[0] == 20)
    Wire.write(b, 4);
  if(cmd[0] == 30 || cmd[0] == 40)
    Wire.write(b, 9);
}
