# GBB Forecast Downloader - Prognoza baterii dla Home Assistant

Pobiera prognozę stanu baterii z **GbbOptimizer** przez MQTT i wyświetla jako **linię przerywaną** na wykresie **ApexCharts**.
<img width="1158" height="1008" alt="obraz" src="https://github.com/user-attachments/assets/7e6d2477-d76e-4630-a835-b0d68d7e3699" />


![Home Assistant Package](https://img.shields.io/badge/Home%20Assistant-Package-blue?style=flat-square&logo=homeassistant)
![MQTT](https://img.shields.io/badge/MQTT-Protocol-purple?style=flat-square)

---

## 🚀 Szybka instalacja (Package)

### Krok 1: Włącz packages w Home Assistant

Dodaj do `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

### Krok 2: Skopiuj package

Skopiuj plik `packages/gbb_battery_forecast.yaml` do folderu `/config/packages/`:

```bash
mkdir -p /config/packages
cp packages/gbb_battery_forecast.yaml /config/packages/
```

### Krok 3: Dostosuj topiki MQTT (opcjonalnie)

Domyślne topiki:
- Request: `ha_gbb/dataserver/serverrequest`
- Response: `ha_gbb/dataserver/serverresponse`

Jeśli używasz innych, zmień je w pliku package (wyszukaj `topic:`).

### Krok 4: Restart Home Assistant

### Krok 5: Gotowe! 🎉

Package utworzy:
- `sensor.gbb_battery_forecast` - sensor z prognozą
- `input_button.gbb_forecast_refresh` - przycisk do ręcznego odświeżenia
- 3 automatyzacje (co 5 min, przy starcie HA, ręczny przycisk)

---

## 📊 Użycie w ApexCharts

Dodaj do swojego wykresu (sekcja `series:`):

```yaml
- entity: sensor.gbb_battery_forecast
  type: line
  yaxis_id: battery-soc
  name: Battery Forecast
  color: "#00BFFF"
  stroke_width: 2
  stroke_dash: 5  # Linia przerywana!
  show:
    extremas: false
    legend_value: true
    in_header: raw
  data_generator: |
    let forecast = entity.attributes.soc_forecast;
    if (typeof forecast === 'string') {
      try { forecast = JSON.parse(forecast); } catch (e) { return []; }
    }
    if (!forecast || !Array.isArray(forecast)) return [];
    return forecast.map(item => [new Date(item.timestamp).getTime(), item.soc]);
```

---

## 🧪 Testowanie

### Sprawdź czy sensor działa

1. **Developer Tools → States** → szukaj `sensor.gbb_battery_forecast`
2. Sprawdź atrybuty:
   - `soc_forecast` - lista prognoz (JSON)
   - `last_update` - czas ostatniej aktualizacji
   - `forecast_count` - liczba punktów

### Wymuś aktualizację

**Developer Tools → Services:**

```yaml
service: mqtt.publish
data:
  topic: "ha_gbb/dataserver/serverrequest"
  payload: '{"Operation": "BatteryForecast_GetChartData"}'
```

### Sprawdź odpowiedź MQTT

**Developer Tools → MQTT** → Subskrybuj: `ha_gbb/dataserver/serverresponse`

---

## 🔧 Rozwiązywanie problemów

| Problem | Rozwiązanie |
|---------|-------------|
| Sensor jest `unknown` | Sprawdź czy MQTT bridge działa i topiki są poprawne |
| Brak danych w `soc_forecast` | Sprawdź czy GbbOptimizer zwraca dane (subskrybuj topic odpowiedzi) |
| Wykres nie pokazuje prognozy | Upewnij się, że `yaxis_id: battery-soc` jest zdefiniowany |
| Automatyzacje nie działają | Sprawdź logi: **Ustawienia → System → Logi** |

---

## 📝 Format danych

### Atrybut `soc_forecast`

```json
[
  {"timestamp": "2024-01-15T10:00:00", "soc": 75.5},
  {"timestamp": "2024-01-15T11:00:00", "soc": 68.2},
  {"timestamp": "2024-01-15T12:00:00", "soc": 82.1}
]
```

### Odpowiedź z GbbOptimizer

Package parsuje odpowiedź `BatteryForecast_GetChartData` i wyciąga:
- `Day` - data
- `Hour` - godzina (0-23)
- `StartBattery_Perc` - SOC na początku godziny

---

## 📚 Odnośniki

- [GbbOptimizer Manual - MQTT API](https://gbboptimizer10.gbbsoft.pl/Manual?Filters.PageNo=19)
- [ApexCharts Card](https://github.com/RomRider/apexcharts-card)
- [Home Assistant Packages](https://www.home-assistant.io/docs/configuration/packages/)

---

## 📄 Licencja

MIT License - używaj dowolnie!
