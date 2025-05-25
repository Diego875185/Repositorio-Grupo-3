
# Taller Conectividad - Entregable

## Video
https://github.com/user-attachments/assets/e04d46ac-23f1-4e2a-8426-9f9c66f6da63

## Capturas de pantalla
![Imagen de WhatsApp 2025-05-24 a las 15 18 52_8e433ea7](https://github.com/user-attachments/assets/2f663322-8390-40ff-b4d8-64225821a3da)
![Imagen de WhatsApp 2025-05-24 a las 15 19 21_a030f30c](https://github.com/user-attachments/assets/b143564f-5856-4c43-a567-9aaec40d530e)



## Código `.ino` comentado

```cpp
#include <WiFiManager.h>  // Librería para gestionar conexiones WiFi con portal cautivo
//#include <WiFiClient.h>  // Comentado, no se usa directamente
#include <WebServer.h>    // Servidor web HTTP para ESP8266/ESP32
#include "index.h"        // Archivo que contiene el HTML de la página principal

WebServer server(80);     // Crea un servidor web en el puerto 80

// Función para manejar la raíz del servidor ("/")
void handleRoot() {
  String s = MAIN_page;  // MAIN_page es una cadena HTML definida en "index.h"
  server.send(200, "text/html", s);  // Envía la página HTML al navegador
}

// Función para manejar la ruta "/adc" y devolver el valor leído del pin analógico
void handleADC() {
  int a = 0;
  a = analogRead(A0);  // Lee el pin analógico A0 (conectado a un potenciómetro)
  a = map(a, 0, 4095, 0, 3300);  // Convierte el valor a milivoltios (0–3300 mV)
  String adcValue = String(a);  // Convierte el valor numérico a texto
  server.send(200, "text/plane", adcValue);  // Envía el valor como texto plano al cliente
}

void setup() {
  Serial.begin(115200);  // Inicializa el puerto serie a 115200 baudios para depuración
  WiFiManager wm;

  // Puedes descomentar la siguiente línea para borrar credenciales WiFi previas
  // wm.resetSettings();

  // Intenta conectarse automáticamente a WiFi con SSID y contraseña dados
  // Si falla, crea un portal cautivo con ese SSID y contraseña para configuración manual
  bool res = wm.autoConnect("ConnectAP", "1234567890");

  if (!res) {
    Serial.println("Fallo la conexion");  // Si falla la conexión
    // Aquí podrías reiniciar el dispositivo o intentar otra acción
  } else {
    Serial.println("Conectado :)");  // Éxito al conectar
  }

  // Asocia las rutas del servidor con las funciones correspondientes
  server.on("/", handleRoot);   // Ruta principal
  server.on("/adc", handleADC); // Ruta de lectura analógica

  server.begin();  // Inicia el servidor web
}
```

---

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
