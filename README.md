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

## 📋 Asignación de Pines Virtuales (Datastreams)

| Pin Virtual | Tipo de Dato | Dirección | Descripción / Componente Asociado |
| :--- | :--- | :--- | :--- |
| **V0** | Double / Float | ESP32 $\rightarrow$ Cloud | Lectura de Temperatura (°C) - Sensor DHT11 |
| **V1** | Double / Float | ESP32 $\rightarrow$ Cloud | Lect


