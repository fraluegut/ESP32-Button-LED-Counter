# ESP32-Button-LED-Counter

A project to count button presses with an ESP32, including an always-on LED and deep sleep functionality.

## Components
- ESP32
- 2 LEDs
- 2 resistors (220 ohms)
- Push button (two-pin)
- Breadboard and connecting wires

## Schematic

![Schematic](path/to/your/schematic.png)

### Connections
- **Status LED:** 
  - Anode (long leg) -> Pin 2 of ESP32 (through a 220-ohm resistor)
  - Cathode (short leg) -> GND
- **Push button:**
  - One terminal -> Pin 4 of ESP32
  - Other terminal -> GND
- **Action LED:**
  - Anode (long leg) -> Pin 15 of ESP32 (through a 220-ohm resistor)
  - Cathode (short leg) -> GND

## Code

```cpp
#include <driver/rtc_io.h>

const int statusLedPin = 2;  // Pin del LED de estado
const int buttonPin = 4;     // Pin del pulsador
const int actionLedPin = 15; // Pin del LED adicional
int buttonPressCount = 0;    // Contador de pulsaciones
RTC_DATA_ATTR int bootCount = 0; // Contador de arranques (persistente)

void setup() {
  Serial.begin(115200);
  delay(1000); // Pequeño retraso para la consola serie

  pinMode(statusLedPin, OUTPUT);
  pinMode(actionLedPin, OUTPUT);

  // Enciende el LED de estado
  digitalWrite(statusLedPin, HIGH);

  // Configura el pulsador como wake-up source
  esp_sleep_enable_ext0_wakeup(GPIO_NUM_4, 0); // 0 significa que el botón está conectado a GND

  // Incrementa el contador de arranques
  bootCount++;
  Serial.print("Boot number: ");
  Serial.println(bootCount);

  // Comprueba la razón del wake-up
  if (esp_sleep_get_wakeup_cause() == ESP_SLEEP_WAKEUP_EXT0) {
    Serial.println("Wakeup caused by external signal using RTC_IO");
    buttonPressCount++;
    Serial.println("Pulsado");
    Serial.print("Conteo de pulsaciones: ");
    Serial.println(buttonPressCount);

    // Enciende el LED adicional durante 2 segundos para indicar la pulsación
    digitalWrite(actionLedPin, HIGH);
    delay(2000);
    digitalWrite(actionLedPin, LOW);
  }

  // Configura el GPIO para el wake-up
  rtc_gpio_pullup_en(GPIO_NUM_4);
  rtc_gpio_pulldown_dis(GPIO_NUM_4);

  // Imprime un mensaje indicando que va a entrar en modo Deep Sleep
  Serial.println("Entering deep sleep...");
  Serial.flush(); // Asegúrate de que todos los datos se envíen antes de dormir

  // Entra en modo Deep Sleep
  esp_deep_sleep_start();
}

void loop() {
  // No es necesario poner nada en el loop ya que el ESP32 estará en modo Deep Sleep
}
