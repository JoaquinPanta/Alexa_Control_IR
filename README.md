# Sistema de Control por Voz con ESP32 y Alexa

## Fases de Implementación

1.-**Captura de señales IR**: Utilizar el script de escaneo para capturar los códigos hexadecimales de los mandos originales y crear la tabla de funciones personalizada.

2.-**Configuración de AWS**: Crear un Thing en AWS IoT Core y descargar los certificados (CA, CRT, Private Key).

3.-**Despliegue de Backend**: Crear una función en AWS Lambda con el código del proyecto y configurarla como disparador de la Skill de Alexa.

4.-**Programación del ESP32**: Actualizar en el archivo main.cpp los datos de WiFi, el endpoint de AWS, los certificados y la tabla de códigos IR obtenidos en la fase 1.

5.-**Configuración de Alexa**: Crear los Intents en la consola de desarrollador de Alexa añadiendo el slot AMAZON.NUMBER para manejar las repeticiones de comandos.

## Credenciales

1.-**AWS IoT Endpoint**: Configurado en main.cpp

2.-**Certificados**: Requiere certificados X.509 (CA, CRT y Private Key) vinculados al dispositivo en AWS IoT.

## Características

1.-Control remoto de dispositivos IR (TV y audio) mediante comandos de voz.

2.-Ejecución de comandos n veces (ej. "subir volumen 3 veces") usando parámetros dinámicos.

3.-Soporte multiformato IR (Samsung, NEC, etc.).

4.-Comunicación segura mediante cifrado TLS/SSL.

5.-Lógica de auto-reconexión a red WiFi y broker MQTT.

## Estructura del Proyecto

1.-**ESP32**: Manejo de protocolos IR, comunicación MQTT y procesamiento JSON.

2.-**Lambda**: Procesador de peticiones de Alexa y publicador de mensajes hacia AWS IoT.

3.-**AWS IoT**: Intermediario seguro para la gestión de mensajes en tiempo real.
