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
graph LR
    subgraph Hardware ["ESP32 (Hardware Local)"]
        DHT11["Sensor DHT11<br/>(Temp / Humedad)"]
        LM393["Sensor LM393<br/>(Detección de Calor)"]
        ACT["Actuadores Físicos<br/>(LED Rojo, Buzzer, OLED)"]
    end

    subgraph Cloud ["Blynk Cloud Broker"]
        DS["Datastreams / Virtual Pins<br/>(V0 - V7)"]
    end

    subgraph Dashboard ["Interfaz de Usuario"]
        App["App Móvil / Web Dashboard<br/>(History Chart, Gauges, Switches)"]
    end

    %% Flujos de Telemetría (Sensores -> Cloud -> Dashboard)
    DHT11 -->|Publicación cada 2.5s| DS
    LM393 -->|Evento Instantáneo| DS
    DS -->|Actualización en vivo| App

    %% Flujos de Control (Dashboard -> Cloud -> ESP32)
    App -->|Comandos Remotos| DS
    DS -->|Callbacks BLYNK_WRITE| ACT
