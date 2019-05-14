---
title: HackerPet Particle Library reference

language_tabs: # must be one of https://git.io/vQNgJ
  - cpp

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/CleverPet/HackerPet'>HackerPet Repo</a>

includes:
  - errors

search: true
---

# Introduction

`hackerpet` is an open source library for programmatically controlling a CleverPet Hub and reporting the data it produces. It takes advantage of the fact that most CleverPet Hubs use a Particle Photon which can be easily replaced with your own.  This library enables your Photon to give you full control over the Hub's touchpads, sounds, and food reward presentation.

If you've not done so already, please visit [hackerpet.com](https://hackerpet.com)

## Getting started

Along with a port of CleverPet's [complete original training curriculum](http://hackerpet.com/cleverpet-curriculum/), hackerpet exists as a [library on the Particle platform.](https://build.particle.io/libs/hackerpet/). 

```cpp
#include <hackerpet.h>
```

### Library 

All your examples must explicitly include the hackerpet library ... 

```cpp
HubInterface hub;
```

### HubInterface declaration

... and subsequently declare the variable that will be used to access your Hub's functionality.

```cpp
void setup() {
  
  Serial1.begin(38400);  // needed for device layer (hub) communication
  hub.Initialize();

  // Other setup commands go here
}
```

### setup() 


Within the <code>setup</code> function make sure to set up serial communication with the rest of the Hub and then initialize it.

```cpp
void loop() {
  
  hub.run(20); // do 20 ms of communication with the rest of the Hub 
  //MyOwnFunction(); 

}
```

### loop() function

Usually, you'll have a function that gets called repeatedly within loop() that will control the behavior of your Hub (presenting lights, distributing foodtreats, etc.).

# Yield macro "magic"

 The HackerPet library provides a set of macros that make programming the Hub much faster, more concise, and more easily understood. These macros let you "yield" from the function, temporarily exiting and re-entering at the same position.  Essentially, these macros grab the function's line number and then jump back to this line when the function is re-entered. This lets the funtion momentarily surrender the Photon to the HackerPet library and give it an opportunity to tell the rest of the Hub to turn on its lights, play sounds, dispense foodtreats, etc.

There are a few things to consider when using yield macros:

* They are ideal for moments when your code would block. E.g., during delays or when waiting for input. Yielding can also be helpful when you want to give the HubInterface a moment to send out instructions, as it can only handle so many at once.
* Functions in which you’d like to use yield(…) must begin with a yield_begin() and a yield_finish(). This is demonstrated in the examples.
* No recursion allowed.
* All local variables must be static. This means that they won’t get re-initialized and receive a new value the next time the function is entered, so be sure to explicitly initialize them to the value you want in a separate statement. To keep things clean, any variable declarations that start with `static` should not be initialized.
* Whenever the function “yields” it returns a value. This can be helpful for telling you whether the function is actually over, or if it’s still working.

## Macros

```cpp
// EXAMPLE USAGE
bool MyFunction() 
{
  yield_begin();

  yield_wait_ms(500, false);

  yield_finish();
  return true;
}
```

### yield_begin()

Place this at start of any function that uses yield statement

### yield_finish()

Place at end of any function that uses yield statement

```cpp
//EXAMPLE USAGE
unsigned long start_ms = millis();
while (millis() - start_ms < 1000) // Pause for 1000 milliseconds
{
  yield(false); // Yield program flow (e.g. to HackerPet library)
}
```

### yield(*ret*)

Yield execution of your yield enabled function while returning `ret`. Next time your yield enabled function is called, execution will continue the last yield point.

```cpp
// EXAMPLE USAGE
unsigned long start_wait = Time.now();
yield_wait_for(Time.now() - start_wait > 10, false) // wait for 10 seconds
```

### yield_wait_for(*condition*, *ret*)

Wait for a condition while yielding whenever the condition is not true. Passes the `ret` parameter whenever yielding.

```cpp
// EXAMPLE USAGE
yield_sleep(50, false); // wait for 50 us 
```

### yield_sleep(*wait_time_in_microseconds*, *ret*)

Wait for the specified number of microseconds, yielding while waiting. Uses particle micros() function, which overflows when it reaches 2^32 (i.e., every ~71.6 minutes)

```cpp
// EXAMPLE USAGE
yield_sleep_ms(300, false); // wait for 300 ms
```

### yield_sleep_ms(*wait_time_in_milliseconds*, *ret*)

Wait for the specified number of milliseconds, yielding while waiting. Uses millis() function, which overflows and returns to zero every ~49 days

```cpp
// EXAMPLE USAGE 
 yield_if(Time.now() > some_time_value, false)
```

### yield_if(*bool*, *ret*)

Yield only if a condition is true (e.g. enough time has passed).

# HubInterface

## HubInterface()

The constructor for the HubInterface. The HubInterface is created automatically and this constructor doesn't usually need to be explicitly called.

## Run(()

```cpp
// SYNTAX
Run(unsigned long forHowLong)

// EXAMPLE USAGE
void loop()
{
  hub.Run(20);
}

```

Advance the device layer state machine, with forHowLong milliseconds as the max time available to spend performing library tasks (e.g., communicating with the rest of the device). It is expected that it will be called every cycle of a loop() function.

## IsReady()

```cpp
// EXAMPLE USAGE
bool ready_bool = hub.IsReady();
```

Whether or not the hub is ready to carry out other commands.

## SetLights()

```cpp
// SYNTAX
SetLights(unsigned char whichLights, unsigned char yellow, unsigned char blue, unsigned char slew)

// EXAMPLE USAGE
SetLights(hub.LIGHT_LEFT, 0, 80, 0); // Make the left touchpad lights blue with brightness 80

// Make the left and right light touchpads "white" ("equal" parts yellow and blue) with brightness 30
hub.SetLights(hub.LIGHT_LEFT | hub.LIGHT_RIGHT, 30, 30, 0);
```

Set light colors with slew (overloaded below for flashing).

*whichLights*: see LIGHT_... constants 

In this class colors and slew can be between [0, 99] each

Constant   | Value     | Description
-----------|-----------|------------
LIGHT_LEFT | 0b00000001| Left touchpad light
LIGHT_MIDDLE | 0b00000010| Middle touchpad light
LIGHT_RIGHT | 0b00000100| Right touchpad light
LIGHT_CUE | 0b00001000| Cue (also "status") light
LIGHT_BTNS | 0b00000111| All the buttons/touchpads
LIGHT_ALL | 0b00001111| All the above


<aside class="notice">
While most humans have trichromatic vision (all the colors we can see can be described as a combination of red, green, and blue), most dogs and cats are dichromats only able to see the equivalent of human yellow and blue. This means that, in making games for dogs and cats, we usually only care about color settings expressed as "BY" rather than "RGB".
</aside>

## SetLightsRGB()

```cpp
// SYNTAX
SetLightsRGB(char whichLights, unsigned char red, unsigned char green, unsigned char blue, unsigned char slew)

// EXAMPLE
hub.SetLightsRGB(hub.LIGHT_CUE, 99, 0, 0, 0);
```

Set light colors with slew using RGB, not BY colors 

In this class colors and slew can be between [0, 99] each

*whichLights*: see LIGHT_... constants 

Constant   | Value     | Description
-----------|-----------|------------
LIGHT_LEFT | 0b00000001| Left touchpad light
LIGHT_MIDDLE | 0b00000010| Middle touchpad light
LIGHT_RIGHT | 0b00000100| Right touchpad light
LIGHT_CUE | 0b00001000| Cue (also "status") light
LIGHT_BTNS | 0b00000111| All the buttons/touchpads
LIGHT_ALL | 0b00001111| All the above

## SetLights() *flashing variant*

```cpp
// SYNTAX
SetLights(unsigned char whichLights, unsigned char yellow, unsigned char blue, unsigned char period, unsigned char on)

// EXAMPLE USAGE
hub.SetLights(hub.LIGHT_BTNS, 80, 80, 60, 30); // Every 600 ms set light to flash for 300 
```

Set lights to flash with given period.

*period*: [Period duration] * 10 ms. (0 => no flashing)

*on*: [duty cycle] * 10 ms, must be < period

*green*, *blue*, *period*, *on*: can be in [0, 99]

*whichLights*: see LIGHT_... constants 

Constant   | Value     | Description
-----------|-----------|------------
LIGHT_LEFT | 0b00000001| Left touchpad light
LIGHT_MIDDLE | 0b00000010| Middle touchpad light
LIGHT_RIGHT | 0b00000100| Right touchpad light
LIGHT_CUE | 0b00001000| Cue (also "status") light
LIGHT_BTNS | 0b00000111| All the buttons/touchpads
LIGHT_ALL | 0b00001111| All the above

## SetLightsRGB() *flashing variant*

```cpp
// SYNTAX
## SetLightsRGB(unsigned char whichLights, unsigned char red, unsigned char green, unsigned char blue, unsigned char period, unsigned char on)

// EXAMPLE USAGE
hub.SetLightsRGB(hub.LIGHT_ALL, 10, 0, 0, 99, 10); // Every 990 ms set the Hub to flash red for 10 ms
```

Set lights to flash with given period, using RGB not BY period in 10 ms increments.

*period*: [Period duration] * 10 ms. (0 => no flashing)

*on*: [duty cycle] * 10 ms, must be < period

*red*, *green*, *blue*, *period*, *on*: can be in [0, 99]

*whichLights*: see LIGHT_... constants 

Constant   | Value     | Description
-----------|-----------|------------
LIGHT_LEFT | 0b00000001| Left touchpad light
LIGHT_MIDDLE | 0b00000010| Middle touchpad light
LIGHT_RIGHT | 0b00000100| Right touchpad light
LIGHT_CUE | 0b00001000| Cue (also "status") light
LIGHT_BTNS | 0b00000111| All the buttons/touchpads
LIGHT_ALL | 0b00001111| All the above

## SetRandomButtonLights()

```cpp
// SYNTAX
SetRandomButtonLights(unsigned char numLights, unsigned char yellow, unsigned char blue, unsigned char period, unsigned char on)

// EXAMPLE USAGE
hub.SetRandomButtonLights(2, 60, 60, 0, 0);
```

Convenience function to randomly pick and illuminate a number of touchpad lights (numLights: how many. Max 3).

*returns* tgtLight: bitwise OR of light ID's selected as "targets" 

*period*: in 10 ms increments, 0=no flash 

*on*: duty cycle, 10 ms increments. must be < period 

*yellow*, *blue*, *period*, *on*: can be [0, 99] each

<aside class="notice">
"Touchpad" and "pad" are the preferred ways to refer to the touchpads, but the code sometimes uses "button".
</aside>

## PlayAudio()

```cpp
// SYNTAX
PlayAudio(unsigned char whichAudio, unsigned char volume)

// EXAMPLE USAGE
hub.PlayAudio(hub.AUDIO_SQUEAK, 20);
```

Play audio according to specified audio file ID.

*whichAudio*: see AUDIO_... constants below

*volume*: [0, 99]

Constant name| Description | Used in curriculum |
-------------|-------------|--------------------|
|AUDIO_ENTICE| Artificial squeaker        | No  | 
|AUDIO_POSITIVE| Reward sound             | Yes |
|AUDIO_DO| Single click                   | Yes | 
|AUDIO_CLICK| Clicker sound               | No  |
|AUDIO_SQUEAK| Natural squeaker           | No  |
|AUDIO_NEGATIVE| Descending low tone      | Yes | 
|AUDIO_L| Left touchpad sound             | Yes |
|AUDIO_M| Middle touchpad sound           | Yes |
|AUDIO_R| Right touchpad sound            | Yes |

## PlayTone()

```cpp
// SYNTAX
PlayTone(unsigned int frequency, unsigned char volume, unsigned char slew)

// EXAMPLE USAGE
hub.PlayTone(440, 30, 0);
```

Play specified frequency through DL speaker

 Variable  | Available values
 --------- | ----------------
 frequency | [0, 20000] Hz
 volume    | [0, 99]
 slew      | [0, 99]

  *Setting frequency to `0` will stop the tone.*

## PresentFoodtreat()

```cpp
// SYNTAX
PresentFoodtreat(unsigned char duration_decisec)

// EXAMPLE USAGE
hub.PresentFoodtreat(100);
```

Present foodtreat for specified duration, then close tray.

*duration_decisec*: specified amount of time (duration x 0.1 secs)

Setting duration_decisec to 0 will cause the foodtreat to be presented  indefinitely (in which case **RetractTray** will need to be called to retract).


## PresentAndCheckFoodtreat()

```cpp
// SYNTAX
PresentAndCheckFoodtreat(unsigned long duration_ms)

// EXAMPLE USAGE
unsigned char pact_state = hub.PACT_BEFORE_PRESENT  hub.PACT_... are states of PresentAndCHeckFoodtreat state machine
// Run until one of two possible final states
while (!(pact_state == hub.PACT_RESPONSE_FOODTREAT_TAKEN || pact_state == hub.PACT_RESPONSE_FOODTREAT_NOT_TAKEN))  
{
    pact_state = hub.PresentAndCheckFoodtreat(1000);  state machine
    hub.Run(20);
}
```

Must be run in a loop as a state machine.

Present foodtreat for specified duration (*duration_ms* in millseconds) and returns a PACT_... state (table below) to indicate if food was eaten or not (defined in this class).

Advantages: some error checking, returns if food was eaten or not.

Constant name | Constant value | Description
--------------|----------------|------------
PACT_BEFORE_PRESENT | 10 | After platter has returned and is in staging mode. Foodmachine may still be active.
PACT_PLATTER_OUT | 11 |  Platter has been sent out. 
PACT_WAIT_TIL_BACK | 12 |  Platter is out. Waiting for it to come back.
PACT_WAIT_DIAG | 13 |  Tray is waiting under the singulator. 
PACT_RESPONSE_FOODTREAT_NOT_TAKEN | 0 | Foodtreat wasn't read as taken. 
PACT_RESPONSE_FOODTREAT_TAKEN | 1 | Food treat was read as taken. 

## RetractTray()
```cpp
// SYNTAX
RetractTray()

// EXAMPLE USAGE
hub.RetractTray();
```
Retract the tray (for use with <code>PresentFoodtreat(0)</code>)

## GetNeedsDIReset()
```cpp
// SYNTAX
GetNeedsDIReset()

// EXAMPLE USAGE
hub.GetNeedsDIReset();
```
Return value of _csf_needs_DI_reset

If it returns **true** the Hub's capacitive touchpads need to be reset. 

*(see Run function implementation for use)*

## SetDIResetLock(bool)
```cpp
// SYNTAX
SetDIResetLock(bool)

// EXAMPLE USAGE

// ... do setup that gets game ready to be played ...
hub.SetDIResetLock(true);
// ... gameplay code ...
// end of interaction. Beginning of inter-game period 
hub.SetDIResetLock(false);
```
Pass 1/true to prevent DI (capacitive sensor) reset, set to 0 to allow it - allows a game to control when hub/dli may reset the capacitive touch sensor (DI) board

## ResetDI()
```cpp
// EXAMPLE USAGE
hub.ResetDI();
```
Reset the capacitive touch sensor board.

<aside class="warning"> Make sure when this functionn is called there's no need to use the touchpads.</aside>
*(see Run function implementation for use)*

## ResetFoodMachine()
```cpp
// EXAMPLE USAGE
hub.ResetFoodMachine();
```
This will reset the food state machine. This can be quite helpful if and when there is a platter that's in the wrong position (e.g., after the platter was forced to stop moving).

## GetButtonVal(unsigned char whichButton)
```cpp
// SYNTAX
GetButtonVal(unsigned char whichButton)

// EXAMPLE USAGE
hub.GetButtonVal(hub.BUTTON_LEFT); // get a reading from the left touchpad
```
Return analog touchpad (button) reading.

*whichButton*: see constants below

Constant   | Value     | Description
-----------|-----------|------------
BUTTON_LEFT | LIGHT_LEFT | Left touchpad
BUTTON_MIDDLE | LIGHT_MIDDLE| Middle touchpad
BUTTON_RIGHT | LIGHT_RIGHT | Right touchpad

## AnyButtonPressed()
```cpp
// EXAMPLE USAGE
unsigned char pressed = hub.AnyButtonPressed();

switch (pressed)
{
  case hub.BUTTON_LEFT:
    Log.info("Pressed left");
    break;
  
  case hub.BUTTON_MIDDLE:
    Log.info("Pressed middle");
    break;

  case hub.BUTTON_RIGHT:
    Log.info("Pressed righ");
    break;
}
```

Return byte representing bitwise OR of any pressed buttons.

## IsButtonPressed(unsigned char whichButton)
```cpp
// SYNTAX
IsButtonPressed(unsigned char whichButton)

// EXAMPLE USAGE
bool is_pressed = hub.IsButtonPressed(hub.BUTTON_MIDDLE);
```
Return bool whether specified button is pressed.

*whichButton*: see BUTTON_... constants

## FoodmachineState()

```cpp
// EXAMPLE USAGE
hub.PresentAndCheckFoodtreat(5000);
do
{
  yield(false);
}
while (hub.FOODMACHINE_IDLE != hub.FoodmachineState());

```

Return state (*unsigned char*) of food machine per the table below:

Constant name| Value | Description
-------------|-------|------------
hub.FOODMACHINE_LID_OPEN | 0 | lid open
hub.FOODMACHINE_MOVING_HOME | 1 | moving towards home
hub.FOODMACHINE_CHECK | 2 | checking for foodtreat in bowl
hub.FOODMACHINE_DISPENSING | 3 | dispensing foodtreat
hub.FOODMACHINE_IDLE | 4 | waiting with food in bowl
hub.FOODMACHINE_MOVING_PRESENT | 5 | moving platter towards present
hub.FOODMACHINE_WAIT | 6 | waiting at present for a time
hub.FOODMACHINE_MOVING_REMOVE | 7 | moving platter towards remove
hub.FOODMACHINE_PLATTER_ERROR_CODE | 8 | platter jammed
hub.FOODMACHINE_SINGULATOR_ERROR_CODE | 9 | singulator jammed
hub.FOODMACHINE_FOODTREAT_ERROR_CODE | 17 | singluator is empty API says 10, DL says 17


## GetDomeOpen()

```cpp
// EXAMPLE USAGE
int is_open = hub.GetDomeOpen();
```

Return dome open state (int)

Returned| Meaning
--------|--------
      -1| unknown
       0| closed 
       1| open

## IsDomeRemoved()

```cpp
// EXAMPLE USAGE
bool is_removed = hub.IsDomeRemoved();
```

Return whether the Hub dome is currently removed.

## SetButtonAudioEnabled(bool buttonAudioEnabled)
```cpp
// SYNTAX
SetButtonAudioEnabled(bool buttonAudioEnabled)

// EXAMPLE USAGE

bool MyGame() 
{
  // ... do interaction setup ...
  hub.SetButtonAudioEnabled(true); // game is ready to go, make touchapds noisier

  // ... game ended, turn off touchpad sounds ...
  hub.SetButtonAudioEnabled(false);

  return true;
}
```

Enable/disable standard touchpad sounds when touchpads are touched.

## SetMaxAudioAmplitude(unsigned char max_audio_amplitude)
```cpp
// SYNTAX
SetMaxAudioAmplitude(unsigned char max_audio_amplitude)

// EXAMPLE USAGE
hub.SetMaxAudioAmplitude(80);
```

Set max audio amplitude

*max_audio_amplitude*: [0, 99]

## SetDoPollButtons()
```cpp
// EXAMPLE USAGE
hub.SetDoPollButtons(true);
```
Turn button polling on and off (i.e., checking of buttons' states when hub.Run() is called)

## SetDoPollDiagnostics()
```cpp
// EXAMPLE USAGE
SetDoPollDiagnostics(true);
```
Turn diagnostic polling (from the HubInterface) on and off.

## IsHubOutOfFood()
```cpp
// EXAMPLE USAGE
bool no_food = hub.IsHubOutOfFood();
if (no_food)
{
  Particle.publish("..."); // (not an actual Particle webhook) perhaps send an SMS message ? 
}
```
Return true if hub is out of food

## IsSingulatorError()

```cpp
// EXAMPLE USAGE
if (hub.IsSingulatorError())
{
  Particle.publish("..."); // (not an actual Particle webhook) perhaps send an SMS message ? 
}
```

Return true if singulator error, for example if singulator jammed

## Report()
```cpp
// SYNTAX
Report(String play_start_time, String player, String challenge_id, uint32_t level, String result, uint32_t duration, bool foodtreat_presented, bool foodtreat_eaten)

//////// OR, WITH EXTRA //////////

Report(String play_start_time, String player, String challenge_id, uint32_t level, String result, uint32_t duration, bool foodtreat_presented, bool foodtreat_eaten, String extra)

// EXAMPLE USAGE
unsigned char pressed = hub.AnyButtonPressed();

String extras = String::format( "{\"pressed\":\"%c%c%c\"}",
            (pressed & 0b001 ? '1' : '0'),
            (pressed & 0b010 ? '1' : '0'),
            (pressed & 0b100 ? '1' : '0'));

// Only record when there was an interaction
hub.Report( Time.format(Time.now(), TIME_FORMAT_ISO8601_FULL),    // play_start_time
            "Pet, Clever",                                         // player
            "LearningTheApi",                                        // challenge_id
            1,     // difficulty increases with level. So level 1 is 3 pads, level 2 is 2 pads, level 3 is 1 pad
            "my_result, // result
            0, // (ms) duration
            1,   // since we're presenting foodtreats 100% of the time
            1 ? foodtreat_state == hub.PACT_RESPONSE_FOODTREAT_TAKEN : 0,// foodtreat_eaten
            extras
        );
```
Sends a report message with standard fields to the particle cloud. Returns true if successful.

 Send a report formatted in JSON to the particle    
 Cloud with Particle.publish(). The standard name   
 For the variable is "report". There are 8 standard 
 values and 1 extra field for custom metrics.       


Parameter | Description | Datatype
----------|-------------|---------
challenge_id| ID of current challenge  | c string                              
play_start_time| UTC start time of game   | String
player| string id of player  | char          
result| game result  | c string              
level| game level  | unsigned int            
duration| duration of game in ms            | unsigned int                          
foodtreat_presented| if food was presented  | bool                                  
foodtreat_eaten| if food was eaten  | bool   
extra| custom field for extra metrics  | char

*Return*                                             
 * True if successful, False otherwise         

## Report(String play_start_time, String player, String challenge_id, uint32_t level, String result, uint32_t duration, bool foodtreat_presented, bool foodtreat_eaten, String extra)
```cpp
// SYNTAX

// EXAMPLE USAGE
```
Send a report message with standard fields and extra field to the particle cloud. Returns true if successful.
