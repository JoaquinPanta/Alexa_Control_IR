#include <Arduino.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <IRremote.h>

/* ---------- Configuración WiFi ---------- */
const char* WIFI_SSID = "S24 de Joaquin";
const char* WIFI_PASS = "12345678";

/* ---------- Configuración AWS IoT ---------- */
const char* AWS_IOT_ENDPOINT = "a1yeki2zwyj8jj-ats.iot.us-east-1.amazonaws.com";
const char* CLIENT_ID        = "ESP32_IR_Control";

// Tópicos
const char* TOPIC_POWER  = "casa/tv/power";
const char* TOPIC_VOLUME = "casa/tv/volume";
const char* TOPIC_AUDIO  = "casa/audio/control";

/* ---------- Certificados ---------- */
static const char AWS_CERT_CA[] PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
MIIDQTCCAimgAwIBAgITBmyfz5m/jAo54vB4ikPmljZbyjANBgkqhkiG9w0BAQsF
ADA5MQswCQYDVQQGEwJVUzEPMA0GA1UEChMGQW1hem9uMRkwFwYDVQQDExBBbWF6
b24gUm9vdCBDQSAxMB4XDTE1MDUyNjAwMDAwMFoXDTM4MDExNzAwMDAwMFowOTEL
MAkGA1UEBhMCVVMxDzANBgNVBAoTBkFtYXpvbjEZMBcGA1UEAxMQQW1hem9uIFJv
b3QgQ0EgMTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALJ4gHHKeNXj
ca9HgFB0fW7Y14h29Jlo91ghYPl0hAEvrAIthtOgQ3pOsqTQNroBvo3bSMgHFzZM
9O6II8c+6zf1tRn4SWiw3te5djgdYZ6k/oI2peVKVuRF4fn9tBb6dNqcmzU5L/qw
IFAGbHrQgLKm+a/sRxmPUDgH3KKHOVj4utWp+UhnMJbulHheb4mjUcAwhmahRWa6
VOujw5H5SNz/0egwLX0tdHA114gk957EWW67c4cX8jJGKLhD+rcdqsq08p8kDi1L
93FcXmn/6pUCyziKrlA4b9v7LWIbxcceVOF34GfID5yHI9Y/QCB/IIDEgEw+OyQm
jgSubJrIqg0CAwEAAaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAOBgNVHQ8BAf8EBAMC
AYYwHQYDVR0OBBYEFIQYzIU07LwMlJQuCFmcx7IQTgoIMA0GCSqGSIb3DQEBCwUA
A4IBAQCY8jdaQZChGsV2USggNiMOruYou6r4lK5IpDB/G/wkjUu0yKGX9rbxenDI
U5PMCCjjmCXPI6T53iHTfIUJrU6adTrCC2qJeHZERxhlbI1Bjjt/msv0tadQ1wUs
N+gDS63pYaACbvXy8MWy7Vu33PqUXHeeE6V/Uq2V8viTO96LXFvKWlJbYK8U90vv
o/ufQJVtMVT8QtPHRh8jrdkPSHCa2XV4cdFyQzR1bldZwgJcJmApzyMZFo6IQ6XU
5MsI+yMRQ+hDKXJioaldXgjUkK642M4UwtBV8ob2xJNDd2ZhwLnoQdeXeGADbkpy
rqXRfboQnoZsG4q5WTP468SQvvG5
-----END CERTIFICATE-----
)EOF";

static const char AWS_CERT_CRT[] PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
MIIDWTCCAkGgAwIBAgIUduJPvyMpIN8wcpdKzlYS1xnyosgwDQYJKoZIhvcNAQEL
BQAwTTFLMEkGA1UECwxCQW1hem9uIFdlYiBTZXJ2aWNlcyBPPUFtYXpvbi5jb20g
SW5jLiBMPVNlYXR0bGUgU1Q9V2FzaGluZ3RvbiBDPVVTMB4XDTI2MDUxMzE2NTQz
NFoXDTQ5MTIzMTIzNTk1OVowHjEcMBoGA1UEAwwTQVdTIElvVCBDZXJ0aWZpY2F0
ZTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAL9t1A1cZjdbG7cPu7P+
YdTpqDG2ZON8B/HYdULh9V80D6AUc7AH9owxpzUWMz0Dfms/S6ttDUUDfYMfhNDI
srGaIs88ekeaSk2rTv8uMy4fS/V6zsep6oZ4VEwz6iuQOHpbHTsRA5309v7oVLII
TkxrCZkwGM5iLHaXBu7gLU71iY6EKIVr/DWaJMDIVrN0naCBzdpHW5byjczAqdvZ
FaxyJbxTW2CTYR/KgPvFwgbbd23PmYLW/c6AJeFbvssHSygGyOfYBH0HZcrs2PIz
NpesyNTu7CfdjakuIiFk8qBi0mAmGWj/hE3/vG35aGlgtiuiS7eB+f1RjCH0DvG5
KV0CAwEAAaNgMF4wHwYDVR0jBBgwFoAUAG3ELzjsr5hnOmDD2zXw0ngqQhwwHQYD
VR0OBBYEFO/Wcp14RxKdlijmAGSiP6q29hSXMAwGA1UdEwEB/wQCMAAwDgYDVR0P
AQH/BAQDAgeAMA0GCSqGSIb3DQEBCwUAA4IBAQCCSN+pR6FJ4JROLhLirPQXm2zU
1bwL8DL+mpFDg49XaVdrxDra+ElgxhUbbFKk8J+UgR8woSdvTzzUrhaKkq+XkhtN
de4hTaNjqFvWDB8a7Lnt5L9G+FBvnO0O6P3AqzMrCTTfMkctEuGVnr+nFmY0h/lg
UWHZC3JOMnBDWc0hhiknQzTUElqwhq5rzb6Wu/6HAHtVZippjywR3KXsZr8wFbRB
Bvcir7Ty6OpxXTzhzPV2Xyj6S8MTafg+uL0L3CeSiCrQqLUKkmAX/tHe3RFnoWJ0
KiWb2na/ytoP/7m1nv+S/SME6TxIm39beYwmocrXN7fr5SkOIS6X+LJi3jJh
-----END CERTIFICATE-----
)EOF";

static const char AWS_CERT_PRIVATE[] PROGMEM = R"EOF(
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAv23UDVxmN1sbtw+7s/5h1OmoMbZk43wH8dh1QuH1XzQPoBRz
sAf2jDGnNRYzPQN+az9Lq20NRQN9gx+E0MiysZoizzx6R5pKTatO/y4zLh9L9XrO
x6nqhnhUTDPqK5A4elsdOxEDnfT2/uhUsghOTGsJmTAYzmIsdpcG7uAtTvWJjoQo
hWv8NZokwMhWs3SdoIHN2kdblvKNzMCp29kVrHIlvFNbYJNhH8qA+8XCBtt3bc+Z
gtb9zoAl4Vu+ywdLKAbI59gEfQdlyuzY8jM2l6zI1O7sJ92NqS4iIWTyoGLSYCYZ
aP+ETf+8bfloaWC2K6JLt4H5/VGMIfQO8bkpXQIDAQABAoIBAQCB3tMUVXGLcKY2
5dekX5qjxfmz8Fg19Tm2m/HB9GZxtnI7WmS6MkzUBAO6gls0s6AaxpBhttRf0+Rd
Bw8mTmLdFwMysYcL+OMcGnK3sPJMAjOYUT7kOqS/ofce08G02F1YtWP5ZKudiEnh
2mFLf1MPiTaz1OufaP9dXUz41uImRES3agCEcXjfmQKXVvI7YbWGa2p4kkgrupyA
vWDWj7i/eBv1REl3AP+zfyEai2GAw5WwJwIpZbqHNxQBeu7VJE8QfAalFVteASuG
JYGipkNcbcvRkVnJ7p9J1kP28Er4ZIcy86ajsvkBkUgr/qDXqVnmHrydJyBNVQH7
/zhXS31BAoGBAPSK2BGK8srx9aaHR5lAw/ZGPNGswdJK/FWC7ZuSjFZ1xe+p/PMH
npDBR1K5JqvEgYeKmWn+1C0kAqgB5ubf5UnVuwLwt55mxw7FuP8yDjwMAmJyW4wW
nSyqnhSg4/rBhKmZCZAneucITjdUz5xv5sxtYcREG5rHSaKgPRqKsc8lAoGBAMhl
6vzQD+M3xVKRSa/naTfZBrFfnvsvOer6NRD1UHtegt/BYes5hDY4cvanbMaSSQqS
Ax/RWXI+z71R3ligUmdGUz2GHSkMFIDZLz7jzvJapjIENhqTZZEXfwi3U84fhLib
19B1r2y7Q7i56hOz3TyB/PDRWx/cpuX9G3ibv1fZAoGAHtE8h4FuNXsmjPTxZfPO
fkO9uGLADkNa51/Xl8pxGxg+eISAlxP3vKBO37Lrl6vs8urZiI0PqhympQuQULoY
iCSCK82wJisnnMn4FVj2iESvLD7jIJVc+1sJDupzsI8gq8rYdK01BNZ8HOb4Puuv
N0X1kylOHUROGEt/R9AAtm0CgYA5JmenhaaiRAtwCqMbGUG2fZwdBYoPVF6OFqD/
nxY+uKtWKI5GSPbdh6exk/m/f4R5ET722ftKS4NvGGPynnYhobqndxHl3N4UCNwi
cVFZaGSxKuZLGfIDONIvqCisRezRwRABES7gMV+U1OBekhLr6J48B1iXc5PdTxuA
f9pHsQKBgQCLbPIv6o9l4mvJc3vt7YTd1a+0b2xPQRHXZ8+APpc4nu3trS1IdPcw
vGL7bJYecCBteXhefPQMrluklk48Nm89Jla2cOIdgbVPipeL9iyKKrLpSrMyQbcn
6DT0P4HuS7XrVN3vos8++TA0xxzToIi+lLyrjS+Va8qC3JKC9nmRyQ==
-----END RSA PRIVATE KEY-----
)EOF";

/* ---------- Hardware ---------- */
#define IR_SEND_PIN 4

/* ---------- Objetos ---------- */
WiFiClientSecure net;
PubSubClient client(net);

/* ================================================
   Lógica de envío IR con repetición dinámia (for)
================================================ */
void triggerIRCommand(String command, int times) {
    Serial.print("Ejecutando comando IR: "); Serial.print(command);
    Serial.print(" - Repeticiones: "); Serial.println(times);

    for (int i = 0; i < times; i++) {
        // --- COMANDOS SAMSUNG (TELEVISOR) ---
        if (command == "tv_on" || command == "tv_off") {
            IrSender.sendSamsung(0x707, 0xE6, 0);
        } 
        else if (command == "volume_up") {
            IrSender.sendSamsung(0x707, 0x07, 0);
        } 
        else if (command == "volume_down") {
            IrSender.sendSamsung(0x707, 0x0B, 0);
        }
        
        // --- COMANDOS NEC (REPRODUCTOR BLUETOOTH) ---
        else if (command == "audio_next") {
            IrSender.sendNEC(0x00, 0x40, 0);
        }
        else if (command == "audio_prev") {
            IrSender.sendNEC(0x00, 0x44, 0);
        }
        else if (command == "audio_pause" || command == "audio_play") {
            IrSender.sendNEC(0x00, 0x43, 0);
        }
        else if (command == "audio_vol_up") {
            IrSender.sendNEC(0x00, 0x15, 0);
        }
        else if (command == "audio_vol_down") {
            IrSender.sendNEC(0x00, 0x07, 0);
        }
        else if (command == "audio_mute") {
            IrSender.sendNEC(0x00, 0x47, 0);
        }

        // Si hay más de una repetición, esperamos un momento entre envíos
        if (times > 1 && i < (times - 1)) {
            delay(150); 
        }
    }
}

void messageHandler(char* topic, byte* payload, unsigned int length) {
    Serial.print("Mensaje recibido en ["); Serial.print(topic); Serial.println("]");
    
    JsonDocument doc;
    deserializeJson(doc, payload, length);
    
    if (doc.containsKey("command")) {
        String command = doc["command"];
        
        // Leemos cuántas veces repetir. Si no viene en el JSON, por defecto es 1
        int times = doc["times"] | 1; 
        
        triggerIRCommand(command, times);
    }
}

void connectAWS() {
    WiFi.mode(WIFI_STA);
    WiFi.begin(WIFI_SSID, WIFI_PASS);

    Serial.println("Conectando a Wi-Fi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    net.setCACert(AWS_CERT_CA);
    net.setCertificate(AWS_CERT_CRT);
    net.setPrivateKey(AWS_CERT_PRIVATE);

    client.setServer(AWS_IOT_ENDPOINT, 8883);
    client.setCallback(messageHandler);

    Serial.println("\nConectando a AWS IoT Core...");
    while (!client.connect(CLIENT_ID)) {
        Serial.print(".");
        delay(1000);
    }

    if (!client.connected()) {
        Serial.println("¡Error de conexión con AWS!");
        return;
    }

    client.subscribe(TOPIC_POWER);
    client.subscribe(TOPIC_VOLUME);
    client.subscribe(TOPIC_AUDIO);

    Serial.println("¡Conectado y suscrito a todos los canales!");
}

void setup() {
    Serial.begin(115200);
    IrSender.begin(IR_SEND_PIN, true, 0);
    connectAWS();
}

void loop() {
    if (!client.connected()) {
        connectAWS();
    }
    client.loop();
}