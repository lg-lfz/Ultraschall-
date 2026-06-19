## JSN-SR04T – Verdrahtung & Code

### Verdrahtung Arduino Uno

| JSN-SR04T | Arduino Uno |
|-----------|-------------|
| VCC | 5V |
| GND | GND |
| TRIG | D9 |
| ECHO | D10 |

> ⚠️ Arduino Uno gibt 5V auf dem ECHO-Pin zurück — beim Uno kein Problem,
> da beide mit 5V arbeiten.

---

### Verdrahtung ESP32

| JSN-SR04T | ESP32 |
|-----------|-------|
| VCC | 3.3V |
| GND | GND |
| TRIG | GPIO 5 |
| ECHO | GPIO 18 |

> ✅ Bei 3.3V Betrieb direkt kompatibel — kein Spannungsteiler nötig!

---

### Schematisch

```
          ┌─────────────┐
          │  JSN-SR04T  │
          │  Platine    │
Uno D9 ───┤ TRIG        │
Uno D10───┤ ECHO        ├──── Kabel ──── [Wasserdichter Sensor]
5V ───────┤ VCC         │
GND ──────┤ GND         │
          └─────────────┘
```

---

### Code

**Keine externe Bibliothek nötig.**

```cpp
// ===== Pin Definitions =====
// Uno:   TRIG=9,  ECHO=10
// ESP32: TRIG=5,  ECHO=18

#define TRIG_PIN   9
#define ECHO_PIN   10
#define TANK_HEIGHT 100.0  // Tank height in cm → adjust!

void setup() {
  Serial.begin(9600);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  Serial.println("JSN-SR04T ready.");
  Serial.println("---");
}

float measureDistance() {
  // Send trigger pulse
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure echo (30ms timeout)
  long duration = pulseIn(ECHO_PIN, HIGH, 30000);

  // Convert to cm
  return duration * 0.0343 / 2.0;
}

void loop() {
  // Average over 5 measurements
  float total = 0;
  int valid = 0;

  for (int i = 0; i < 5; i++) {
    float d = measureDistance();
    if (d >= 20 && d <= 600) {
      total += d;
      valid++;
    }
    delay(50);
  }

  if (valid == 0) {
    Serial.println("Distance   : out of range");
    Serial.println("Water level: unknown");
    Serial.println("Fill level : unknown");
  } else {
    float distance   = total / valid;
    float waterLevel = TANK_HEIGHT - distance;
    float percentage = (waterLevel / TANK_HEIGHT) * 100.0;

    // Clamp percentage
    if (percentage < 0) percentage = 0;
    if (percentage > 100) percentage = 100;

    Serial.print("Distance   : ");
    Serial.print(distance, 1);
    Serial.println(" cm");

    Serial.print("Water level: ");
    Serial.print(waterLevel, 1);
    Serial.println(" cm");

    Serial.print("Fill level : ");
    Serial.print(percentage, 1);
    Serial.println(" %");
  }

  Serial.println("---");
  delay(500);
}
```

> 💡 Für den ESP32 einfach `TRIG_PIN` auf `5` und `ECHO_PIN` auf `18` ändern — Rest bleibt gleich.

---

### Example Serial Monitor Output

```
JSN-SR04T ready.
---
Distance   : 63.2 cm
Water level: 36.8 cm
Fill level : 36.8 %
---
Distance   : 63.1 cm
Water level: 36.9 cm
Fill level : 36.9 %
---
Distance   : out of range
Water level: unknown
Fill level : unknown
---
```

---

### Key Parameters

| Parameter | Value |
|-----------|-------|
| Measuring range | 20 cm – 600 cm |
| Accuracy | ~3–5 mm |
| IP rating (sensor) | IP67 |
| Averaging | 5 measurements per reading |
| Update interval | ~500 ms |
