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

El flujo de información entre el hardware físico, la nube y los paneles de usuario se sintetiza en el siguiente esquema UML:

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

    Asignación de Pines Virtuales (Datastreams)Pin VirtualTipo de DatoDirecciónDescripción / Componente AsociadoV0Double / FloatESP32 $\rightarrow$ CloudLectura de Temperatura (°C) - Sensor DHT11V1Double / FloatESP32 $\rightarrow$ CloudLectura de Humedad (%) - Sensor DHT11V2Integer (0/1)ESP32 $\rightarrow$ CloudAlerta de Calor / Radiación (0 = Normal, 1 = Calor) - LM393V3StringESP32 $\rightarrow$ CloudDirección IP asignada en la red localV4StringESP32 $\rightarrow$ CloudEstado de la conexión a Blynk CloudV5Integer (0/1)Cloud $\rightarrow$ ESP32Interruptor de control remoto para el LED RojoV6Integer (0/1)Cloud $\rightarrow$ ESP32Interruptor de control remoto para el BuzzerV7Integer (0/1)Cloud $\rightarrow$ ESP32Botón virtual para Rotar Pantalla OLED

🚀 Instrucciones para Replicar el ProyectoSigue estos pasos en orden para montar y ejecutar el proyecto desde cero en cualquier entorno:Paso 1: Configuración de la Plataforma Cloud (Blynk IoT)Ingresa a blynk.cloud e inicia sesión con tu cuenta.Ve a Developer Zone ➔ Templates ➔ Create New Template:Nombre: AlarmaHardware: ESP32Connection Type: WiFiVe a la pestaña Datastreams y crea los siguientes 8 Pines Virtuales:V0: Double/Float (Temperatura) | Rango: 0 a 100V1: Double/Float (Humedad) | Rango: 0 a 100V2: Integer (Alerta Calor) | Rango: 0 a 1V3: String (Dirección IP)V4: String (Estado Blynk)V5: Integer (LED Rojo) | Rango: 0 a 1V6: Integer (Buzzer) | Rango: 0 a 1V7: Integer (Cambiar Pantalla) | Rango: 0 a 1Ve a la pestaña Web Dashboard (o en la App Móvil) y arrastra los widgets:History Chart ➔ Vinculado a V0 (Temperatura).Gauge / Value Display ➔ Vinculado a V1 (Humedad).LED Widget / Color Block ➔ Vinculado a V2 (Calor).Switch ➔ Vinculado a V5 (LED Rojo).Switch ➔ Vinculado a V6 (Buzzer).Button (Modo PUSH) ➔ Vinculado a V7 (Navegación OLED).Labels / Text ➔ Vinculados a V3 (IP) y V4 (Estado).Ve a Search ➔ Devices ➔ New Device ➔ From Template y selecciona Alarma.En la pestaña Device Info, copia las 3 líneas con el BLYNK_TEMPLATE_ID, BLYNK_TEMPLATE_NAME y BLYNK_AUTH_TOKEN.Paso 2: Conexión del Hardware (Esquema de Pines GPIO)Realiza las conexiones en la protoboard según la siguiente tabla de asignación de pines GPIO del ESP32:ComponentePin del ComponenteGPIO ESP32DHT11DATA / OUTGPIO 23LM393 (Calor)DO (Digital Out)GPIO 22OLED SH1106SDA / SCLGPIO 4 (SDA) / GPIO 5 (SCL)Push ButtonPin FísicoGPIO 17 (INPUT_PULLUP interno)LED RojoÁnodo (+)GPIO 18LED AmarilloÁnodo (+)GPIO 19LED VerdeÁnodo (+)GPIO 21Buzzer ActivoPositivo (+)GPIO 16Paso 3: Configuración del Entorno en Arduino IDEAbre el Arduino IDE e instala el soporte para placas ESP32 desde el Board Manager.Ve a Herramientas ➔ Administrar Bibliotecas e instala las siguientes librerías:WiFiManager (por tzapu)Blynk (por Volodymyr Shymanskyy)DHT sensor library (por Adafruit)Adafruit SH110X y Adafruit GFX LibraryAbre el archivo del proyecto PROYECTOIOT.ino.En las primeras líneas del código, reemplaza las constantes con el Token de tu dispositivo obtenido en el Paso 1:C++#define BLYNK_TEMPLATE_ID   "TMPL2YN-F-M4Y"
#define BLYNK_TEMPLATE_NAME "Alarma"
#define BLYNK_AUTH_TOKEN    "UtYj3HhSHoIrAXDtISwUVzEYqyDhH0ex"
Conecta el ESP32 a la PC mediante USB, selecciona la placa ESP32 Dev Module, escoge el puerto COM correspondiente y presiona Subir (Upload).Paso 4: Puesta en Marcha y Conexión de RedUna vez grabado el programa, el ESP32 encenderá el LED Amarillo e iniciará la red WiFi de configuración llamada Alarma-Aula-Juanes.Desde tu teléfono celular o PC, conéctate a la red WiFi Alarma-Aula-Juanes.Se abrirá automáticamente el portal cautivo de WiFiManager (si no abre, ingresa desde el navegador a la IP 192.168.4.1).Selecciona Configure WiFi, busca la red WiFi de tu aula/hogar, ingresa la clave de red y presiona Save.El ESP32 guardará los datos, se conectará a la red local, encenderá el LED Verde y pasará al estado Online en la consola de Blynk.Paso 5: Pruebas de VerificaciónTelemetría: Comprueba que los valores de temperatura y humedad se muestren en tiempo real en los widgets de la app.Sensor de Calor: Acerca una fuente de calor al sensor LM393 y verifica que el indicador cambie de 0 a 1 al instante.Control Remoto: Presiona los botones virtuales V5 (LED Rojo), V6 (Buzzer) y V7 (Rotar OLED) desde tu celular y confirma la activación inmediata de los componentes físicos en el ESP32.🧪 Pruebas de Integración y Resultados (Fase 3 y 4)Se diseñó una interfaz móvil e hiperconectada orientada a la operación en tiempo real. La transmisión de eventos críticos (detección de calor con LM393) se ejecuta mediante la interrupción por cambio de estado, enviando inmediatamente la señal a Blynk sin depender del ciclo de muestreo.PRUEBA 1: TELEMETRÍA Y PUBLICACIÓNResultado: Exitoso. Las variables de humedad y temperatura se actualizan cada 2.5 segundos en la aplicación. El gráfico histórico (History Chart) almacena los registros sin pérdida de paquetes.PRUEBA 2: CONTROL BIDIRECCIONALResultado: Exitoso. La activación de los switches para V5 (LED) y V6 (Buzzer) commuta los pines físicos del ESP32 en menos de 150 ms. El botón virtual V7 rota la pantalla OLED en sincronía con el pulsador físico.PRUEBA 3: SENSÓRICA INMEDIATAResultado: Exitoso. Al acercar calor al LM393, la transición 0 $\rightarrow$ 1 conmuta instantáneamente en la aplicación y activa el indicador de alerta sin sufrir sofocamiento (Rate Limit).PRUEBA 4: RESILIENCIA DE REDResultado: Exitoso. Al desconectar la alimentación o apagar el router Wi-Fi, la app marca el estado Offline. Al restaurar la energía, WiFiManager y Blynk.config() reconectan automáticamente.

