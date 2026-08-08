---
name: clima
description: Obtiene la información del clima localmente desde la terminal, sin API key. Autodetecta tu ubicación por IP o acepta una ciudad concreta. Úsala cuando el usuario pida "el clima", "el tiempo", "temperatura", "pronóstico", "¿va a llover?", "cómo está el clima en <ciudad>", o invoque /clima.
---

# Skill: Clima

Consulta el clima local o de cualquier ciudad usando servicios gratuitos que
**no requieren API key**. Todo se hace con `curl` desde la terminal.

## Cuándo usarla

- El usuario pide el clima / el tiempo / la temperatura / el pronóstico.
- Pregunta si va a llover, cuánto grado hace, etc.
- Menciona una ciudad y quiere sus condiciones ("clima en Madrid").
- Invoca `/clima` (opcionalmente con una ciudad como argumento).

## Ciudad por defecto

Si el usuario **no indica ninguna ciudad**, usa **Puebla** (Puebla, México).
Solo autodetecta por IP si el usuario lo pide explícitamente ("dónde estoy",
"mi ubicación actual").

## Cómo obtener el clima

### 1. Clima actual (rápido)

```bash
curl -s "wttr.in/Puebla?format=%l:+%c+%t+(sensación+%f),+humedad+%h,+viento+%w&m"
```

- `%l` ubicación, `%c` icono, `%t` temperatura, `%f` sensación térmica,
  `%h` humedad, `%w` viento. `&m` fuerza unidades métricas (°C).
- Para **otra ciudad**, cámbiala en la ruta (codifica los espacios con `+`):

```bash
curl -s "wttr.in/Ciudad+de+Mexico?format=%l:+%c+%t+(sensación+%f),+humedad+%h,+viento+%w&m"
```
- Para **autodetectar por IP** (solo si el usuario lo pide), deja la ruta vacía:
  `curl -s "wttr.in/?format=..."`.

### 2. Pronóstico visual de 3 días (panel ASCII)

```bash
curl -s "wttr.in/Madrid?m&lang=es"
```

Añade `?0` para solo el día actual, `?1` un día, `?2` dos días. `&lang=es`
devuelve las descripciones en español.

### 3. Datos estructurados (JSON, vía Open-Meteo)

Útil cuando necesitas los números para procesarlos. Requiere primero
geocodificar la ciudad a lat/lon:

```bash
# Paso A: obtener coordenadas de la ciudad
curl -s "https://geocoding-api.open-meteo.com/v1/search?name=Sevilla&count=1&language=es&format=json"

# Paso B: clima actual con esas coordenadas (ejemplo Sevilla)
curl -s "https://api.open-meteo.com/v1/forecast?latitude=37.39&longitude=-5.99&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,wind_speed_10m&timezone=auto"
```

### 4. Aviso de lluvia (comprobar siempre)

En **cada** consulta de clima, comprueba la probabilidad máxima de lluvia de hoy
y, si es alta, añade un aviso destacado en la respuesta:

```bash
curl -s "wttr.in/Puebla?format=j1" | python -c "import sys,json; h=json.load(sys.stdin)['weather'][0]['hourly']; print(max(int(x['chanceofrain']) for x in h))"
```

Devuelve un número (0–100) = probabilidad máxima de lluvia hoy. Cambia `Puebla`
por la ciudad consultada. Umbral sugerido: **≥ 50 %** → aviso.

## Cómo responder al usuario

1. Ejecuta el comando adecuado (rápido → opción 1; pronóstico → opción 2;
   necesitas los datos crudos → opción 3) usando **Puebla** si no se indicó ciudad.
2. Ejecuta también la **opción 4** para saber la probabilidad de lluvia de hoy.
3. Resume en **español**, de forma breve y clara: temperatura, sensación
   térmica, condición (soleado, nublado, lluvia…), humedad y viento.
4. **Aviso de lluvia:** si la probabilidad es ≥ 50 %, añade una línea destacada
   al inicio, p. ej. `☔ Aviso: alta probabilidad de lluvia hoy (X%), lleva paraguas.`
   Si es baja, puedes mencionarlo en una frase corta o omitirlo.
5. Si `curl` falla (sin red o servicio caído), dilo claramente y sugiere
   reintentar; no inventes datos del clima.

## Notas

- Estos servicios son gratuitos y de uso comunitario: no abuses con muchas
  peticiones seguidas. wttr.in puede limitar por IP si se consulta en exceso.
- Todo funciona sin claves ni configuración: solo hace falta `curl` y conexión
  a internet.
