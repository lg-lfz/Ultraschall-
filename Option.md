## JSN-SR04T – ESP32 mit 5V (Option B)

### Warum Option B?

- Stärkeres Ultraschallsignal durch 5V → bessere Reichweite
- Empfehlenswert bei großen Tanks (> 2m)
- ECHO-Pin gibt 5V aus → Spannungsteiler nötig zum Schutz des ESP32

---

### Spannungsteiler (ECHO-Pin)

```
ECHO ──┬── 1kΩ ──┬── GPIO 18
                 │
                2kΩ
                 │
                GND
```

> 5V × (2kΩ / (1kΩ + 2kΩ)) = **3.33V** → sicher für ESP32

---

### Verdrahtung ESP32

| JSN-SR04T | | ESP32 |
|-----------|--|-------|
| VCC | → | 5V (VIN) |
| GND | → | GND |
| TRIG | → | GPIO 5 |
| ECHO | → | 1kΩ → 2kΩ → GND, Mitte an GPIO 18 |

---

### Benötigte Bauteile

| Bauteil | Wert |
|---------|------|
| Widerstand R1 | 1kΩ |
| Widerstand R2 | 2kΩ |

> 💡 Alternativ: 2× 1kΩ in Reihe als R1 (2kΩ) und 1× 1kΩ als R2 — falls nur 1kΩ Widerstände vorhanden.

---

### Code

Identisch zu Option A — nur Pi
