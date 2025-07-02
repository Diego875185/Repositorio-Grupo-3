```cpp
#include <Arduino.h>
#include <esp_now.h>
#include <WiFi.h>

// Estructura de datos recibida por ESP-NOW
typedef struct struct_message {
    int valor; // Dato que se espera recibir (por ejemplo, una señal de activación)
} struct_message;

struct_message datosRecibidos; // Variable global para almacenar los datos recibidos

// Variables para el control del motor
unsigned long motorOnMillis = 0; // Marca de tiempo cuando se activó el motor
bool motorOn = false;            // Estado del motor (true = encendido, false = apagado)

/**
 * Callback de ESP-NOW.
 * Esta función se ejecuta automáticamente cada vez que se reciben datos por ESP-NOW.
 * Si el valor recibido es igual a 1381125955, activa el motor conectado al pin 8.
 */
void OnDataRecv(const uint8_t * mac, const uint8_t *incomingData, int len) {
    memcpy(&datosRecibidos, incomingData, sizeof(datosRecibidos)); // Copia los datos recibidos a la estructura
    Serial.print("Datos recibidos por ESP-NOW: ");
    Serial.println(datosRecibidos.valor);

    // Solo activa el motor si el valor recibido es 1381125955
    if (datosRecibidos.valor == 1381125955) {
        digitalWrite(8, LOW); // Activa el motor (asumiendo que el motor se activa con LOW)
        motorOn = true;       // Marca el motor como encendido
        motorOnMillis = millis(); // Guarda el tiempo de activación
    }
}

/**
 * setup()
 * Inicializa el sistema:
 * - Configura el pin del motor como salida y lo apaga.
 * - Inicializa la comunicación serie para depuración.
 * - Configura el WiFi en modo estación (requisito para ESP-NOW).
 * - Inicializa ESP-NOW y registra el callback de recepción.
 */
void setup() {
    Serial.begin(9600);

    // Inicializa el pin del motor
    pinMode(8, OUTPUT);
    digitalWrite(8, HIGH); // Apaga el motor al inicio (asumiendo activo en LOW)

    // Inicializa WiFi en modo estación para ESP-NOW
    WiFi.mode(WIFI_STA);

    // Inicializa ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("Error inicializando ESP-NOW");
        return;
    }
    esp_now_register_recv_cb(OnDataRecv); // Registra el callback de recepción

    Serial.println("Esperando datos por ESP-NOW...");
}

/**
 * loop()
 * Revisa constantemente si el motor debe apagarse.
 * Si el motor está encendido y ha pasado 1 segundo (1000 ms) desde su activación,
 * apaga el motor y actualiza el estado.
 */
void loop() {
    // Apaga el motor después de 1 segundo
    if (motorOn && (millis() - motorOnMillis >= 1000)) {
        digitalWrite(8, HIGH); // Apaga el motor (asumiendo activo en LOW)
        motorOn = false;       // Marca el motor como apagado
    }
}

```