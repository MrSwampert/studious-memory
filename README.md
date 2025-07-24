# Proyecto Final: Sistema de Control de Acceso y Climatización Remoto

El diseño y la implementación de un sistema integrado completo para la gestión inteligente de una habitación, conforman este proyecto. El sistema, gestionado por un microcontrolador STM32, integra una interfaz de usuario clara a través de una pantalla OLED, un sistema automatizado de gestión climática que mantiene una temperatura ambiente y un control de acceso seguro mediante un teclado numérico.

El MCU STM32 es el microcontrolador principal. Es el encargado de ejecutar el firmware para que lea los sensores, procesar la lógica, y activar las salidas. El chip corre una máquina de estados y un super loop para ser rápido y eficiente, asegurando que nunca se quede colgado.

# Integrantes:

Juan Diego Restrepo Hernández

Santiago Galeano Castaño

# Arquitectura del Hardware:

Diagrama de bloques:

<img width="3380" height="3840" alt="Mermaid Chart - Create complex, visual diagrams with text  A smarter way of creating diagrams -2025-07-24-015713" src="https://github.com/user-attachments/assets/2d12c53d-6074-4555-8c93-d8d341c8660b" />



**Las entradas:**

- **El Teclado Matricial 4x4** Es la principal forma de interacción local del usuario. Permite introducir la **contraseña de 4 dígitos** para desbloquear el sistema y navegar por los menús para configurar la climatización.

- El **Sensor de Temperatura**, **se usa un sensor BMP280**. Este sensor es crucial para la función de la climatización. En este caso, el MCU lo lee cada dos segundos para saber la temperatura del ambiente y así decidir si debe encender el ventilador (por el calor) o el calefactor (por el frío). 

Estos son los dispositivos que le envían información al STM32 para que tome decisiones.


**Las salidas:**

- La **pantalla OLED 0.96** es la interfaz visual principal. Muestra los mensajes como el Sistema Bloqueado, solicitar la clave y presenta el menú de opciones junto con los datos de temperatura.

- El **ventilador** forma parte del sistema de climatización. Se lelga a activar automáticamente cuando la temperatura supera el objetivo establecido. El control de este se realiza por PWM.

- El **calefactor** es el otro componente clave de la climatización, ya que se enciende cuando la temperatura baja del umbral deseado.

- El **LED de puerta abierta** sirve como un indicador visual del estado de la puerta o del sistema

- Finalmente el **LED de estado**, es el que parpadea constantemente. La única función, es confrmar visualmente que el sistema esta en cendido y esta corriendo sin fallos, lo que es conocido como heatbeat.

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

El núcleo de esta función se basa en las mediciones precisas obtenidas por el **sensor**, que constantemente informa la **temperatura ambiente al microcontrolador**. Este valor se compara en tiempo real contra la temperatura objetivo que el usuario ha configurado previamente.

El algoritmo de decisión actúa de la siguiente manera:

- Si la lectura del **sensor** cae por debajo del objetivo, el sistema activa el calefactor para elevar la temperatura.

- Si la lectura supera el objetivo, se enciende el ventilador para refrescar el ambiente.

**El proceso es un ciclo constante:** cada dos segundos, el microcontrolador consulta el sensor de temperatura para obtener una lectura precisa del ambiente. Luego, compara esta lectura con la temperatura objetivo fijada por el usuario para decidir si es necesario activar el ventilador (si hace calor) o el calefactor (si hace frío), manteniendo así la habitación siempre confortable.

Para garantizar un funcionamiento eficiente y evitar el desgaste de los componentes, el sistema no reacciona a cada pequeña fluctuación. En su lugar, se implementa un **control por histéresis.**

Esto significa que existe un margen de **tolerancia** alrededor de la **temperatura objetivo**. El **ventilador** o el **calefactor** solo se activarán cuando la temperatura cruce completamente este margen, evitando así que los equipos se enciendan y apaguen repetidamente por cambios mínimos. Esto resulta en un control más estable y un menor consumo energético.

El sistema está diseñado para ser controlado no solo localmente, sino también a distancia.

**Doble Canal de Comunicación:**

Para lograrlo, se emplean dos puertos serie **UART** independientes. Esta configuración es clave para un funcionamiento robusto:

- Un canal se reserva para la **depuración y programación** del sistema a través de una conexión directa con el PC.

- El segundo canal está dedicado exclusivamente a la conectividad externa, diseñado para interactuar con módulos de expansión, como podría ser un **ESP01 WiFi**.

Esta separación de canales permite que el sistema esté preparado para integrarse con plataformas de IoT o aplicaciones externas. La **arquitectura del firmware** está estructurada para interpretar directivas enviadas a través del puerto de comunicación externo.

Esto sienta las bases para que, en futuras implementaciones, un sistema remoto pueda:

- **Monitorear** el estado completo del dispositivo.
- **Modificar** parámetros de funcionamiento, como la temperatura objetivo.
- **Gestionar** las credenciales de acceso de forma segura.

# Arquitectura de Firmware:

Para garantizar un sistema robusto, que responda al instante y nunca se congele, el **firmware** se construyó sobre dos pilares fundamentales que trabajan en perfecta sincronía. Una organización lógica estricta y un motor de ejecución de **alta eficiencia**.

**La Organización: Arquitectura de Máquina de Estados**

A alto nivel, toda la lógica del sistema se organiza como una **Máquina de Estados Finita** (FSM). Esto significa que el programa no es un caos de funciones, sino un organismo que solo puede existir en un estado bien definido a la vez (por ejemplo, BLOQUEADO, INGRESANDO_CLAVE o DESBLOQUEADO).

Cada estado actúa como una **burbuja** con sus **propias reglas**, definiendo qué acciones son posibles y cuáles deben ser ignoradas.

- **Ejemplo Práctico:** En el estado **BLOQUEADO**, el sistema es sordo a cualquier tecla del menú. Su única preocupación es detectar el inicio de un intento de acceso.

- **Ventaja Principal:** Este modelo impone un orden increíble, haciendo que el código sea predecible y seguro. Previene errores lógicos (como intentar abrir un menú mientras la puerta está bloqueada) y facilita enormemente la adición de nuevas funcionalidades sin romper las existentes.

**El Motor: Bucle Principal No Bloqueante**

A bajo nivel, el motor que impulsa esta máquina de estados es un bucle principal no bloqueante (también conocido como Superloop).

A diferencia de un enfoque simple que usaría pausas (HAL_Delay()), el bucle implementado es un ciclo de sondeo ultrarrápido que nunca se detiene a esperar. En cada vuelta, que dura microsegundos, pregunta a cada componente: ¿Hay algo nuevo para mí?. Revisando si se ha presionado una tecla, si ha llegado un comando por el puerto serie o si es momento de medir la temperatura.

- **Ventaja Principal:** El procesador nunca está ocupado esperando. Siempre está disponible y listo para reaccionar de inmediato ante cualquier evento.

# Protocolo de Comandos:


# Optimización:

Un firmware robusto no solo debe ser funcional, sino también eficiente. El diseño se enfoca en tres áreas clave de optimización para garantizar un rendimiento superior.

**Capacidad de Respuesta Instantánea**

La prioridad es que el sistema reaccione al instante. Esto se logra combinando dos técnicas:

- **Arquitectura No Bloqueante:** El corazón del firmware es un bucle principal que nunca se detiene con pausas (como HAL_Delay()). Este motor de ejecución ultrarrápido asegura que el procesador siempre esté disponible para atender cualquier evento sin demora.

- **Manejo por Interrupciones:** En lugar de que el procesador malgaste recursos preguntando constantemente si se ha pulsado una tecla (polling), se usan interrupciones. El propio hardware (teclado, puerto serie) avisa activamente al procesador cuando ocurre un evento, permitiéndole ahorrar energía y tiempo de cómputo.

**Fluidez en la Interfaz de Usuario**

- Redibujar la pantalla OLED es una operación lenta que puede consumir muchos recursos. Para optimizarla, la actualización de la pantalla es condicional. Se utiliza una bandera de software que solo se activa cuando un dato relevante (como la temperatura o el estado del sistema) ha cambiado. De esta forma, la pantalla solo se refresca cuando es estrictamente necesario, lo que resulta en una interfaz fluida, sin parpadeos, y libera al procesador para otras tareas.

**Fiabilidad en las Mediciones Críticas**

- La precisión del termostato depende directamente de la correcta lectura del sensor. La comunicación con la termocupla requiere una transacción de datos que no debe ser interrumpida para garantizar su integridad.

- Para lograrlo, se implementa una sección crítica: justo antes de iniciar la comunicación con el sensor, el firmware deshabilita momentáneamente otras interrupciones (como las del teclado). Inmediatamente después de obtener la lectura, las interrupciones se reactivan. Este breve instante de "concentración" exclusiva asegura que cada medición de temperatura sea precisa y fiable, lo cual es fundamental para el correcto funcionamiento del sistema.
