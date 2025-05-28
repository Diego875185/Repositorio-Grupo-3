# Informe de Calibración del Sensor Flex

## Introducción

En este proyecto se implementó un sistema de lectura analógica utilizando un **sensor flex** conectado a un **ESP32**. La lectura del sensor se visualiza mediante una interfaz web alojada en el propio microcontrolador. El montaje incluye un **potenciómetro** como parte de un divisor de tensión, permitiendo un ajuste fino del voltaje aplicado al pin analógico **A0**.

## Propósito de la Calibración

La calibración tuvo como propósito verificar la correspondencia entre los valores digitales leídos por el ESP32 y los valores reales de voltaje proporcionados por el sensor. Esto se logró mediante la comparación con las mediciones realizadas con un **multímetro digital**, utilizado como instrumento de referencia.

## Procedimiento

1. Se conectó el sensor flex y el potenciómetro en configuración de divisor de voltaje.
2. Se realizaron mediciones de voltaje con un multímetro en dos posiciones extremas del sensor (sin doblar y completamente doblado).
3. Paralelamente, se leyeron los datos del pin A0 utilizando `analogRead()`.
4. Los valores leídos se convirtieron de la escala ADC del ESP32 (0 a 4095) a milivoltios (0 a 3300 mV) usando la función `map()`.

```cpp
a = analogRead(A0);
a = map(a, 0, 4095, 0, 3300); // Conversión a mV
```

## Datos Obtenidos

- **Lectura mínima (multímetro):** aproximadamente 2.8 V
- **Lectura máxima (multímetro):** aproximadamente 3.3 V

Estas mediciones se correspondieron bien con los valores leídos por el ESP32, una vez convertidos:

- **Valor mínimo en el ESP32 (mV):** alrededor de 2850 mV
- **Valor máximo en el ESP32 (mV):** alrededor de 3290 mV

Las pequeñas diferencias son atribuibles al ruido eléctrico, imprecisiones del ADC o variaciones en el divisor de tensión.

## Análisis

La conversión de datos y el comportamiento del sensor fueron consistentes. La escala ajustada en milivoltios permite integrar fácilmente los datos en una aplicación móvil, como la desarrollada en App Inventor. El sistema responde correctamente a la flexión del sensor dentro del rango útil de 2850 a 3290 mV.

## Conclusión

- La calibración fue exitosa: las lecturas digitales reflejan de forma fiable el comportamiento físico del sensor.
- El sistema es adecuado para aplicaciones que requieran monitoreo de flexión o presión leve mediante sensores resistivos.