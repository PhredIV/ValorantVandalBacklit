# ValorantVandalBacklit
Fun Gaming Wall Decour
#include <Adafruit_NeoPixel.h>

Adafruit_NeoPixel strip(22, 9);

int bPin = 8;
boolean currentBState = false;
boolean prevBState = false;

// int bPin2 = 9;
// boolean currentB2State = false;
// boolean prevB2State = false;

int spot = 0;

uint32_t purpleish = strip.Color(200, 0, 200);
uint32_t white = strip.Color(255, 255, 255);
uint32_t red = strip.Color(255, 0, 0);
uint32_t green = strip.Color(0, 255, 0);
uint32_t blue = strip.Color(0, 0, 255);

int interval = 5;

int soundPin = 10;

void setup() {
  Serial.begin(9600);
  strip.begin();
  strip.clear();

  pinMode(soundPin, OUTPUT);

  pinMode(bPin, INPUT_PULLUP);  //happens to be 8 rn
  digitalRead(bPin);
}

void loop() {

  prevBState = currentBState;  //my debounce things plus button
  currentBState = debounce(bPin, prevBState);

  // prevB2State = currentB2State;  //my debounce things 2nd button
  // currentB2State = debounce(bPin, prevBState);
  
  if (currentBState == true && prevBState == false) {  //toggel if statement
    spot++;
    Serial.println("BUTTON HAS BEEN PRESSED O MAH GERSH");
  }
  if (spot < 0) {
    spot = 4;
  }
  if (spot > 4) {
    spot = 0;
  }

  switch (spot) {
    case 0:  //on state; slow bouncing brightness (blue perhaps)
      Serial.println("first case");
      strip.clear();
      strip.setBrightness(100);
      break;
    case 1:  //won a game state (happy sound and fast running green lights, something about one light per win, another strip?)
      Serial.println("second kase");
      threeColors(red, green, blue, interval);
      break;
    case 2:  //lost game state; red blinking light, warden sound
      Serial.println("third case!");
      bouncingBrightness(purpleish, 10);
      digitalWrite(soundPin, HIGH);
      // delay(1000); // for setting the
      break;
    case 3:  //party mode
      Serial.println("4th case!");
      rainbowTimer();
      break;
    case 4:  //party mode
      Serial.println("5th case!");
      bouncingBrightness(purpleish, 10);
      break;
  }
}

uint32_t Wheel(byte WheelPos) {
  if (WheelPos < 85) {
    return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
  } else if (WheelPos < 170) {
    WheelPos -= 85;
    return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  } else {
    WheelPos -= 170;
    return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
}
void rainbowTimer() {  // modified from Adafruit example to make it a state machine
  static unsigned long startTime = millis();
  int interval = 20;
  static uint16_t j = 0;


  if (millis() - startTime >= interval) {
    for (int i = 0; i < strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel((i + j) & 255));
    }
    strip.show();
    j++;
    if (j >= 256) j = 0;
    startTime = millis();  // time for next change to the display
  }
}
void bouncingBrightness(uint32_t aColor, int anInterval) {
  static int brightness = 0;
  static boolean isIncreasing = true;
  static unsigned long startTime = millis();  //static bc it shouldnt reset every time
  unsigned long currentTime = millis();
  
  if (currentTime - startTime >= anInterval) {
    for (int index = 0; index < strip.numPixels(); index++) {
      strip.setPixelColor(index, aColor);
    }
    if (isIncreasing == true) {
      brightness++;
    }
    if (isIncreasing == false) {
      brightness--;
    }
    if (brightness >= 255) {
      isIncreasing = false;
    }
    if (brightness == 0) {
      isIncreasing = true;
    }
    strip.setBrightness(brightness);
    strip.show();
    startTime = millis();
  }
}
void threeColors(uint32_t c1, uint32_t c2, uint32_t c3, int anInterval) {
  static int state = 0;
  unsigned long currentTime = millis();
  static unsigned long startTime = millis();
  strip.setBrightness(200);

  if (currentTime - startTime >= anInterval) {
    state++;
    startTime = millis();
  }
  if (state == 0) {
    for (int index = 0; index < strip.numPixels(); index++) {

      //check module for color 1
      if (index % 3 == 0) {
        strip.setPixelColor(index, c1);
      }
      if (index % 3 == 1) {
        strip.setPixelColor(index, c2);
      }
      if (index % 3 == 2) {
        strip.setPixelColor(index, c3);
      }
      strip.show();
    }
  }
  if (state == 1) {
    for (int index = 0; index < strip.numPixels(); index++) {

      //check module for color 1
      if (index % 3 == 0) {
        strip.setPixelColor(index, c2);
      }
      if (index % 3 == 1) {
        strip.setPixelColor(index, c3);
      }
      if (index % 3 == 2) {
        strip.setPixelColor(index, c1);
      }
      strip.show();
    }
  }
  //last state
  if (state == 2) {
    for (int index = 0; index < strip.numPixels(); index++) {

      //check module for color 1
      if (index % 3 == 0) {
        strip.setPixelColor(index, c3);
      }
      if (index % 3 == 1) {
        strip.setPixelColor(index, c1);
      }
      if (index % 3 == 2) {
        strip.setPixelColor(index, c2);
      }
      strip.show();
    }
  }
  // Serial.println(state);
  if (state > 2) {
    state = 0;
  }
}
boolean debounce(int aButtonPin, boolean aPrevState) {
  boolean testState = digitalRead(aButtonPin);

  if (testState == true && aPrevState == false) {
    delay(20);
  }

  return testState;
}
