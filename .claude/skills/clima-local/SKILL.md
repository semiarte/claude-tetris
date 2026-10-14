---
name: clima-local
description: Use when the user asks for the current weather, temperature, humidity, or wind locally (e.g. "qué clima hace", "cómo está el tiempo", "dame el clima de aquí") without specifying a city.
---

# Clima Local

## Overview

Consulta el clima actual detectando la ubicación automáticamente por IP, sin necesidad de API key. Usa `ip-api.com` para geolocalizar y `open-meteo.com` para el pronóstico.

## Cuándo usar

- El usuario pide el clima/tiempo sin dar una ciudad concreta ("qué clima hace ahora").
- Si el usuario SÍ especifica una ciudad distinta, pásale `latitude`/`longitude` de esa ciudad en vez de la IP detectada (puedes geocodificar con `https://geocoding-api.open-meteo.com/v1/search?name=<ciudad>`).

## Procedimiento

1. Obtener lat/lon por IP:

```bash
curl -s "http://ip-api.com/json/?fields=status,lat,lon,city,regionName,country"
```

```powershell
Invoke-RestMethod "http://ip-api.com/json/?fields=status,lat,lon,city,regionName,country"
```

2. Consultar el clima actual con esas coordenadas:

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=<LAT>&longitude=<LON>&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code&timezone=auto"
```

```powershell
Invoke-RestMethod "https://api.open-meteo.com/v1/forecast?latitude=<LAT>&longitude=<LON>&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code&timezone=auto"
```

3. Traducir `weather_code` (WMO) a una descripción y responder al usuario con ciudad, temperatura, humedad, viento y descripción.

## Tabla WMO weather_code (resumida)

| Código | Descripción |
|---|---|
| 0 | Despejado |
| 1-3 | Poco nublado / parcialmente nublado / nublado |
| 45, 48 | Niebla |
| 51, 53, 55 | Llovizna (ligera/moderada/densa) |
| 61, 63, 65 | Lluvia (ligera/moderada/fuerte) |
| 71, 73, 75 | Nieve (ligera/moderada/fuerte) |
| 80, 81, 82 | Chubascos |
| 95 | Tormenta |

## Notas

- Ambas APIs son gratuitas y no requieren registro ni API key.
- La ubicación por IP es aproximada (nivel ciudad), no exacta.
- Si `ip-api.com` falla (rate limit, sin red), informar al usuario en vez de inventar datos.
