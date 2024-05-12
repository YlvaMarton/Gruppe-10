// ----- legger til kodebiblioteket Adafruit Neopixel for å kunne kontrollere LED-strip -----
#include <Adafruit_NeoPixel.h>
// ----- LED-strip "nervesignal" eller "kroppen til ninjaen" -----
#define LED_PIN_A1 A1
#define LED_COUNT_1 54
#define MIDPOINT 36
Adafruit_NeoPixel strip(LED_COUNT_1, LED_PIN_A1, NEO_GRB + NEO_KHZ800);
// ----- LED-strip "poeng" -----
#define LED_PIN_A2 A2
#define LED_COUNT_2 21
Adafruit_NeoPixel strip2(LED_COUNT_2, LED_PIN_A2, NEO_GRB + NEO_KHZ800);

// ----- starter Millis -----
unsigned long lastTimeButtonStateChanged;   
unsigned long debounceDuration = 50;
unsigned long event_time;

// ----- definerer knapper og leds til ulike pins -----
const int startButton = 13;                    // startknapp til digital pin 13
const int startLed = 7;                        // startknappled til digital pin 7
const int buttons[] = {8, 9, 10, 11, 12};      // array av digitale pins knappene er i på arduinoen
const int leds[] = {2, 3, 4, 5, 6};            // array av digitale pins led-lysene er i på arduinoen

// ----- definerer noen globale startvariabler -----
int buttonState;
int lastButtonState;                            // må ha lastButtonState også for å ta hensyn til rippelverdien
int ledState = HIGH;                            // lyset til å være på
int rounds = 21;                                // poeng variabel, må starte fra 21 fordi den er loddet motsatt av den retningen den skal fylles med poeng
int timeSpent;                                  //tidBrukt
int maxTime = 3000;                             // Maksimum tid
int minTime = 250;                              // Minimum tid
int thisButton;                                 //tilfeldig
int lastButton = 0;                             //forrigeTilfeldig
int consecutiveButtonReleases = 0; 

// ----- setup -----
void setup() {
  Serial.begin(9600);                           // starter seriell monitoren
  // ----- starter LED-strip "nervesignal" -----
  strip.begin();                                
  strip.show();
  strip.setBrightness(100);                     // setter lysstyrken til ledstripen til 100
  // ----- starter LED-strip "poeng" -----
  strip2.begin();                               
  strip2.show();
  strip2.setBrightness(100);                    // setter lysstyrken til ledstripen til 100
  // ----- setter leds til output -----
  for (int led : leds) {                        // for løkke for å sette alle ledlysene til output
    pinMode(led, OUTPUT);
  }
  // ----- setter bryterne til output -----
  for (int button : buttons) {                  // for løkke for å sette alle knappene til inputs
    pinMode(button, INPUT);
  }
  // ----- setter startknappen og tilhørende LED til hhv. input og output -----
  pinMode(startButton, INPUT);
  pinMode(startLed, OUTPUT);

  lastButtonState = digitalRead(startButton);       // leser av verdien for startknappen i begynnelsen, må gjøre dette for å ta hensyn til rippelverdien
  lastTimeButtonStateChanged = millis();

  timeSpent = millis() - event_time;                // tid fra koden starter til oppdatert tid

  randomSeed(analogRead(0));                        // initialiserer den tilfeldige tallgeneratoren med analog lesning
}

// ----- loopen, kode som kjøres om og om igjen -----
void loop() {                                     
  digitalWrite(7, HIGH);                                  // startknapp lyser
  for (int i = 0; i < 55; i++) {                          // alle led lysene, men ikke strip2 lyser før startknappen trykkes
    strip.setPixelColor(i, strip.Color(100, 0, 0));
    strip.show();
  }

  if (millis() - lastTimeButtonStateChanged >= debounceDuration) {                 // hvis knappen blir trykket               
    buttonState = digitalRead(startButton);                                        // oppdaterer verdien
    if (buttonState != lastButtonState) {                                          // hvis knappen har endret verrdi så skal...
      lastTimeButtonStateChanged = millis();                                                  // oppdatere verdi
      lastButtonState = buttonState;                                                          // oppdatere verdi
      if (buttonState == HIGH) {                                                   // hvis knappen var av og blir på
        if (ledState == HIGH) {                                                    // skal oppdatere tilstanden til å være LOW
          ledState = LOW;
        }
        else {                                                    
          ledState = HIGH;                                                         // skal oppdatere tilstanden til å være HIGH
        }
        digitalWrite(startLed, ledState);                                          // skal skru av/på startlyset    
      }

      // hvis startknappen blir trykket på så...

      for (int i = 0; i < 55; i++) {                                    // skal alle led lysene slukkes
        strip.setPixelColor(i, strip.Color(0, 0, 0));
        strip.show();
      }

      delay(500);                                                       // et delay før minispillene starter

      for (int numGames = 0; numGames < 20; numGames++) {                          // for løkke for at spillet skal vare i 20 minispill
        choosingWhichButton();                                                     // velger et random spill

        event_time = millis();

        if (thisButton == 2) {                // Spill 1, venstre bein
          nerveSignalVBG();                   // hvite lyser som går til venstrebein           
          game(0, 0, nerveSignalVB);          // knapp og led lys nr. 1 (index 0), tilhørende nervesignal
        }
        if (thisButton == 3) {                // Spill 2, venstre arm
          nerveSignalVAG();                   // hvite lyser som går til venstrearm   
          game(1, 1, nerveSignalVA);          // knapp og led lys nr. 2 (index 1), tilhørende nervesignal
        }
        if (thisButton == 4) {                // Spill 3, hodet
          nerveSignalHG();                    // hvite lyser som går til hodet   
          game(2, 2, Hodet);                  // knapp og led lys nr. 3 (index 2), tilhørende nervesignal
        }
        if (thisButton == 5) {                // Spill 4, høyre arm
          nerveSignalHBG();                   // hvite lyser som går til høyre bein
          game(3, 3, nerveSignalHB);          // knapp og led lys nr. 4 (index 3), tilhørende nervesignal
        }
        if (thisButton == 6) {                // Spill 5, høyre arm
          nerveSignalHAG();                   // hvite lyser som går til venstre arm
          game(4, 4, nerveSignalHA);          // knapp og led lys nr. 5 (index 4), tilhørende nervesignal
        }
        
        strip2.show();                        // poengene/de poengene man har bommet på lyser
        digitalWrite(thisButton, LOW);
        delay(600);                           // pause før neste minispill går


      }
      endOfGameLights();                      // diskolys på nervesignalene for å signalisere at spillet er over
      delay(5000);                            // 5 sek på å lese scoren før den forsvinner
      strip2.fill((0, 0, 0), 0, 21);          // nullstiller poeng-ledstripen
      strip2.show();                          // slår av lysene til poeng-stripen
      maxTime = 3000;                         // oppdaterer variabelen til det som de var før spillet startet
      rounds = 21;                            // --''--
      lastButtonState = LOW;                  // --''--
      ledState = HIGH;                        // --''--
    }
  }
}



void game(int buttonIndex, int ledIndex, void (*nerveSignal)()) {                            // generell kode for minispillene
  rounds--;                                                                                  // oppdaterer runden man er på, skal bli en mindre for hver gang minispillet kjører 

  while (millis() - event_time < maxTime) {                                                  // så lenge tiden er mindre enn makstiden
    int buttonState = digitalRead(buttons[buttonIndex]);                                     // oppdaterer den knappen som er indeksert
    digitalWrite(leds[ledIndex], HIGH);                                                      // skrur på tilhørende lys                                                     
    if (buttonState == HIGH) {                                                               // om den knappen blir trykket på så...
      digitalWrite(leds[ledIndex], LOW);                                                     // skal lyset slutte å lyse
      nerveSignal();                                                                         // nervesignalet skal gå
      strip2.setPixelColor(rounds, strip2.Color(255, 0, 0));                                 // runde led stripen blir fyllt med ett rødt lys som er veldig sterkt for å signalisere ett poeng

//------ kode for å tilpassse vansklighetsnivået til hver enkelt spiller --------------
//------ har store intervaller for å korte ned veldig for å luke ut de som er velkdig gode også blir intervallene mindre -----------
      if ((timeSpent < maxTime - 1500) && (maxTime > minTime)) {                             // hvis man trykker på knappen fortere enn makstid - 1500                           
        maxTime -= 1000;                                                                     // skal makstiden bli mindre med 1000
      } else if ((timeSpent < maxTime - 1000) && (maxTime > minTime)) {                      // hvis man trykker på knappen fortere enn makstid - 1000 
        maxTime -= 750;                                                                      // skal makstiden bli mindre med 750
      } else if ((timeSpent < maxTime - 500) && (maxTime > minTime)) {                       // hvis man trykker på knappen fortere enn makstid - 500  
        maxTime -= 75;                                                                       // skal makstiden bli mindre med 75
      } else if ((timeSpent < maxTime - 450) && (maxTime > minTime)) {                       // hvis man trykker på knappen fortere enn makstid - 450 
        maxTime -= 50;                                                                       // skal makstiden bli mindre med 50
      } else if ((timeSpent < maxTime - 350) && (maxTime > minTime)) {                       // hvis man trykker på knappen fortere enn makstid - 350 
        maxTime -= 25;                                                                       // skal makstiden bli mindre med 25
      } 
      break;                                                                                 // skal breake ut av  while løkka om man trykke på knappen 
    } else {                                                                                  
      strip2.setPixelColor(rounds, strip2.Color(5, 0, 0));                                   // hvis ikke så skal runden fylles med et svakt rødt lys for å signalisere at man ikke fikk poeng    
    }
  }
}



//---- generell kode for nervesignalene ----------
void nerveSignal(int start, int end, int increment) {           
  for (int i = start; i != end; i += increment) {               
    strip.setPixelColor(i, strip.Color(100, 0, 0));                 // setter farge til rød
    strip.setPixelColor(i - increment, strip.Color(0, 0, 0));       // skal sette farge til ingen ting på forrige led-stripe lys
    strip.show();                                                   // viser det som er satt
    delay(35);                                                      // pause mellom sånn at man ikke går så fort
    if (i == end - increment) {                                     // hvis man er på den siste før midtLed-en
      strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));        // skal midtLed-ens farge setter til rød
      strip.show();                                                 // viser det
      delay(35);                                                    // pause mellom sånn at man ikke går så fort
      strip.fill((0, 0, 0), start, end);                            // slukker alle led-ene
      strip.show();                                                 // viser det
      delay(35);
      strip.setPixelColor(MIDPOINT, strip.Color(0, 0, 0));          // slukker midtLed-lyset
      strip.show();                                                 // viser det
      delay(35);                                                    // pause før poeng-led lyser
    }
  }
}


void nerveSignalHA() {                      // for venstre arm, sett fra figuren på pleksiglasset sitt perspektiv
  nerveSignal(2, 12, 1);                    // fra 2 til 12 pga. to led sitter på baksiden for å lettere kunne lodde

  int groups[][4] = {                       // arrays for at nervesignat fra midten og ut til de andre lemmene
    {53, 13, 35, 37},
    {52, 14, 34, 38},
    {51, 15, 33, 39},
    {50, 16, 32, 40},
    {49, 31, 31, 41},                       // siden veien til hodet er kortere enn til armer og bein så setter vi lyset -
    {49, 18, 30, 42},                       // på baksiden av prototypen slik at vi slipper å lage to forløkker, ettersom -
    {49, 19, 29, 43},                       // arrayene må være av lik lengde for at forløkken skal fungere
    {49, 20, 28, 44},
    {49, 21, 27, 45},
    {49, 22, 26, 46}
  };

  for (int i = 0; i < 10; i++) {
    // Turn on LEDs to red
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(100, 0, 0));
    }
    strip.show();
    delay(35);

    // Turn off LEDs
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(0, 0, 0));
    }
    strip.show();
    delay(35);
  }
}





void nerveSignalHB() {                                                    // nerveSignal for høyre bein
  for (int i = 46; i > 36; i--) {                                         
    strip.setPixelColor(i, strip.Color(100, 0, 0));                       // tanken er lik som for nerveSignal bortsett fra med andre verdier
    strip.setPixelColor(i + 1, strip.Color(0, 0, 0));                     // siden den går motsatt vei må det bli motsatt så i+1 istedenofr i-1 som er på de andre
    strip.show();
    delay(35);
    if (i == 37) {
      strip.setPixelColor(37, strip.Color(100, 0, 0));
      strip.show();
      delay(35);
      strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));
      strip.show();
      delay(35);
      strip.fill(strip.Color(0, 0, 0), 35, 45);                    // slukker led lysene fra 31 til 38          // denne her funker ikke på den generelle koden og vi må derfor lage en egen for HB og VB istedenfor å bruke det generelle nerveSignalet
      strip.show();
      delay(0);
    }
  }
  

  int groups[][4] = {                                              // samme som den over, men for andre veier
    {11, 13, 35, 53},
    {10, 14, 34, 52},
    {9, 15, 33, 51},
    {8, 16, 32, 50},
    {7, 31, 31, 49},
    {6, 18, 30, 49},
    {5, 19, 29, 49},
    {4, 20, 28, 49},
    {3, 21, 27, 49},
    {2, 22, 26, 49}
  };

  for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(100, 0, 0));               
    }
    strip.show();
    delay(35);

    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(0, 0, 0));
    }
    strip.show();
    delay(35);
  }
  
}

void Hodet() {                                                                // nerve signal for hodet
  for (int i = 50; i < 54; i++) {
    strip.setPixelColor(i, strip.Color(100, 0, 0));
    strip.setPixelColor(i - 1, strip.Color(0, 0, 0));
    strip.show();
    delay(35);
  }
  strip.setPixelColor(53, strip.Color(0, 0, 0));
  strip.show();
  delay(35);

  strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));
  strip.show();
  delay(35);
  strip.setPixelColor(MIDPOINT, strip.Color(0, 0, 0));
  strip.show();
  delay(35);

  int groups[][4] = {                                                        // samme prinsipp, men andre veier
    {11, 13, 35, 37},
    {10, 14, 34, 38},
    {9, 15, 33, 39},
    {8, 16, 32, 40},
    {7, 31, 31, 41},
    {6, 18, 30, 42},
    {5, 19, 29, 43},
    {4, 20, 28, 44},
    {3, 21, 27, 45},
    {2, 22, 26, 46}
  };

  for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(100, 0, 0));               
    }
    strip.show();
    delay(35);
    // Turn off all LEDs in the group
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(0, 0, 0));
    }
    strip.show();
    delay(35);
  }
}

void nerveSignalVB() {                                                             // nerve signal for venstre bein
  for (int i = 22; i > 11; i--) {
    strip.setPixelColor(i, strip.Color(100, 0, 0));
    strip.setPixelColor(i + 1, strip.Color(0, 0, 0));
    strip.show();
    delay(35);
    if (i == 12) {
      strip.setPixelColor(12, strip.Color(0, 0, 0));
      strip.show();
      delay(35);
      strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));
      strip.show();
      delay(35);
      strip.fill(strip.Color(0, 0, 0), 10, 36); // slukker led-lysene fra 10 til 19    // denne her funker ikke på den generelle koden og vi må derfor lage en egen for HB og VB istedenfor å bruke det generelle nerveSignalet
      strip.show();
      delay(0);
      // nerveSignal2();
    }
  }
  strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));
  strip.show();
  delay(35);
  strip.setPixelColor(MIDPOINT, strip.Color(0, 0, 0));
  strip.show();
  delay(35);

  int groups[][4] = {                                                       // samme prinsipp som før, men andre veier
    {11, 53, 35, 37},
    {10, 52, 34, 38},
    {9, 51, 33, 39},
    {8, 50, 32, 40},
    {7, 49, 31, 41},
    {6, 49, 30, 42},
    {5, 49, 29, 43},
    {4, 49, 28, 44},
    {3, 49, 27, 45},
    {2, 49, 26, 46}
  };

  for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(100, 0, 0));               
    }
    strip.show();
    delay(35);
    // Turn off all LEDs in the group
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(0, 0, 0));
    }
    strip.show();
    delay(35);
  }
}
void nerveSignalVA() {                                                         // nerve signal for venstre arm, fortsatt sett fra ninjaen (prototypen) sitt perspektiv
  nerveSignal(26, 36, 1);

  strip.setPixelColor(MIDPOINT, strip.Color(100, 0, 0));
  strip.show();
  delay(35);
  strip.setPixelColor(MIDPOINT, strip.Color(0, 0, 0));
  strip.show();
  delay(35);

  int groups[][4] = {                                                         // amme prinsipp som før, men med andre veier
    {11, 13, 53, 37},
    {10, 14, 52, 38},
    {9, 15, 51, 39},
    {8, 16, 50, 40},
    {7, 31, 49, 41},
    {6, 18, 49, 42},
    {5, 19, 49, 43},
    {4, 20, 49, 44},
    {3, 21, 49, 45},
    {2, 22, 49, 46}
  };

  for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(100, 0, 0));               // setter fargen til rød
    }
    strip.show();
    delay(35);
    // Turn off all LEDs in the group
    for (int j = 0; j < 4; j++) {
      strip.setPixelColor(groups[i][j], strip.Color(0, 0, 0));
    }
    strip.show();
    delay(35);
  }
}



//------- signal før knappen skal begynne å lyse ----------

void pulse(int start, int end, int step) {                               // generell kode
  for (int i = start; i != end; i += step) { 
    strip.setPixelColor(i, strip.Color(100, 100, 100));                  // setter fargen til rød
    strip.setPixelColor(i - step, strip.Color(0, 0, 0));                 // slukker forrige pixel
    strip.show();
    delay(10);
  }
  strip.setPixelColor(end - step, strip.Color(0, 0, 0));                 // slukker alle pixler
  strip.show();
}

void nerveSignalHAG() {                                            // for høyre arm
  pulse(12, 2, -1); 
  }
                               
void nerveSignalHBG() {                                            // for høyre bein 
  pulse(36, 46, 1); 
  }
  
void nerveSignalHG() {                                            // for hodet 
  pulse(54, 50, -1); 
  }
  
void nerveSignalVBG() {                                            // for venstre bein 
  pulse(13, 22, 1); 
  }
void nerveSignalVAG() {                                            // for venstre arm 
  pulse(36, 26, -1); 
  }






//-------- diskolys for å signalisere at spillet er over -----------
void endOfGameLights() {
  for (int s = 0; s < 15; s++) {                                    // varigheten til diskolyset
    strip.fill((100, 0, 0), 0, 54);                                 // fyller med rød
    for (int i = 1; i < 55; i += 2) {                               // fyller annenhver pixel til nervesignal-led-stripene, oddetall
      strip.setPixelColor(i, strip.Color(100, 0, 0));               
    }
    strip.show();
    delay(100);
    strip.fill((100, 0, 0), 0, 54);
    strip.show();
    for (int n = 0; n < 55; n += 2) {                               // fyller annenhver pixel til nervesignal-led-stripene, partall
      strip.setPixelColor(n, strip.Color(100, 100, 100));           // fyller med hvit
    }
    strip.show();
    delay(100);
    strip.fill((100, 0, 0), 0, 54);
    strip.show();
  }
  strip.fill((0, 0, 0), 0, 54);                                     // slukker led-stripen
  strip.show();
}



// --------- velger en tilfeldig knapp. hvis det blir samme tall to ganger på rad, generer den et nytt tilfeldig tall. ----------
void choosingWhichButton() {
  thisButton = leds[random(0, sizeof(leds) / sizeof(int))];          // en tilfeldig led skal lyse
  while (thisButton == lastButton) {                                 // så lenge ny knapp er lik den forrige knappen 
    thisButton = leds[random(0, sizeof(leds) / sizeof(int))];        // skal den generere en ny tilfeldig knapp
  }
  lastButton = thisButton;                                           // oppdatere forrige knapp til den nye
}


//------ vi krediterer biblioteket Adafruit_NeoPixel.h ------------

//------ vi vil kreditere alle gruppemedlemene ---------------
// Helene Unander Netland
// Madeleine Overdale
// Sara Sandnesmo Næssan
// Ylva Unni Falk Marton
