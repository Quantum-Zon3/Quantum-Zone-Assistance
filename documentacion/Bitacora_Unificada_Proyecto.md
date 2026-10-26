---
tipo: bitacora-unificada
curso: Electiva 3 — Automatización inteligente de procesos con n8n
institucion: EAM
periodo: EAM-2026II
equipo: Carlos Mario Patiño Ramirez, Carlos Andres Lopez Perez, Miguel Angel Lievano Buitrago
corte: 2
clase: "8"
fecha: 2026-10-25
estado: borrador
tags:
  - n8n
  - bitacora
  - bitacora-unificada
---

# Bitácora unificada — Quantum Zone Assistant
---

## 1. Identificación y alcance

### 1.1 Datos generales

| Campo                  | Registro                                                                   |
| ---------------------- | -------------------------------------------------------------------------- |
| Proyecto               | Quantum Zone Assistant                                                     |
| Curso                  | Electiva 3 — Automatización inteligente de procesos con n8n (EAM, 2026 II) |
| Organización ficticia  | Quantum Zone                                                               |
| Fecha de actualización | 2026-10-25                                                                 |
| Repositorio            | https://github.com/Quantum-Zon3/Quantum-Zone-Assistance                    |

### 1.2 Integrantes y responsabilidades

| Integrante                    | Roles                   | Responsabilidad en el proyecto                          |
| ----------------------------- | ----------------------- | ------------------------------------------------------- |
| Carlos Mario Patiño Ramirez   | DevOps, Scrum Master    | Gestion de bitacora, ayudas en workflows                |
| Carlos Andres Lopez Perez     | DevOps, Product Manager | estructuración del proyecto, creación de diagramas      |
| Miguel Angel Lievano Buitrago | DevOps, QA              | creación de workflows, estructuras, manejo de los datos |

### 1.3 Problema

Quantum Zone es una organización ficticia cuyos creadores atienden de forma manual preguntas y solicitudes que se repiten, y tienen **poca automatización en sus propios procesos**. Eso les quita tiempo y hace que la información dependa de quién esté disponible para responder.

### 1.4 Quién lo enfrenta

Los **creadores** de Quantum Zone (usuarios beneficiados definidos desde la clase 2).

### 1.5 Objetivo de la automatización

Crear un asistente interno que **consulte información y ejecute acciones controladas**, capaz de responder automáticamente ciertas preguntas y de guardar en una base de datos los datos de la persona que interactúa con él.

### 1.6 Qué incluye esta versión

- Formulario de entrada de preguntas(comentarios especificos)/solicitudes. 
- Enrutamiento de la solicitud mediante un nodo Switch (rutas 1 y 2 evidenciadas en el corte 1). 
- Almacenamiento en una Data Table de n8n.
- Webhook, consumo de API con HTTP Request, subworkflow y manejo de errores (requisitos del corte 2). 

### 1.7 Qué queda por fuera

La implementación del asistente en Whatsapp y ampliar sus horizontes a ser mas escalable en la solución de problemas y accioes.

---

## 2. Proceso manual y proceso automatizado

### 2.1 Cómo se hace hoy, sin automatización

1. Llega un correo a la bandeja.

2. Un creador lo abre y decide qué es: una entrega, algo importante, spam u otra cosa.

3. Si corresponde, escribe una respuesta de recepción.

4. Si quiere dejar constancia, la anota a mano en otro lugar.

5. Para saber qué llegó hoy o cuántos correos hay, entra a Gmail y los revisa.

**Dónde aparecen los problemas:**

| Problema           | Dónde ocurre                              | Consecuencia                                    |
| ------------------ | ----------------------------------------- | ----------------------------------------------- |
| Demoras            | Entre que llega el correo y alguien lo ve | El remitente no sabe si su mensaje fue recibido |
| Errores            | Clasificar o registrar a mano             | Mensajes importantes sin atender o sin registro |
| Tareas repetitivas | Escribir la misma respuesta de acuse      | Tiempo de los creadores                         |


**Diagrama del proceso manual** (borrador, ajústenlo):

![[Pasted image 20260924132330.png]]

### 2.2 Cómo cambia con n8n


![[Pasted image 20260924132539.png]]

---

## 3. Funcionamiento de la solución

### 3.1 Recorrido de una solicitud, de principio a fin

**Ruta A — Consulta por formulario**
1. Una persona abre el formulario (_On form submission_, título "Quantum zone assistens") y pasa a la segunda página (_Form_).
2. Allí elige qué desea hacer (obtener todos los emails y/o los de hoy) y, opcionalmente, cuántos correos quiere (por defecto, 10).
3. Un nodo Code calcula el rango de fechas de hoy.
4. El nodo _Switch2_ decide la consulta: con filtro de fecha ("hoy") o sin filtro ("todos").
5. Gmail devuelve los correos solicitados. El resultado se ve en la ejecución de n8n; esta ruta **no** guarda ni envía los resultados a otro sistema.

**Ruta B — Correo entrante**
1. El *Gmail Trigger* revisa la bandeja cada minuto buscando correos del remitente configurado (incluye spam y papelera).

2. El nodo *Switch* clasifica el correo (ver 3.5).

3. Según la ruta, el workflow obtiene el mensaje completo, responde con un texto automático y prepara tres datos: remitente, asunto y resumen.

4. Llama al subworkflow, que envía esos datos por HTTP Request junto con la fecha de procesamiento.

### 3.2 Qué activa el proceso y qué datos recibe

| Entrada                             | Tipo de disparador                                       | Datos que recibe                                                                           |
| ----------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Formulario "Quantum zone assistens" | Form Trigger + nodo Form                                 | Opción(es) elegida(s) y cantidad de correos                                                |
| Gmail Trigger                       | Consulta periódica (_polling_) cada minuto               | Correo nuevo: id, asunto (_Subject_), remitente (_From_), fragmento (_snippet_), etiquetas |
| Subworkflow                         | _When Executed by Another Workflow_ (modo _passthrough_) | remitente, asunto, resumen                                                                 |
| Manejador de errores                | Error Trigger                                            | Objeto `execution` con el error y el id de la ejecución                                    |



### 3.3 Campos utilizados

| Campo                            | Dónde                               |     ¿Obligatorio?      | Para qué se usa                                        |
| -------------------------------- | ----------------------------------- | :--------------------: | ------------------------------------------------------ |
| Bienvenido ¿Qué desea hacer?     | Formulario (casillas)               | Sí, mínimo 1 selección | Decide la consulta en Switch2                          |
| Cuantos correos desea recibir?   | Formulario (número)                 |  No (por defecto 10)   | Límite de correos de Gmail                             |
| `Subject`, `snippet`             | Correo entrante                     |           -            | Detectar "adjunto la entrega" / "adjunto la evidencia" |
| labels                           | Correo entrante                     |           -            | Detectar etiquetas `IMPORTANT` y `SPAM`                |
| `remitente`, `asunto`, `resumen` | Entre el principal y el subworkflow |  Sí para el registro   | Contenido que se envía en el POST                      |

### 3.4 Limpieza, transformación y validación

- **Validación del formulario:** el nodo Form exige al menos una opción marcada.
- **Cantidad por defecto:** si la persona deja 0 (o vacío) se usan 10 correos.
- **Transformación de fechas:** el Code convierte hoy a `AAAA/MM/DD` y calcula el día siguiente para armar la consulta de Gmail.
- **Normalización de texto:** en la clasificación se pasa el asunto y el fragmento a minúsculas antes de buscar las frases.
- **Valores por defecto en el subworkflow:** si falta algún dato se envía "Remitente no especificado", "Sin Asunto" o "Sin Cuerpo".

### 3.5 Reglas de decisión y rutas
Switch2 (formulario)

| Salida   | Condición                                        | Resultado                                      |
| -------- | ------------------------------------------------ | ---------------------------------------------- |
| 0        | La selección contiene "Obtener emails de hoy"    | Gmail `getAll` con `after:` y `before:` de hoy |
| 1        | La selección contiene "Obtener todos los emails" | Gmail `getAll` sin filtro                      |
| fallback | `Ninguna (configurado en "none")`                | No hace nada                                   |

**Switch (correos entrantes)**: se evalúan en orden y gana la primera regla que cumpla.

| Salida                    | Condición                                                                 | Respuesta automática                                              | ¿Llama al subworkflow? |
| ------------------------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------- | ---------------------- |
| 0— Entrega/evidencia      | Asunto o fragmento contiene "adjunto la entrega" o "adjunto la evidencia" | "Su mensaje fue recibido y se encuentra en espera a ser leído."   | Sí                     |
| 1— Importante             | Tiene la etiqueta `IMPORTANT`                                             | "El mensaje se guardo como importante, en breve será respondido." | Sí                     |
| 2 — Spam                  | Tiene la etiqueta `SPAM`                                                  | "No me interesa, muchas gracias"                                  | No                     |
| Fallback — No contemplado | Ninguna de las anteriores                                                 | Ninguna                                                           | Sí                     |

### 3.6 Responsabilidad de cada workflow

| Workflow (archivo JSON) | Responsabilidad | Recibe | Devuelve |
|---|---|---|---|
| `Quantum_Zone_Assistens_Principal.json` | Recibir la solicitud (formulario o correo), clasificar, responder y llamar al subworkflow | Envío del formulario o correo nuevo | Respuestas por Gmail y llamada al subworkflow |
| `Quantum_Zone_Assistance_Subworkflow_Proyecto.json` | Registrar el correo procesado en un servicio externo por HTTP | `remitente`, `asunto`, `resumen` | No devuelve datos que el principal use: después del HTTP Request no hay más nodos |
| `Quantum_Zone_Assistance_Manejador_Errores.json` | Armar y enviar el reporte cuando falla otro workflow | Datos del error (`execution`) | Respuesta del POST de aviso |

Relaciones: el principal llama al subworkflow desde tres nodos *Execute Workflow* (uno por ruta), y tanto el principal como el subworkflow tienen configurado el manejador como *Error Workflow*.

### 3.7 Dónde se consulta, crea o actualiza información

| Recurso | Operación | Nodo |
|---|---|---|
| Gmail (bandeja) | Consultar varios correos | `Obtener todos los mensajes`, `Obtener todos los mensajes de hoy` |
| Gmail | Consultar un correo | `Obtener mensaje de Adjunto la evidencia`, `Obtener mensaje importante`, `Obtener mensaje de spam`, `Obtener un mensaje default` |
| Gmail | Crear (responder) | `Mensaje automatico de recibido`, `...y importante`, `...y spam` |
| webhook.site | Crear (POST) | `HTTP Request - Registrar en API Externa`, `HTTP Request - Notificar Administrador` |

### 3.8 Expresiones importantes

**1. Cantidad de correos por defecto** (nodos Gmail de la ruta A)

```
{{ $if($('Form').item.json['Cuantos correos desea recibir?'] == 0, 10, $('Form').item.json['Cuantos correos desea recibir?']) }}
```

- Para qué sirve: usa 10 si la persona dejó 0; si no, usa lo que escribió.
- Entrada → salida: `0` → `10`; `5` → `5`. `[POR COMPLETAR: probar con el campo vacío y anotar qué ocurre]`

**2. Fechas de hoy** (nodo Code)

- Para qué sirve: genera el texto que Gmail necesita para filtrar un día.
- Entrada → salida: sin fecha de consulta, usa la fecha actual. Si hoy es 2026-10-25 → `fechaInicio = "2026/10/25"`, `fechaFin = "2026/10/26"`. El nodo Gmail arma `after: 2026/10/25 before: 2026/10/26`.

**3. Regla de entrega/evidencia** (Switch, salida 0)

```
($json.snippet ?? "").toLowerCase().includes("adjunto la entrega") || ... "adjunto la evidencia"
```

- Para qué sirve: detecta las frases sin importar mayúsculas y sin fallar si el campo viene vacío (`?? ""`).
- Entrada → salida: asunto "Adjunto la Entrega del taller" → `true`; asunto "Hola" → `false`.

**4. Regla de importante** (Switch, salida 1)

```
$json.labels.some(label => label.id === "IMPORTANT")
```

- Para qué sirve: revisa si alguna etiqueta del correo es `IMPORTANT`.
- Entrada → salida: `labels = [{id:"IMPORTANT"}]` → `true`; `labels = [{id:"INBOX"}]` → `false`.

**5. Datos para el subworkflow** (nodo Set)

```
{{ $('Obtener mensaje de Adjunto la evidencia').item.json.From }}
```

- Para qué sirve: toma el remitente del correo obtenido por Gmail.
- Entrada → salida: `From = "Ana <ana@ejemplo.com>"` → `remitente = "Ana <ana@ejemplo.com>"`.
- ⚠ Ver limitación L1 en la sección 7.6.

**6. Valores por defecto en el subworkflow**

```
{{ $json.remitente || $json['De parte de:'] || 'Remitente no especificado' }}
```

- Para qué sirve: usa el primer valor disponible.
- Entrada → salida: `remitente` vacío y `De parte de:` = "Ana" → "Ana"; ambos vacíos → "Remitente no especificado".

---

## 4. Integraciones y configuración

### 4.1 Servicios externos

| Servicio | Función | Tipo de nodo |
|---|---|---|
| Gmail | Leer correos (consulta y trigger) y enviar respuestas | Nodos nativos Gmail y Gmail Trigger |
| webhook.site | Recibir el registro del correo y el reporte de errores (receptor de pruebas, **simulado**) | HTTP Request |

### 4.2 Llamadas con HTTP Request

**Subworkflow — `HTTP Request - Registrar en API Externa`**

| Campo | Detalle |
|---|---|
| Método | POST |
| Endpoint | `https://webhook.site/<tu-URL-única>` `[CONFIRMAR: en el JSON aparece solo https://webhook.site, cada persona debe crear su propia URL]` |
| Cuerpo (*multipart-form-data*) | `proyecto`, `remitente`, `asunto`, `resumen`, `fecha_procesado` |
| Respuesta relevante | `[POR COMPLETAR: código HTTP y cuerpo que se vio en webhook.site]` |
| Paginación | No aplica: se envía un registro por llamada |

**Manejador — `HTTP Request - Notificar Administrador`**

| Campo | Detalle |
|---|---|
| Método | POST |
| Endpoint | `https://webhook.site/<tu-URL-única>` |
| Cuerpo (JSON) | `resumen`, `nodo`, `error`, `id_ejecucion` |
| Respuesta relevante | `[POR COMPLETAR]` |

### 4.3 Credenciales y permisos (sin valores)

| Credencial | Servicio | Permisos | Dónde se crea |
|---|---|---|---|
| Gmail OAuth2 (en el JSON se llama "Gmail account") | Gmail | Leer correos y enviar respuestas `[CONFIRMAR: permisos que pidió al autorizar]` | Credentials de n8n, con un cliente OAuth propio de Google Cloud |

El JSON del principal conserva el nombre de la credencial con un id de prueba (`CREDENCIAL_DE_PRUEBA`): después de importar hay que **volver a seleccionar la credencial propia en cada nodo Gmail**.

### 4.4 Recursos que debe preparar quien configure su copia

- Una cuenta de Gmail con la credencial OAuth2 autorizada.
- Una URL propia de webhook.site (gratuita) para recibir los registros y los reportes de error.
- En el *Gmail Trigger*, cambiar el remitente de ejemplo (`example@gmail.com`) por el correo desde el que se enviarán las pruebas.
- Reasignar el subworkflow en los tres nodos *Execute Workflow* y el manejador en *Settings → Error Workflow* del principal y del subworkflow.

| Campo              | Detalle                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------- |
| URL del formulario | `[POR COMPLETAR: ruta /form/... de prueba y de producción]`                                                         |
| Cómo probarlo      | Ejecutar el workflow, abrir la URL de prueba del formulario y enviar los datos                                      |
| Túnel              | `[POR COMPLETAR: si usaron uno, cómo se inicia]`. Una dirección temporal puede cambiar o dejar de estar disponible. |

---

## 5. Errores y recuperación

### 5.1 Qué pasa en cada situación

| Situación | Tipo | Qué ocurre | Dónde queda el registro |
|---|---|---|---|
| No se marca ninguna opción en el formulario | Respuesta de negocio | El formulario no permite enviar (mínimo 1 selección) | En la propia página del formulario |
| Cantidad = 0 o vacía | Respuesta de negocio | Se usan 10 correos | — |
| El correo no encaja en ninguna regla | Respuesta de negocio | Va a la ruta "no contemplado" y se registra por el subworkflow | webhook.site |
| Falla Gmail, el subworkflow o una expresión | **Error técnico** | n8n ejecuta el manejador de errores | Historial de ejecuciones de n8n y POST del reporte a webhook.site |

### 5.2 Rutas de error, reintentos y manejador

- **Error Workflow:** el principal y el subworkflow apuntan al manejador (`Quantum Zone Assistance Manejador_Errores`).
- **Qué hace el manejador:** *Error Trigger* → *Estructurar Reporte* (alerta, mensaje de error e id de ejecución) → POST a webhook.site.
- **Reintentos:** no se configuraron reintentos automáticos en los nodos .
- **Salidas de error por nodo:** no se usan; el manejo es global.
- **Importante para probar:** el Error Workflow solo se dispara en ejecuciones **automáticas** (workflow publicado/activo), no cuando se pulsa "Execute workflow" a mano.

### 5.3 Recuperación y no duplicación

- Una ejecución fallida se puede repetir desde el historial de ejecuciones de n8n.
- **No hay un mecanismo implementado contra duplicados:** si se repite una ejecución de la ruta B, se volvería a enviar la respuesta automática y a registrar el correo. Propuesto: guardar el id del mensaje de Gmail ya procesado y consultarlo antes de responder. 

---

## 6. Pruebas y evidencias

> Mínimo: caso normal, entrada incompleta o no válida y fallo técnico controlado, más las decisiones principales. Abajo dejé las pruebas diseñadas con entradas y resultados esperados según los JSON; falta ejecutarlas y llenar lo observado.

### Prueba 1 — Caso normal (consulta del día)

| Campo      | Registro                                                     |
| ---------- | ------------------------------------------------------------ |
| Entrada    | Formulario: "Obtener emails de hoy", cantidad 5              |
| Pasos      | Ejecutar el principal, abrir el formulario de prueba, enviar |
| Tipo       | Manual                                                       |
| Esperado   | Gmail devuelve como máximo 5 correos de hoy                  |

### Prueba 2 — Entrada no válida o incompleta

| Campo      | Registro                                 |
| ---------- | ---------------------------------------- |
| Entrada    | Formulario sin ninguna casilla marcada   |
| Pasos      | Intentar enviar el formulario            |
| Tipo       | Manual                                   |
| Esperado   | El formulario no deja enviar             |


### Prueba 3 — Fallo técnico controlado

| Campo           | Registro                                                                                                    |
| --------------- | ----------------------------------------------------------------------------------------------------------- |
| Cómo se provoca | En una **copia** de trabajo, dejar una credencial inválida o un nombre de nodo mal escrito en una expresión |
| Pasos           | Activar el workflow y enviar un correo de prueba (el manejador no corre en pruebas manuales)                |
| Tipo            | Automática                                                                                                  |
| Esperado        | n8n ejecuta el manejador y llega el POST con `resumen`, `nodo`, `error` e `id_ejecucion` a webhook.site     |


### Pruebas 4–7 — Una por cada ruta del Switch

| #   | Correo de prueba                      | Esperado                                         |
| --- | ------------------------------------- | ------------------------------------------------ |
| 4   | Asunto "Adjunto la entrega de prueba" | Respuesta de recibido + registro en webhook.site |
| 5   | Correo marcado como importante        | Respuesta "guardó como importante" + registro    |
| 6   | Correo en spam                        | Respuesta "No me interesa" y **sin** registro    |
| 7   | Correo con asunto cualquiera          | Sin respuesta automática, con registro           |

Pruebas sueltas recomendadas: ejecutar el subworkflow con datos fijados (para ver el POST) y el manejador con datos de ejemplo del Error Trigger.

> Las rutas 5 y 7 tienen una posible falla de referencia de nodos (limitación L1). Si la prueba falla, **es un hallazgo válido**: regístrenlo, corríjanlo y repitan la prueba.

---

## 7. Evolución y estado final

### 7.1 Qué cambió desde el primer corte

- **Clase 2 (24/08):** ambiente de trabajo (carpeta compartida y bóveda), instalación de n8n y Obsidian, primer workflow con una decisión IF (resultados true/false).
- **Clase 3 (31/08):** elección del proyecto, revisión en n8n de cómo construirlo y charla con el docente; workflow con formulario y Switch.
- **Clase 4 (07/09):** ajuste de Edit Fields con campos reales y avance del asistente.
- **Corte 2:** el proyecto evolucionó hacia la gestión de correo con Gmail: clasificación de correos entrantes, respuestas automáticas, subworkflow con HTTP Request y manejador de errores.

### 7.2 Decisiones técnicas importantes

| Decisión                                    | Razón                                                   | Alternativa descartada                                                                     |
| ------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Usar Gmail como fuente y canal de respuesta | Debido al tiempo y la poca funcionn gratuita de Twilo   | Utilizar la oficial de Whatsapp (Se tratara de implementar mas adelante bien estructurado) |
| Separar el registro en un subworkflow       | Una responsabilidad clara y reutilizable por tres rutas | Repetir el HTTP Request en cada ruta                                                       |
| Un manejador de errores global              | Centralizar el reporte de fallos                        | Salidas de error en cada nodo                                                              |
| webhook.site como receptor                  | Probar HTTP Request sin contratar un servicio           | NO HAY                                                                                     |

### 7.3 Dificultades y soluciones

| Dificultad                                  | Causa                   | Solución                                                                          | Cómo se verificó             |
| ------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------- | ---------------------------- |
| Falta de conocimiento inicial de n8n        | Poca experiencia previa | Aprendizaje de nociones básicas con apoyo de IA, contrastado con la documentación | Workflows creados y probados |


### 7.4 Aportes de los integrantes

| Integrante | Aportes verificables |
|---|---|
| Carlos Mario Patiño Ramirez | Workflow de la clase 2; bitácora de la clase 3; apoyo en la bitácora de la clase 4. `[POR COMPLETAR: corte 2 + evidencia]` |
| Carlos Andres Lopez Perez | Carpeta y bóveda compartidas; corrección de la bitácora de la clase 4. `[POR COMPLETAR: corte 2 + evidencia]` |
| Miguel Angel Lievano Buitrago | Workflow de la clase 3; edición del workflow de la clase 4. `[POR COMPLETAR: corte 2 + evidencia]` |

### 7.5 Uso de herramientas de apoyo e IA

| Herramienta                                                       | Para qué                         | Cómo se verificó     | Qué se modificó                          |
| ----------------------------------------------------------------- | -------------------------------- | -------------------- | ---------------------------------------- |
| ChatGPT (clase 2)                                                 | Conceptos clave de n8n           | Documentación de n8n | No se copiaron elementos de la respuesta |
| Gemini (clases 3 y 4)                                             | Lógica de tablas de datos en n8n | Documentación de n8n | No se copiaron elementos de la respuesta |


### 7.6 Estado final

| Funcionalidad | Estado |
|---|---|
| Formulario de consulta de correos | Implementado `[CONFIRMAR con prueba]` |
| Clasificación de correos con Switch (4 rutas) | Implementado `[CONFIRMAR con pruebas 4–7]` |
| Respuestas automáticas por Gmail | Implementado `[CONFIRMAR]` |
| Subworkflow con HTTP Request | Implementado, con destino **simulado** (webhook.site) |
| Manejador de errores | Implementado, aviso **simulado** (POST a webhook.site) |
| Webhook de entrada | Pendiente (ver 4.5) |
| Almacenamiento permanente de datos | Propuesto |
| Reintentos y prevención de duplicados | Propuesto |
| Inteligencia artificial en el asistente | Propuesto (corte 3) |

**Limitaciones conocidas** ]`

- **L1.** Los tres nodos *Formato de datos para el Subworkflow* leen los datos del nodo `Obtener mensaje de Adjunto la evidencia`, incluso en las rutas "importante" y "no contemplado", donde ese nodo no se ejecuta. Es probable que esas rutas fallen y activen el manejador.
- **L2.** En el manejador, el campo `Nodo_Causante` usa el mismo valor que `Detalle_Error` (el mensaje), por lo que no identifica el nodo que falló.
- **L3.** El cuerpo JSON del manejador se arma con texto: si el mensaje de error trae comillas o saltos de línea, el propio aviso podría fallar.
- **L4.** El *Switch2* envía solo a la primera salida que cumple: si se marcan ambas casillas, solo se ejecuta "hoy".
- **L5.** La ruta de spam responde "No me interesa" al remitente y no se registra.
- **L6.** La consulta del formulario no entrega ni guarda el resultado fuera de la ejecución de n8n.
- **L7.** El Gmail Trigger tiene un remitente de ejemplo que cada persona debe cambiar.
