```cpp
#include <Arduino.h>
#include <esp_now.h>
#include <WiFi.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

// UUIDs para el servicio y características UART BLE (protocolo estándar para comunicación serie sobre BLE)
#define SERVICE_UUID           "6E400001-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_RX "6E400002-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_TX "6E400003-B5A3-F393-E0A9-E50E24DCCA9E"

// Dirección MAC del receptor ESP-NOW (modificar por la dirección del dispositivo receptor real)
uint8_t broadcastAddress[] = {0x28, 0x37, 0x2F, 0x6A, 0xB6, 0xB0};

// Variables globales para la conexión BLE
BLEServer* pServer = nullptr;                // Puntero al servidor BLE
BLECharacteristic* pTxCharacteristic = nullptr; // Característica para enviar datos (TX)
bool deviceConnected = false;                // Estado de conexión BLE
bool oldDeviceConnected = false;             // Estado anterior de conexión BLE

// Variables para el control del LED de advertencia
unsigned long ledOnMillis = 0;               // Marca de tiempo cuando se encendió el LED
bool ledOn = false;                          // Estado del LED

// Configuración del sensor flex
const int flexPin = A0;                      // Pin analógico donde está conectado el sensor flex
const int umbral = 30;                       // Umbral para activar la advertencia (ajustar según el sensor)

unsigned long previousMillis = 0;            // Marca de tiempo para el temporizador principal
const long interval = 1000;                  // Intervalo de muestreo/envío (1 segundo)

/**
 * Callback de ESP-NOW: indica si el mensaje fue enviado correctamente.
 * Se ejecuta automáticamente después de cada envío por ESP-NOW.
 */
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    Serial.print("Estado del envío: ");
    Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Éxito" : "Fallo");
}

/**
 * Clase de callbacks para eventos de conexión BLE.
 * Permite saber cuándo un cliente BLE se conecta o desconecta.
 */
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) override {
        deviceConnected = true; // Se conectó un cliente BLE
    }
    void onDisconnect(BLEServer* pServer) override {
        deviceConnected = false; // Se desconectó el cliente BLE
    }
};

/**
 * setup()
 * Inicializa todos los periféricos y protocolos:
 * - Serial para depuración
 * - Pin del LED de advertencia
 * - WiFi en modo estación (necesario para ESP-NOW)
 * - ESP-NOW y registro de peer receptor
 * - BLE y servicio UART BLE para comunicación con apps móviles
 */
void setup() {
    Serial.begin(115200); // Inicializa la comunicación serie para depuración

    // Inicializa el pin del LED (puede variar según el hardware)
    pinMode(8, OUTPUT);
    digitalWrite(8, HIGH); // Apaga el LED al inicio (asume LED activo en bajo)

    // Inicializa WiFi en modo estación (requisito para ESP-NOW)
    WiFi.mode(WIFI_STA);
    Serial.println("Modo WiFi: STA");

    // Inicializa ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("Error inicializando ESP-NOW");
        return;
    }

    // Registra el callback de envío de ESP-NOW
    esp_now_register_send_cb(OnDataSent);

    // Configura el peer receptor para ESP-NOW (dirección MAC del receptor)
    esp_now_peer_info_t peerInfo = {};
    memcpy(peerInfo.peer_addr, broadcastAddress, 6);
    peerInfo.channel = 0;  // Canal por defecto
    peerInfo.encrypt = false; // Sin cifrado

    if (esp_now_add_peer(&peerInfo) != ESP_OK) {
        Serial.println("Error agregando peer");
        return;
    }

    // Inicializa BLE y crea el servidor
    BLEDevice::init("ESP32_C3_BLE");
    pServer = BLEDevice::createServer();
    pServer->setCallbacks(new MyServerCallbacks());

    // Crea el servicio BLE UART
    BLEService *pService = pServer->createService(SERVICE_UUID);

    // Característica para notificaciones (TX, del ESP32 al cliente)
    pTxCharacteristic = pService->createCharacteristic(
        CHARACTERISTIC_UUID_TX,
        BLECharacteristic::PROPERTY_NOTIFY
    );
    pTxCharacteristic->addDescriptor(new BLE2902());

    // Característica para escritura (RX, del cliente al ESP32, no usada en este ejemplo)
    BLECharacteristic *pRxCharacteristic = pService->createCharacteristic(
        CHARACTERISTIC_UUID_RX,
        BLECharacteristic::PROPERTY_WRITE
    );

    pService->start();
    pServer->getAdvertising()->start(); // Comienza a anunciar el servicio BLE
    Serial.println("Esperando conexión de cliente BLE...");
}

/**
 * loop()
 * - Lee el sensor flex cada segundo.
 * - Si detecta una flexión incorrecta (valor menor al umbral):
 *      - Enciende el LED de advertencia brevemente.
 *      - Envía mensaje de advertencia por ESP-NOW al receptor.
 *      - Envía mensaje por BLE si hay cliente conectado.
 * - Apaga el LED después de 100 ms.
 */
void loop() {
    unsigned long currentMillis = millis();

    // Verifica si ha pasado el intervalo de muestreo/envío
    if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;

        // --- Lectura y análisis del sensor FLEX ---
        long suma = 0;
        for (int i = 0; i < 50; i++) {
            suma += analogRead(flexPin); // Lee el sensor flex varias veces para promediar
            delay(5); // Pequeña pausa para estabilidad
        }
        int flexValue = suma / 50; // Valor promedio del sensor
        Serial.print("Valor del sensor flex: ");
        Serial.println(flexValue);

        // Si el valor es menor al umbral, se considera una flexión incorrecta
        if (flexValue < umbral) {
           const char estado[] = "CORRIGE FLEXION"; // Mensaje de advertencia

            // Enciende el LED de advertencia
            digitalWrite(8, LOW); // LED ON (asume LED activo en bajo)
            ledOn = true;
            ledOnMillis = currentMillis;

            // Envía el mensaje por ESP-NOW al receptor
            esp_err_t result = esp_now_send(broadcastAddress, (uint8_t *)estado, sizeof(estado));
            if (result == ESP_OK) {
                Serial.print("Enviado estado por ESP-NOW: ");
                Serial.println(estado);
            } else {
                Serial.println("Error enviando los datos por ESP-NOW");
            }

            // Envía el mensaje por BLE si hay cliente conectado
            if (deviceConnected) {
                String mensaje = String(estado) + "\r\n"; // Formato compatible con apps UART BLE
                pTxCharacteristic->setValue(mensaje.c_str());
                pTxCharacteristic->notify();
                Serial.print("Enviado por BLE: ");
                Serial.print(mensaje);
            }
        }
    }

    // Apaga el LED después de 100 ms para indicar advertencia breve
    if (ledOn && (millis() - ledOnMillis >= 100)) {
        digitalWrite(8, HIGH); // LED OFF
        ledOn = false;
    }
}

```