# MP3 Encoding with LAME

[LAME](https://lame.sourceforge.io/about.php) is a open source implementation of a MP3 encoder.
This project just reorganized the code to follow Arduino Library conventions, so that you can use it in Arduino and PlatformIO. 

I have also added a simple Arduino style C++ API which allows to output the resulting MP3 via to a Arduino Stream or to receive it via a callback. 

### API
Here is an example Arduino sketch for encoding PCM data into MP3:

```
#include "MP3EncoderLAME.h"
#include <stdlib.h>  // for rand

using namespace liblame;

void dataCallback(uint8_t *mp3_data, size_t len) {
    Serial.print("mp3 generated with ");
    Serial.print(len);
    Serial.println(" bytes");
}

MP3EncoderLAME mp3(dataCallback);
AudioInfo info;
int16_t buffer[512];

void setup() {
    Serial.begin(115200);

    info.channels = 1;
    info.sample_rate = 44100;
    mp3.begin(info);

    Serial.println("writing...");

}

void loop() {
    Serial.println("writing 512 samples of random data");
    for (int j=0;j<512;j++){
        buffer[j] = (rand() % 100) - 50;         
    }
    mp3.write(buffer, 512);
}

```

### Installation

In Arduino, you can download the library as zip and call include Library -> zip library. Or you can git clone this project into the Arduino libraries folder e.g. with

```
cd  ~/Documents/Arduino/libraries
git clone pschatzmann/arduino-liblame.git

```

This project can also be built and executed on your desktop with cmake:

```
cd arduino-liblame
mkdir build
cd build
cmake ..
make
```

### Changes to the Original Code
The initial restructured code was working prefectly with my [Arduino Simulator](https://github.com/pschatzmann/Arduino-Emulator) on the desktop, but as soon I as I deployed it on an ESP32 it was crashing because of different reasons:

- The memory structures were partly too big 
- Long arrays were allocated on the stack

So I needed to adjust the original code quite a bit. You can activate some micro processor specific functionaitly in config.h. 

```
// use dynamically precalculated log table
#define USE_FAST_LOG 0

// use precalculated log table as const -> in the ESP32 this will end up in flash memory
#define USE_FAST_LOG_CONST 1

// Avoid big memory allocations in replaygain_data
#define USE_MEMORY_HACK 1

// Not all microcontroller support vararg methods: alternative impelemtation of logging using the preprocessor
#define USE_LOGGING_HACK 1

// Print debug and trace messages
#define USE_DEBUG 0

// Print memory allocations and frees
#define USE_DEBUG_ALLOC 0

// The stack on microcontrollers is very limited - use the heap for big arrays instead of the stack! 
#define USE_STACK_HACK 1
```


### Documentation

The [Class Documentation can be found here](https://pschatzmann.github.io/arduino-liblame/html/annotated.html)

