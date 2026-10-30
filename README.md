# Sistema IoT de Monitoreo Ambiental y Control Bidireccional

**Asignatura:** IoT (Internet de las Cosas)  
**Institución:** Unidades Tecnológicas de Santander (UTS)  
**Estudiante:** Juan Esteban Cárdenas López  
**Profesor:** Eliecer Montero  

---

## 📌 Descripción del Proyecto

Este proyecto implementa un sistema IoT bidireccional que recopila datos ambientales en tiempo real (Temperatura, Humedad y Alerta de Calor) mediante un microcontrolador **ESP32** y los transmite a la plataforma en la nube **Blynk IoT**. 

Adicionalmente, el sistema permite el **control remoto de actuadores** (LED Rojo, Buzzer sonoro y navegación en pantalla OLED SH1106) desde una aplicación móvil o un dashboard web, garantizando alta disponibilidad, baja latencia y reconexión automática mediante un portal cautivo WiFi (*WiFiManager*).

---

## 🏗️ Diagrama de Arquitectura

El flujo de información entre el hardware físico, la nube y los paneles de usuario se sintetiza en el siguiente esquema:

```mermaid
graph TD
    subgraph ESP32 ["ESP32 (Hardware Local)"]
        DHT["Sensor DHT11"]
        LM["Sensor LM393"]
        ACT["Actuadores Físicos<br/>(LED Rojo, Buzzer, Pantalla OLED)"]
    end

    subgraph BlynkCloud ["Blynk Cloud Broker (Servidor)"]
        DS["Datastreams / Pines Virtuales<br/>(V0, V1, V2, V3, V4, V5, V6, V7)"]
    end

    subgraph UserInterface ["Interfaz de Usuario (Cliente)"]
        UI["App Móvil / Web Dashboard<br/>(Charts, Gauges, Switches)"]
    end

    %% Enlaces de Telemetría
    DHT -->|V0: Temperatura / V1: Humedad| DS
    LM -->|V2: Alerta Calor| DS

    %% Enlaces de Diagnóstico
    ESP32 -->|V3: IP / V4: Estado| DS

    %% Conexión Bidireccional
    DS <-->|Sincronización Cloud| UI

    %% Control Remoto
    UI -->|Comandos V5, V6, V7| DS
    DS -->|Callbacks BLYNK_WRITE| ACT


## 🚀 Instrucciones para Replicar el Proyecto

Sigue estos pasos en orden para montar y ejecutar el proyecto desde cero en cualquier entorno:

### Paso 1: Configuración de la Plataforma Cloud (Blynk IoT)

1. Ingresa a [blynk.cloud](https://blynk.cloud/) e inicia sesión con tu cuenta.
2. Ve a **Developer Zone** ➔ **Templates** ➔ **Create New Template**:
   * **Nombre:** `Alarma`
   * **Hardware:** `ESP32`
   * **Connection Type:** `WiFi`
3. Ve a la pestaña **Datastreams** y crea los siguientes 8 Pines Virtuales:
   * `V0`: Double/Float (Temperatura) | Rango: 0 a 100
   * `V1`: Double/Float (Humedad) | Rango: 0 a 100
   * `V2`: Integer (Alerta Calor) | Rango: 0 a 1
   * `V3`: String (Dirección IP)
   * `V4`: String (Estado Blynk)
   * `V5`: Integer (LED Rojo) | Rango: 0 a 1
   * `V6`: Integer (Buzzer) | Rango: 0 a 1
   * `V7`: Integer (Cambiar Pantalla) | Rango: 0 a 1
4. Ve a la pestaña **Web Dashboard** (o en la App Móvil) y arrastra los widgets:
   * **History Chart** ➔ Vinculado a `V0` (Temperatura).
   * **Gauge / Value Display** ➔ Vinculado a `V1` (Humedad).
   * **LED Widget / Color Block** ➔ Vinculado a `V2` (Calor).
   * **Switch** ➔ Vinculado a `V5` (LED Rojo).
   * **Switch** ➔ Vinculado a `V6` (Buzzer).
   * **Button (Modo PUSH)** ➔ Vinculado a `V7` (Navegación OLED).
   * **Labels / Text** ➔ Vinculados a `V3` (IP) y `V4` (Estado).
5. Ve a **Search** ➔ **Devices** ➔ **New Device** ➔ **From Template** y selecciona `Alarma`.
6. En la pestaña **Device Info**, copia las 3 líneas con el `BLYNK_TEMPLATE_ID`, `BLYNK_TEMPLATE_NAME` y `BLYNK_AUTH_TOKEN`.

---

### Paso 2: Conexión del Hardware (Esquema de Pines GPIO)

Realiza las conexiones en la protoboard según la siguiente tabla de asignación de pines GPIO del ESP32:

| Componente | Pin del Componente | GPIO ESP32 |
| :--- | :--- | :--- |
| **DHT11** | DATA / OUT | `GPIO 23` |
| **LM393 (Calor)** | DO (Digital Out) | `GPIO 22` |
| **OLED SH1106** | SDA / SCL | `GPIO 4` (SDA) / `GPIO 5` (SCL) |
| **Push Button** | Pin Físico | `GPIO 17` (INPUT_PULLUP interno) |
| **LED Rojo** | Ánodo (+) | `GPIO 18` |
| **LED Amarillo** | Ánodo (+) | `GPIO 19` |
| **LED Verde** | Ánodo (+) | `GPIO 21` |
| **Buzzer Activo** | Positivo (+) | `GPIO 16` |

---

### Paso 3: Configuración del Entorno en Arduino IDE

1. Abre el **Arduino IDE** e instala el soporte para placas ESP32 desde el *Board Manager*.
2. Ve a **Herramientas** ➔ **Administrar Bibliotecas** e instala las siguientes librerías:
   * `WiFiManager` (por tzapu)
   * `Blynk` (por Volodymyr Shymanskyy)
   * `DHT sensor library` (por Adafruit)
   * `Adafruit SH110X` y `Adafruit GFX Library`
3. Abre el archivo del proyecto `PROYECTOIOT.ino`.
4. En las primeras líneas del código, reemplaza las constantes con el Token de tu dispositivo obtenido en el Paso 1:
   ```cpp
   #define BLYNK_TEMPLATE_ID   "TMPL2YN-F-M4Y"
   #define BLYNK_TEMPLATE_NAME "Alarma"
   #define BLYNK_AUTH_TOKEN    "UtYj3HhSHoIrAXDtISwUVzEYqyDhH0ex"
