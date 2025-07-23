# Proyecto Final: Sistema de Control de Acceso y Climatización Remoto

# Arquitectura del Hardware:

Diagrama de bloques:

<img width="3433" height="3840" alt="Mermaid Chart - Create complex, visual diagrams with text  A smarter way of creating diagrams -2025-07-23-200716" src="https://github.com/user-attachments/assets/3318207f-a0e4-4fa0-a936-2f5d0a8a129d" />

El diseño y la implementación de un sistema integrado completo para la gestión inteligente de una habitación, conforman este proyecto. El sistema, gestionado por un microcontrolador STM32, integra una interfaz de usuario clara a través de una pantalla OLED, un sistema automatizado de gestión climática que mantiene una temperatura ambiente y un control de acceso seguro mediante un teclado numérico.

El MCU STM32 es el microcontrolador principal. Es el encargado de ejecutar el firmware para que lea los sensores, procesar la lógica, y activar las salidas.El chip corre una máquina de estados y un super loop para ser rápido y eficiente, asegurando que nunca se quede colgado.

# Las entradas:

- **El Teclado Matricial 4x4** Es la principal forma de interacción local del usuario. Permite introducir la **contraseña de 4 dígitos** para desbloquear el sistema y navegar por los menús para configurar la climatización.

- El **Sensor de Temperatura y Humedad**, **se usa una termocupla**. Este sensor es crucial para la función de la climatización. En este caso, el MCU lo lee cada dos segundos para saber la temperatura y humedad del ambiente y así decidir si debe encender el ventilador (por el calor) o el calefactor (por el frío). 

Estos son los dispositivos que le envían información al STM32 para que tome decisiones.

# Las salidas:

- La **pantalla OLED 0.96** es la interfaz visual principal. Muestra los mensajes como el Sistema Bloqueado, solicitar la clave y presenta el menú de opciones junto con los datos de temperatura y humedad. 

- El **ventilador** forma parte del sistema de climatización. Se lelga a activar automáticamente cuando la temperatura supera el objetivo establecido. El control de este se realiza por PWM.

- El **calefactor** es el otro componente clave de la climatización, ya que se enciende cuando la temperatura baja del umbral deseado.

- El **LED de puerta abierta** sirve como un indicador visual del estado de la puerta o del sistema

- Finalmente el **LED de estado**, es el que parpadea constantemente. La única funcion, es confrmar visualmente que el sistema esta en cendido y esta corriendo sin fallos, lo que es conocido como heatbeat.

Estos son los dispositivos que el STM32 controla para interactuar con el mundo físico y el usuario.

