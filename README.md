# Proyecto Final de Sistemas Digitales
## Carro Seguidor de Líneas de Colores con Comunicación Bluetooth y Dashboard Web

---

## Tabla de Contenidos

1. [Descripción General](#descripción-general)
2. [Objetivos](#objetivos)
3. [Materiales y Componentes](#materiales-y-componentes)
4. [Arquitectura del Sistema](#arquitectura-del-sistema)
5. [Diseño del Hardware](#diseño-del-hardware)
6. [Diseño del Software](#diseño-del-software)
7. [Comunicación Bluetooth](#comunicación-bluetooth)
8. [Dashboard Web con Streamlit](#dashboard-web-con-streamlit)
9. [Chatbot Conversacional](#chatbot-conversacional)
10. [Control por Comandos de Voz](#control-por-comandos-de-voz)
11. [Evidencias](#evidencias)
12. [Instrucciones de Uso](#instrucciones-de-uso)
13. [Resultados y Pruebas](#resultados-y-pruebas)
14. [Conclusiones](#conclusiones)
15. [Referencias](#referencias)

---

## Descripción General

Este proyecto consiste en el diseño, construcción y programación de un carro robótico seguidor de líneas de colores, desarrollado como proyecto final de la asignatura de Sistemas Digitales. El sistema integra un microcontrolador Arduino Uno, un sensor de color TCS34725, un módulo Bluetooth HC-06 y un driver de motores L298N, conformando una plataforma robótica capaz de detectar y diferenciar al menos cinco colores de línea: negro, blanco, rojo, verde y azul.

El carro transmite en tiempo real el color de la línea detectada hacia una aplicación web desarrollada con Streamlit en Python, la cual presenta un dashboard interactivo con visualización del color activo, un chatbot conversacional alimentado por un modelo de lenguaje local (Gemma 3 mediante Ollama) que explica el proyecto, y un módulo de control por comandos de voz que permite dirigir el movimiento del carro de forma inalámbrica.

La comunicación entre el Arduino y la PC se establece de forma bidireccional mediante Bluetooth, permitiendo tanto el envío del color detectado hacia la aplicación, como la recepción de comandos de movimiento desde la interfaz web.

---

## Objetivos

### Objetivo General

Diseñar e implementar un sistema embebido basado en Arduino que integre un carro seguidor de líneas de colores, capaz de transmitir el color detectado a una aplicación web desarrollada con Streamlit, con chatbot conversacional y control por voz.

### Objetivos Específicos

- Construir un carro robótico con sensor de color TCS34725 capaz de diferenciar al menos cinco colores de línea: negro, blanco, rojo, verde y azul.
- Desarrollar una interfaz web en Streamlit que muestre en tiempo real el color detectado por el sensor.
- Incorporar un chatbot conversacional basado en un modelo de lenguaje local (Gemma 3 / Ollama) que explique la construcción, componentes y tecnologías usadas en el proyecto.
- Indicar en el dashboard de Streamlit la línea por la que está pasando el carro en cada momento.
- Establecer comunicación bidireccional entre Arduino y la PC mediante el módulo Bluetooth HC-06, para enviar el color detectado y recibir comandos de movimiento.
- Implementar control por comandos de voz desde la interfaz Streamlit con los comandos: Arriba, Abajo, Derecha e Izquierda.
- Documentar todo el proceso y publicarlo en GitHub como evidencia del proyecto.

---

## Materiales y Componentes

| Componente | Especificación | Función |
|---|---|---|
| Arduino Uno | ATmega328P | Control principal del sistema |
| Sensor de color | TCS34725 (I2C) | Detección del color de la línea |
| Módulo Bluetooth | HC-06 | Comunicación inalámbrica bidireccional |
| Chasis de carro | 2WD | Base robótica del sistema |
| Motores DC | 3-6V con ruedas | Tracción y movimiento del carro |
| Driver de motores | L298N | Control de dirección y velocidad |
| Batería | Pack de pilas AA | Alimentación del sistema |
| Cables y protoboard | — | Conexiones entre componentes |
| PC con Python 3.11 | Windows | Ejecución del dashboard Streamlit |
| Ollama | Gemma 3 1B | Modelo de lenguaje local para el chatbot |

---

## Arquitectura del Sistema

El sistema se compone de tres capas principales que interactúan entre sí:

```
+----------------------------------------------------------+
|                     CAPA DE HARDWARE                     |
|                                                          |
|   [TCS34725] --I2C--> [Arduino Uno] <------ [L298N]     |
|   (Sensor color)      (Control)             (Motores DC) |
|                            |                             |
|                         [HC-06]                          |
|                        (Bluetooth)                       |
+----------------------------+-----------------------------+
                             | Bluetooth (COM4)
                             v
+----------------------------------------------------------+
|                  CAPA DE COMUNICACION                    |
|                                                          |
|          bridge.py -- Puerto Serial COM4                 |
|     (Recibe color del Arduino / Envia comandos)          |
+----------------------------+-----------------------------+
                             |
                             v
+----------------------------------------------------------+
|                  CAPA DE APLICACION                      |
|                                                          |
|             app.py -- Streamlit Dashboard                |
|   +--------------+---------------+------------------+   |
|   |  Dashboard   |    Chatbot    |   Control Voz    |   |
|   | (Color Live) |   (Ollama)   |  (SpeechRec.)    |   |
|   +--------------+---------------+------------------+   |
+----------------------------------------------------------+
```

---

## Diseño del Hardware

### Conexión del Sensor TCS34725 al Arduino Uno

El sensor TCS34725 utiliza el protocolo de comunicación I2C, lo cual simplifica el cableado a dos líneas de datos además de la alimentación.

| Pin TCS34725 | Pin Arduino Uno | Descripción |
|---|---|---|
| VIN | 5V | Alimentación |
| GND | GND | Tierra común |
| SDA | SDA (A4) | Línea de datos I2C |
| SCL | SCL (A5) | Línea de reloj I2C |

### Conexión del Módulo Bluetooth HC-06 al Arduino Uno

| Pin HC-06 | Pin Arduino Uno | Descripción |
|---|---|---|
| VCC | 5V | Alimentación |
| GND | GND | Tierra común |
| TXD | Pin 11 (RX Software) | Transmisión de datos |
| RXD | Pin 10 (TX Software) | Recepción de datos |

**Nota:** Se utilizó la librería SoftwareSerial en los pines 10 y 11 para liberar el puerto Serial hardware (pines 0 y 1), reservado para la comunicación con el Monitor Serial durante el desarrollo y depuración.

### Conexión del Driver L298N al Arduino Uno

| Pin L298N | Pin Arduino Uno | Descripción |
|---|---|---|
| IN1 | Pin 5 | Dirección motor izquierdo |
| IN2 | Pin 4 | Dirección motor izquierdo |
| IN3 | Pin 3 | Dirección motor derecho |
| IN4 | Pin 2 | Dirección motor derecho |
| ENA | Pin 6 (PWM) | Velocidad motor izquierdo |
| ENB | Pin 9 (PWM) | Velocidad motor derecho |
| VCC | Batería AA | Alimentación de los motores |
| GND | GND común | Tierra común |

### Fotografías del Hardware

---

## Diseño del Software

### Estructura del Proyecto

```
C:\proyecto\
|
|-- venv\                    <- Entorno virtual Python
|
|-- carro_robot.ino          <- Codigo Arduino principal
|
|-- app.py                   <- Aplicacion Streamlit
|                               (Dashboard + Chatbot + Voz)
|
+-- bridge.py                <- Puente Serial <-> Streamlit
```


### Codigo Arduino (carro_robot.ino)

El sketch de Arduino implementa las siguientes funcionalidades:

**Deteccion de color:** Lectura del sensor TCS34725 mediante I2C usando la libreria Adafruit_TCS34725. Los valores RGB normalizados se comparan contra umbrales predefinidos para clasificar el color detectado como negro, blanco, rojo, verde o azul.

**Comunicacion Bluetooth:** Uso de SoftwareSerial para enviar el color detectado cada 500 ms hacia la PC, y recibir comandos de movimiento de forma continua sin bloquear el ciclo principal del programa.

**Control de motores:** Implementacion de funciones de movimiento (avanzar, retroceder, girar derecha, girar izquierda, detener) mediante senales PWM en los pines ENA y ENB del L298N.

**Modo de respaldo:** Si el sensor no es detectado al iniciar, el sistema continua operando y permite el control por Bluetooth igualmente, garantizando la funcionalidad del carro independientemente del estado del sensor.


### Librerias de Arduino Utilizadas

| Libreria | Funcion |
|---|---|
| Wire.h | Comunicacion I2C (incluida en el IDE) |
| Adafruit_TCS34725 | Control del sensor de color |
| SoftwareSerial.h | Comunicacion serial por software (incluida en el IDE) |

### Entorno Python

El proyecto utiliza Python 3.11.9 con un entorno virtual ubicado en C:\proyecto\venv. Las dependencias instaladas son las siguientes:

| Libreria | Version | Funcion |
|---|---|---|
| streamlit | 1.57.0 | Framework de la interfaz web |
| pyserial | 3.5 | Comunicacion con puerto serial Bluetooth |
| SpeechRecognition | 3.16.1 | Reconocimiento de voz |
| PyAudio | 0.2.14 | Captura de audio del microfono |
| ollama | Ultima disponible | Integracion con modelos de IA local |
| requests | 2.34.2 | Comunicacion HTTP |

---

## Comunicacion Bluetooth

### Configuracion del HC-06

El modulo HC-06 fue configurado mediante comandos AT utilizando el Arduino como puente serial. Los parametros establecidos fueron los siguientes:

| Parametro | Valor configurado |
|---|---|
| Nombre del dispositivo | CarroRobot |
| PIN de emparejamiento | 1234 |
| Velocidad de comunicacion | 9600 baudios |

### Protocolo de Comunicacion

La comunicacion es bidireccional y sigue el siguiente protocolo:

**Arduino hacia PC — envio de color cada 500 ms:**

```
NEGRO
BLANCO
ROJO
VERDE
AZUL
DESCONOCIDO
SENSOR_NO_DISPONIBLE
```

**PC hacia Arduino — comandos de movimiento:**

```
F  ->  Avanzar   (Forward)
B  ->  Retroceder (Backward)
R  ->  Girar derecha (Right)
L  ->  Girar izquierda (Left)
S  ->  Detener (Stop)
```

### Emparejamiento en Windows

El modulo HC-06 fue emparejado exitosamente con Windows mediante Bluetooth. El sistema operativo asigno automaticamente los siguientes puertos:

| Puerto | Tipo | Uso |
|---|---|---|
| COM3 | USB Arduino | Programacion del microcontrolador |
| COM4 | Bluetooth Salida | Comunicacion principal con HC-06 |
| COM5 | Bluetooth Entrada | Asignado automaticamente por Windows |


---

## Dashboard Web con Streamlit

La aplicacion web desarrollada con Streamlit presenta tres modulos principales en una interfaz unificada:

### Modulo 1 — Dashboard en Tiempo Real

Muestra el color actualmente detectado por el sensor con su representacion visual en el color correspondiente. Indica en todo momento la linea por la que esta pasando el carro y se actualiza automaticamente cada 500 ms.

### Modulo 2 — Chatbot Conversacional

Basado en el modelo Gemma 3 1B corriendo localmente mediante Ollama. Responde preguntas sobre el proyecto, sus componentes y las tecnologias empleadas. No requiere conexion a internet ni clave de API. El sistema es completamente gratuito y opera de forma privada en la maquina local.

### Modulo 3 — Control por Comandos de Voz

Reconocimiento de voz mediante la libreria SpeechRecognition. Soporta los comandos: Arriba, Abajo, Derecha, Izquierda y Detener. Los comandos de voz se traducen a senales Bluetooth que son enviadas al Arduino.

---

## Chatbot Conversacional

### Tecnologia Utilizada

El chatbot fue implementado usando Ollama, una plataforma de codigo abierto bajo licencia MIT que permite ejecutar modelos de lenguaje de gran escala directamente en la computadora local, sin necesidad de conexion a internet ni servicios en la nube.

| Componente | Detalle |
|---|---|
| Plataforma | Ollama (codigo abierto, licencia MIT) |
| Modelo | Gemma 3 1B (Google) |
| Tamano del modelo | Aproximadamente 800 MB |
| RAM requerida | 8 GB minimo |
| Costo | Completamente gratuito |
| Conexion a internet | No requerida |

### Capacidades del Chatbot

El chatbot esta instruido para responder consultas sobre los siguientes temas:

- Descripcion general del proyecto y sus objetivos
- Componentes utilizados y sus funciones especificas
- Proceso de construccion del carro robotico
- Explicacion del codigo Arduino
- Funcionamiento del sensor TCS34725
- Protocolo de comunicacion Bluetooth
- Descripcion de la interfaz Streamlit y sus modulos

---

## Control por Comandos de Voz

El sistema de control por voz permite dirigir el carro mediante comandos hablados en espanol. El flujo de funcionamiento es el siguiente:

```
[Microfono de la PC]
         |
         v
[SpeechRecognition -- Captura y transcripcion]
         |
         v
[Traduccion a comando de movimiento]
   "Arriba"     ->  'F'
   "Abajo"      ->  'B'
   "Derecha"    ->  'R'
   "Izquierda"  ->  'L'
   "Detener"    ->  'S'
         |
         v
[PySerial -- Puerto COM4]
         |
         v
[HC-06 Bluetooth -- Transmision inalambrica]
         |
         v
[Arduino -- L298N -- Motores DC]
```

---

## Evidencias

### Videos de Funcionamiento



https://github.com/user-attachments/assets/3b6ed585-a298-403e-a988-e872d8173a84


---

## Instrucciones de Uso

### Requisitos Previos

- Arduino IDE 2.3.8 o superior instalado
- Python 3.11.9 instalado
- Ollama instalado con el modelo Gemma 3 1B descargado
- Modulo HC-06 emparejado con Windows y disponible en COM4

### Paso 1 — Preparar el Arduino

```
1. Abrir Arduino IDE
2. Cargar el archivo carro_robot.ino
3. Seleccionar: Herramientas > Placa > Arduino Uno
4. Seleccionar: Herramientas > Puerto > COM3
5. Hacer clic en la flecha de carga (Subir)
6. Esperar el mensaje "Subida completada"
7. Desconectar el cable USB y alimentar con la bateria
```

### Paso 2 — Iniciar Ollama

Abrir PowerShell y ejecutar el siguiente comando. Dejar esta ventana abierta durante toda la sesion:

```
ollama serve
```

---

## Resultados y Pruebas

| Prueba | Resultado |
|---|---|
| Movimiento hacia adelante | Funcional |
| Movimiento hacia atras | Funcional |
| Giro a la derecha | Funcional |
| Giro a la izquierda | Funcional |
| Detencion del carro | Funcional |
| Emparejamiento HC-06 con Windows | Exitoso |
| Asignacion de puertos COM4 y COM5 | Exitosa |
| Envio de comandos desde PC por Bluetooth | Funcional |
| Deteccion de colores con TCS34725 | En proceso de calibracion |
| Sistema con modo de respaldo sin sensor | Funcional |
| Streamlit instalado y operativo | Confirmado |
| Ollama con Gemma 3 1B | Funcional |
| Comunicacion serial con PySerial | Funcional |

---

## Conclusiones

**Integracion hardware-software:** Se logro establecer una arquitectura funcional que integra un microcontrolador Arduino Uno con una aplicacion web en Python, demostrando la viabilidad de los sistemas embebidos conectados a interfaces web modernas mediante comunicacion inalambrica.

**Comunicacion Bluetooth bidireccional:** El modulo HC-06 fue configurado y emparejado exitosamente con Windows mediante comandos AT, permitiendo la comunicacion inalambrica entre el carro y la PC a traves de los puertos COM4 y COM5.

**Control de motores:** El driver L298N junto con los pines PWM del Arduino permitieron implementar control completo de direccion y velocidad de los motores DC del carro, validado mediante pruebas de movimiento en las cuatro direcciones.

**Inteligencia artificial local:** La implementacion del chatbot mediante Ollama y Gemma 3 demuestra que es posible integrar inteligencia artificial conversacional en proyectos academicos sin necesidad de APIs de pago ni conexion permanente a internet.

**Desafios encontrados:** La calibracion del sensor TCS34725 presento dificultades relacionadas con la comunicacion I2C, lo cual motivo el diseno de un modo de respaldo que permite al sistema operar independientemente del estado del sensor, garantizando la funcionalidad del carro en todo momento.

**Aprendizajes:** El proyecto permitio adquirir experiencia practica en sistemas embebidos, protocolos de comunicacion I2C y UART, comunicacion Bluetooth, desarrollo web con Python, y la integracion de modelos de lenguaje locales en aplicaciones academicas.

*Proyecto desarrollado con fines academicos para la asignatura de Sistemas Digitales.*
