¡Excelente! Tienes el código final de tu ESP32 completo con la lógica de Polling, la ejecución de comandos (`FIND_PET`), la confirmación (`ACK`) y la simulación de ubicación GPS para una actualización de emergencia.

Aquí tienes un archivo `README.md` exhaustivo para el proyecto **Alpha Tech Smart Collar**, incluyendo el circuito y la configuración de uso.

---

#🐕 Alpha Tech Smart Collar - Código del Collar (ESP32)Este proyecto implementa el *firmware* para el collar inteligente de mascotas Alpha Tech, utilizando un módulo **ESP32 Dev Module** para conectividad Wi-Fi, control de hardware (Buzzer/LED) y comunicación con el Backend a través de **Polling HTTP/S**.

##1. ⚙️ Visión General del ProyectoEl collar funciona en modo **"Siempre Encendido"** (Loop Polling), chequeando periódicamente un *endpoint* de la API para ver si el usuario ha solicitado la función **"Encontrar Mascota"** (`FIND_PET`).

###1.1. 📡 Flujo de Comunicación (Polling)1. **Polling (Cada 5s):** El ESP32 envía un `GET` a la API (`/commands/:petId`).
2. **Recepción:** Si la API responde con `{"command": "FIND_PET"}`.
3. **Ejecución de Emergencia:** El ESP32 actualiza la ubicación (simulada), activa el *buzzer* y el LED por 3 segundos.
4. **Confirmación (ACK):** El ESP32 envía un `POST` a la API (`/commands/ack`) para borrar la orden y detener la repetición.

##2. 🔌 Componentes de Hardware Requeridos| Componente | Descripción | Función en el Proyecto |
| --- | --- | --- |
| **ESP32 Dev Module** | Placa principal con Wi-Fi/Bluetooth. | Lógica de Polling y control de pines. |
| **Buzzer Activo/Pasivo** | Componente de sonido (se recomienda Pasivo con Transistor). | Alarma audible para encontrar a la mascota. |
| **LED** | LED estándar de 5mm (con resistencia de 220 \Omega). | Indicador visual durante el comando (`ledPin`). |
| **LED de Estado** | LED para el estado de la conexión (`statusLedPin`). | Muestra si el collar está conectado a Wi-Fi. |
| **Transistor NPN (ej. BC547)** | **(Opcional, pero recomendado)** Necesario para amplificar la señal si el *buzzer* requiere más corriente de la que el pin GPIO del ESP32 puede suministrar. | Activa el *buzzer* con una fuente externa de 5 \text{V} o el voltaje de la batería. |
| **Batería 18650** | Mínimo una, o dos en **Paralelo** (para larga duración). | Fuente de alimentación para el sistema (conectada al pin \text{VIN} o \text{VBUS} si se usa USB). |

##3. 🖼️ Esquema de Conexión (Circuito)El componente crítico es el **Buzzer**, que debe ser manejado por un transistor para proteger el pin GPIO del ESP32.

###Conexión del Buzzer con Transistor (Configuración Recomendada)| Pin del ESP32 | Conexión del Componente |
| --- | --- |
| **GPIO 2** (`buzzerPin`) | Base del **Transistor NPN** (a través de una Resistencia de 1 \text{K}\Omega). |
| **GPIO 4** (`ledPin`) | **Ánodo** (Pata Larga) del LED (a través de 220 \Omega a \text{GND}). |
| **GPIO 5** (`statusLedPin`) | **Ánodo** (Pata Larga) del LED (a través de 220 \Omega a \text{GND}). |
| **GND** | **Emisor** del Transistor, **Cátodos** de LEDs. |
| **VIN (o \text{VBUS}/5V)** | **Colector** del Transistor (a través del **Buzzer +**). |

##4. 🛠️ Configuración Inicial del CódigoAntes de subir el código, debes instalar la librería y personalizar las constantes en el archivo `.ino`.

###4.1. Instalación de Librería| Librería | Cómo Instalar |
| --- | --- |
| **`ArduinoJson`** | Abrir el Administrador de Librerías en el IDE de Arduino y buscar/instalar `ArduinoJson` (versión 6+). |

###4.2. Variables de ConfiguraciónAjusta estos valores en la sección `// Configuration` del código:

| Constante | Valor Actual (Ejemplo) | Descripción |
| --- | --- | --- |
| `ssid` | `"CRISTO ES MI REFUGIO"` | **Tu red Wi-Fi local.** |
| `password` | `"RestrepoAyala"` | **Tu contraseña de Wi-Fi.** |
| `apiUrl` | `https://executive-cent-reliability-eva.trycloudflare.com` | **API de Comandos (Backend).** |
| `locationApiUrl` | `https://pacific-fighter-missile-stuffed.trycloudflare.com` | **API de Actualización de Ubicación.** |
| `petId` | `"3c5387e8-cb87-4fc7-8e18-0fe44adc9519"` | **ID de Mascota Único (Debe coincidir con la DB del Backend).** |
| `pollingInterval` | `5000` | Frecuencia de chequeo del servidor (5 segundos). |

##5. 💻 Funcionalidad del Código (Funciones Principales)| Función | Propósito | Rutas de API Involucradas |
| --- | --- | --- |
| `connectToWiFi()` | Intenta conectar el ESP32 a la red Wi-Fi por un tiempo límite. | N/A |
| `checkForCommands()` | Realiza la petición `GET` al servidor cada `pollingInterval`. | `GET /commands/:petId` |
| `executeCommand()` | Llama a `sendLocationUpdate()`, activa el *buzzer* y el LED. | N/A |
| `sendLocationUpdate()` | Envía las coordenadas GPS simuladas al servidor de ubicación (función de emergencia). | `POST /location/collar/:collarId/position` |
| `sendAcknowledgment()` | Envía el `POST` para notificar que el comando fue ejecutado. | `POST /commands/ack` |

---

###**⚠️ Nota de Despliegue (HTTPS)**Dado que el `apiUrl` y `locationApiUrl` son HTTPS (`https://`), para que la conexión funcione sin el error **HTTP: -1** o **500**, es altamente recomendable que tu código **incluya la sincronización de tiempo (NTP)** y use **`WiFiClientSecure`** con la opción `client.setInsecure();` (para evitar problemas de certificado con túneles como Cloudflare), aunque el código actual utiliza `HTTPClient` con la URL HTTPS completa. Si tienes problemas de conexión, añade la librería `WiFiClientSecure.h` y la lógica NTP.