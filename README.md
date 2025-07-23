# Proyecto Final: Sistema de Control de Acceso y Climatización Remoto

El diseño y la implementación de un sistema integrado completo para la gestión inteligente de una habitación, conforman este proyecto. El sistema, gestionado por un microcontrolador STM32, integra una interfaz de usuario clara a través de una pantalla OLED, un sistema automatizado de gestión climática que mantiene una temperatura ambiente y un control de acceso seguro mediante un teclado numérico.

El MCU STM32 es el microcontrolador principal. Es el encargado de ejecutar el firmware para que lea los sensores, procesar la lógica, y activar las salidas.El chip corre una máquina de estados y un super loop para ser rápido y eficiente, asegurando que nunca se quede colgado.

# Integrantes:

Juan Diego Restrepo Hernández

Santiago Galeano Castaño

# Arquitectura del Hardware:

Diagrama de bloques:

<img width="3433" height="3840" alt="Mermaid Chart - Create complex, visual diagrams with text  A smarter way of creating diagrams -2025-07-23-200716" src="https://github.com/user-attachments/assets/3318207f-a0e4-4fa0-a936-2f5d0a8a129d" />


**Las entradas:**

- **El Teclado Matricial 4x4** Es la principal forma de interacción local del usuario. Permite introducir la **contraseña de 4 dígitos** para desbloquear el sistema y navegar por los menús para configurar la climatización.

- El **Sensor de Temperatura y Humedad**, **se usa una termocupla**. Este sensor es crucial para la función de la climatización. En este caso, el MCU lo lee cada dos segundos para saber la temperatura y humedad del ambiente y así decidir si debe encender el ventilador (por el calor) o el calefactor (por el frío). 

Estos son los dispositivos que le envían información al STM32 para que tome decisiones.


**Las salidas:**

- La **pantalla OLED 0.96** es la interfaz visual principal. Muestra los mensajes como el Sistema Bloqueado, solicitar la clave y presenta el menú de opciones junto con los datos de temperatura y humedad. 

- El **ventilador** forma parte del sistema de climatización. Se lelga a activar automáticamente cuando la temperatura supera el objetivo establecido. El control de este se realiza por PWM.

- El **calefactor** es el otro componente clave de la climatización, ya que se enciende cuando la temperatura baja del umbral deseado.

- El **LED de puerta abierta** sirve como un indicador visual del estado de la puerta o del sistema

- Finalmente el **LED de estado**, es el que parpadea constantemente. La única funcion, es confrmar visualmente que el sistema esta en cendido y esta corriendo sin fallos, lo que es conocido como heatbeat.

Estos son los dispositivos que el STM32 controla para interactuar con el mundo físico y el usuario.

# Funcionalidad

**Control de Acceso por Teclado**

El núcleo de la seguridad del sistema es un control de acceso que gestiona la cerradura electrónica, operando a través de varios estados definidos.

- **Estado Inicial (BLOQUEADO):** Por defecto, el sistema se encuentra inactivo y seguro. La pantalla OLED muestra el mensaje "SISTEMA BLOQUEADO" y espera la interacción del usuario.

- **Ingreso de Clave:** Al presionar la primera tecla, el sistema transita al estado de ingreso de clave. La pantalla solicita la contraseña y muestra un asterisco (*) por cada uno de los 4 dígitos introducidos para mantener la privacidad.

- **Validación:** Una vez que se ingresan los cuatro dígitos, el sistema los compara con la contraseña almacenada.

- **Acceso Concedido:** Si la clave es correcta, el sistema pasa al estado DESBLOQUEADO. La cerradura electrónica se abre, y la pantalla muestra un mensaje de confirmación antes de dar acceso a las funciones de climatización y a los menús de navegación.

- **Acceso Denegado:** Si la clave es incorrecta, la pantalla muestra el mensaje "ACCESO DENEGADO" durante 5 segundos. Simultáneamente, se envía una alerta por el puerto serie y el sistema regresa automáticamente al estado BLOQUEADO.

- **Re-bloqueo del Sistema:** El sistema puede volver al estado BLOQUEADO de dos maneras, manual y automático.

 - **Manualmente:** El usuario puede presionar la tecla * en cualquier momento desde el estado desbloqueado.

 - **Automáticamente:** Por seguridad, si el sistema detecta 10 segundos de inactividad durante el ingreso de la clave, cancela la operación y se bloquea de nuevo.

Una vez que el acceso es concedido, el sistema cobra vida y presenta su interfaz principal en la pantalla OLED, mostrando en tiempo real la temperatura y humedad de la habitación.

Paralelamente, un LED de estado (conocido como heartbeat) parpadea de forma constante en la placa. Este parpadeo es la señal visual que confirma que el sistema está operando correctamente y no se ha bloqueado.

Desde esta pantalla principal, el usuario puede presionar la tecla '1' para entrar al menú de operaciones, donde puede tomar el control manual del sistema.

**Menú de Navegación:**

Este menú le da al usuario el poder de anular temporalmente el modo automático para ajustar el ambiente a su gusto. Las opciones son:

- Ajustar el Termostato: Permite establecer la temperatura de confort deseada que el sistema usará como referencia.

- Forzar Ventilación: Ofrece la capacidad de encender o apagar el ventilador manualmente, sin importar la temperatura actual.

- Activar Calefactor: De igual manera, permite controlar el calefactor directamente.

**Sistema de Climatización Automático**

El sistema opera como un termostato inteligente cuyo objetivo es mantener una temperatura de confort de manera totalmente autónoma.

El núcleo de esta función se basa en las mediciones precisas obtenidas por una **termocupla**, que constantemente informa la **temperatura ambiente al microcontrolador**. Este valor se compara en tiempo real contra la temperatura objetivo que el usuario ha configurado previamente.

El algoritmo de decisión actúa de la siguiente manera:

- Si la lectura de la **termocupla** cae por debajo del objetivo, el sistema activa el calefactor para elevar la temperatura.

- Si la lectura supera el objetivo, se enciende el ventilador para refrescar el ambiente.

**El proceso es un ciclo constante:** cada dos segundos, el microcontrolador consulta el sensor de temperatura para obtener una lectura precisa del ambiente. Luego, compara esta lectura con la temperatura objetivo fijada por el usuario para decidir si es necesario activar el ventilador (si hace calor) o el calefactor (si hace frío), manteniendo así la habitación siempre confortable.

Para garantizar un funcionamiento eficiente y evitar el desgaste de los componentes, el sistema no reacciona a cada pequeña fluctuación. En su lugar, se implementa un **control por histéresis.**

Esto significa que existe un margen de **tolerancia** alrededor de la **temperatura objetivo**. El **ventilador** o el **calefactor** solo se activarán cuando la temperatura cruce completamente este margen, evitando así que los equipos se enciendan y apaguen repetidamente por cambios mínimos. Esto resulta en un control más estable y un menor consumo energético.

