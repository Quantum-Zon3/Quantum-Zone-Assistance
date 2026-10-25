### Quantum Zone Assistens 🤖✨

### Sistema Modular Automatizado de Clasificación de Requerimientos y Tolerancia a Fallos en n8n

Este repositorio contiene la entrega oficial para el **Segundo Corte** de la asignatura *Automatización Inteligente de Procesos con n8n* (Periodo 2026 II) de la **Institución Universitaria EAM**. 

Quantum Zone Assistens es un ecosistema desacoplado de automatización diseñado para optimizar flujos de comunicación electrónica. Resuelve el problema de la gestión manual ineficiente de correspondencia mediante la interceptación de eventos en tiempo real, clasificación lógica condicional, normalización de datos en origen y retransmisión segura hacia APIs externas. 

### 👥 Integrantes del Equipo

* **Miguel Angel Lievano Buitrago** - *Responsable de la creación y mantenimiento de los workflows*
* **Carlos Mario Patiño Ramirez** - *Responsable de la investigación de herramientas y credenciales necesarias*
* **Carlos Andres Lopez Perez** - *Responsable de Documentación y auditor de pruebas*

### 🏗️ Arquitectura del Ecosistema y Estructura de Archivos

Para cumplir con los estándares académicos y garantizar la mantenibilidad, la lógica de la solución se encuentra estrictamente segmentada en tres archivos JSON independientes dentro de la carpeta /workflows: 

1. **Principal_Proyecto.json:** Controla las compuertas de entrada (Formulario/Gmail), ejecuta la bifurcación lógica condicional mediante un nodo Switch y sanitiza los datos mediante un nodo Edit Fields antes de invocar el subworkflow.
2. **Subworkflow_Proyecto.json:** Actúa como un componente reutilizable y desacoplado. Hereda la estructura limpia del flujo padre y consume una API de terceros mediante peticiones HTTP Request.
3. **Manejador_Errores.json:** Componente perimetral pasivo conectado a nivel de canvas global. Intercepta excepciones de red o autenticación de forma asíncrona para reportar fallas sin tumbar el servidor principal.

### Mapa del Flujo de Datos

mermaid

graph TD
    A[Form Trigger / Formulario] --> B(Workflow Principal)
    C[Gmail Trigger / Escucha Activa] --> B
    B --> D{Clasificación Switch}
    D -->|Evidencia / Importante / Extra| E[Edit Fields: Sanitización]
    D -->|Ruta Spam| F[Mensaje de Rechazo y Fin]
    E -->|Mapeo Limpio| G[Execute Workflow Node]
    G --> H[Subworkflow: Receptor]
    H --> I[HTTP Request POST] --> J[API Destino: Webhook.site]
    B -.->|En caso de Falla Técnica| K[Manejador de Errores Global]
    H -.->|En caso de Falla Técnica| K


### 🛠️ Requisitos de Configuración e Importación

Por directrices estrictas de **Desarrollo Seguro y Limpieza de Secretos**, todas las credenciales de la API de Gmail, tokens privados y endpoints específicos han sido removidos de los archivos JSON cargados en este repositorio. Siga estos pasos para su despliegue local: 

### Paso 1: Jerarquía del Repositorio

Asegúrese de clonar este proyecto respetando la estructura de carpetas exigida por la universidad: 


Quantum_Zone_Assistens/
├── README.md
├── workflows/
│   ├── Principal_Proyecto.json
│   ├── Subworkflow_Proyecto.json
│   └── Manejador_Errores.json
└── documentacion/
    ├── Bitacora_Unificada_Proyecto.md
    └── evidencias/


### Paso 2: Importación Secuencial en n8n

Cree tres workflows en blanco en su instancia de n8n e importe los JSONs desde la interfaz gráfica de la plataforma en el siguiente orden estricto para evitar referencias huérfanas: 

1. Importar e iniciar Manejador_Errores.json.
2. Importar e iniciar Subworkflow_Proyecto.json.
3. Importar e iniciar Principal_Proyecto.json.

### Paso 3: Vinculación e Infraestructura

1. **Configurar Gmail:** Abra los nodos de Gmail en el flujo principal y asocie sus propias credenciales OAuth2 activas en su Google Cloud Console.
2. **Configurar Remitente en el Gmail Trigger:** Para que este Trigger funcione debe colocar el correo de el remitente en el apartado "Sender".
3. **Configurar Endpoint:** Abra el subworkflow y el manejador de errores, entre en sus nodos HTTP Request y reemplace el valor temporal de la URL por su propio endpoint exclusivo generado en Webhook.site.
4. **Activar Gestión de Errores:** En el lienzo del *Workflow Principal* y del *Subworkflow*, ingrese al ícono de engranaje (Settings del canvas) en la esquina superior derecha y seleccione Manejador_Errores en la opción **Error Workflow**.

### 🧪 Casos de Prueba Ejecutables y Matriz de Evidencias

El comportamiento del ecosistema está blindado para responder a tres escenarios operativos críticos, cuyas capturas de pantalla de respaldo se encuentran detalladas en la bitácora unificada: 

Caso de PruebaEntrada SimuladaComportamiento del SistemaResultado en Webhook.site (Evidencia)
****1. Caso Normal****
Correo o formulario con Asunto y Cuerpo válidos.El Switch clasifica la ruta, el nodo Edit Fields encapsula las variables reales y el subworkflow despacha el lote con éxito.Recibe un JSON dinámico con los valores reales del remitente y contenido con código **HTTP 200 OK**.
****2. Entrada No Válida****
Carga de datos incompleta (campos vacíos).Las expresiones lógicas con operador `
****3. Fallo Controlado****
Desconexión intencional o host falso en el HTTP Request.El nodo del subworkflow falla en rojo, n8n detiene el editor interactivo y desvía la carga útil en segundo plano hacia el manejador.La sección **Executions** de n8n registra el flujo padre en rojo (Failed) y el flujo de Manejador_Errores en verde (Success).
