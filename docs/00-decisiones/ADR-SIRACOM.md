# SIRACOM — Registro de decisiones arquitectónicas (ADR) y contexto para el agente

**Versión del documento:** 1.0 (definitivo para la fase de contexto)
**Fecha:** 26 de octubre de 2026
**Ubicación sugerida en el repositorio:** `docs/00-decisiones/ADR-SIRACOM.md`
**Destinatario:** agente de desarrollo (codeador)

---

## 0. Lectura obligatoria antes de hacer cualquier cosa

### 0.1 Fase actual: solo contexto y diseño

Estamos en la **fase de contexto**. Este documento se entrega para que el agente conozca todo lo decidido hasta ahora. El proyecto se construirá **poco a poco**.

**Reglas para esta fase:**

1. **No escribir código de aplicación todavía.** No crear proyectos .NET, tablas SQL, migraciones ni endpoints hasta que se apruebe la arquitectura técnica y el plan de calidad (etapa 8 de la metodología combinada).
2. **No inventar decisiones.** Si algo no está resuelto en este documento, queda marcado como *Pendiente* y se pregunta al responsable del proyecto (Jeanpier).
3. **No reinterpretar las decisiones aceptadas.** Son obligatorias para todo diseño posterior.
4. **Lo permitido ahora:** crear la estructura de carpetas de documentación, guardar este documento en el repositorio y redactar los documentos que se soliciten, uno por vez.
5. Ante la duda entre hacer más o hacer menos, hacer menos y consultar.

### 0.2 Control de versiones: todo se hace por commits en GitHub

El repositorio de GitHub **ya existe**. Todo el trabajo (documentos, diagramas y, más adelante, código) debe quedar registrado mediante commits para no perder versiones.

**Credenciales:**

- El token de acceso será entregado al agente **fuera de este documento** (variable de entorno `GITHUB_TOKEN` o gestor de credenciales de Git).
- **Nunca** se escribe el token en archivos del repositorio, en documentos, en mensajes de commit, en la URL del remoto guardada en el historial ni en registros de salida.
- Si el token aparece accidentalmente en algún archivo o commit, se detiene el trabajo y se avisa de inmediato para revocarlo.
- Se debe incluir un `.gitignore` que excluya `.env`, secretos, respaldos, credenciales y datos reales de clientes.

**Reglas de trabajo con Git:**

| Regla | Detalle |
|---|---|
| Commits pequeños | Un commit por cambio lógico: un documento, una ficha, un diagrama, una decisión. |
| Mensajes claros | Formato sugerido: `docs: agrega ficha de proceso registro de animal`. Prefijos: `docs`, `feat`, `fix`, `refactor`, `test`, `chore`. |
| Idioma | Mensajes de commit en español, en tiempo presente e imperativo corto. |
| Historial inmutable | **Prohibido** `git push --force`, reescribir historial o borrar ramas sin autorización. |
| Antes de cambiar | Hacer `git pull` y revisar el estado antes de modificar archivos. |
| Después de cambiar | Commit + push al terminar cada unidad de trabajo. No acumular cambios sin subir. |
| Ramas | Mientras sea solo documentación, se puede trabajar sobre `main`. Cuando haya código, se usarán ramas `feature/...` y `fix/...` con integración controlada. |
| Versiones | Cada hito aprobado se marca con una etiqueta (tag), por ejemplo `docs-v0.1`. |
| Cambios en decisiones | Una decisión aceptada no se edita en silencio: se agrega un ADR nuevo que la reemplace o modifique y se indica en el commit. |

---

## 1. Qué es SIRACOM

**SIRACOM** es un producto de software integral para gestión institucional, campañas de bienestar animal y servicios veterinarios. Se desarrolla **desde cero** con **C# y .NET**.

Puede comercializarse a municipalidades, instituciones públicas, clínicas veterinarias y empresas privadas que gestionan servicios veterinarios.

**Principio rector:** un producto común, con módulos configurables, que se instala de forma independiente para cada cliente. No son dos sistemas (público y privado): es un solo producto con núcleo común.

### 1.1 Alcance funcional (integral, sin MVP)

El sistema se **diseña de forma integral**. La implementación se organizará por dependencias técnicas y funcionales, pero eso **no reduce el alcance** previsto. No se debe proponer un MVP que deje funcionalidades esenciales fuera del diseño.

**A. Plataforma y administración local**
Usuarios, autenticación, roles y permisos; configuración institucional; auditoría de acciones relevantes; parámetros del sistema; archivos y evidencias; respaldos, recuperación, monitoreo y mantenimiento; seguridad y protección de datos.

**B. Animales y responsables**
Registro e identificación de animales (especie, raza, sexo, edad estimada, características, estado); responsables y datos de contacto necesarios; identificadores como microchip cuando corresponda; historial de cambios y trazabilidad; prevención de duplicados con criterios verificables.
*Regla:* nunca fusionar animales automáticamente por coincidencia de nombre o por el nombre de sus responsables.

**C. Gestión veterinaria**
Citas y agenda; consultas y atenciones; historia clínica; diagnósticos, tratamientos y prescripciones; vacunaciones, desparasitaciones y procedimientos; seguimientos y controles; adjuntos y evidencias; historial y reportes.

**D. Campañas y gestión institucional**
Planificación y organización de campañas; registro de participantes y animales atendidos; programación, ejecución y cierre; control de personal y actividades; resultados y evidencias; metas y estadísticas; esterilizaciones, adopciones y otros procesos institucionales; reportes operativos y de gestión.
Las campañas se **integran** con la gestión veterinaria y el inventario: una atención en campaña puede generar el registro clínico y los movimientos de inventario, sin duplicar registros.

**E. Inventario y abastecimiento**
Productos e insumos; existencias y movimientos; lotes, vencimientos y trazabilidad; entradas, salidas, ajustes y transferencias; consumo asociado a atenciones y campañas; proveedores y compras cuando apliquen; alertas de stock y vencimiento; historial con responsables.
Una institución pública que no cobra igualmente necesita controlar inventario, compras, consumos y costos.

**F. Funcionalidades comerciales configurables**
Servicios, tarifas y listas de precios; ventas y cobros; caja; comprobantes e integraciones de facturación cuando correspondan; compras, costos y reportes comerciales; pagos electrónicos según acuerdos.
**No deben acoplarse al núcleo clínico.** Una municipalidad y una clínica privada comparten los procesos veterinarios esenciales sin estar obligadas a usar las mismas funciones comerciales.

### 1.2 Diferencias típicas entre tipos de cliente

- **Municipalidad:** campañas públicas, control institucional, esterilizaciones, adopciones, reportes de gestión.
- **Clínica privada:** citas, atención, tarifas, ventas, cobros y caja.
- Ambas comparten las funciones veterinarias. La configuración de una instalación nunca da acceso a datos de otra.

---

## 2. Estados de las decisiones

- **Aceptada:** confirmada y obligatoria.
- **Diferida:** decidido no hacerla ahora; se conserva como referencia futura.
- **Propuesta:** recomendación pendiente de aprobación.
- **Pendiente:** requiere análisis antes de cerrarse.

---

## 3. Decisiones

### ADR-001 — Instalaciones independientes por cliente
**Estado:** Aceptada.

**Decisión.** Cada cliente tiene su propia instalación de SIRACOM, con su propia base de datos y recursos. **No** habrá plataforma multiempresa centralizada, base de datos compartida entre clientes ni administrador global.

Cada instalación es independiente en: datos y registros; usuarios, credenciales, roles y permisos; archivos, evidencias y documentos; configuración y módulos habilitados; auditoría y registros operativos; respaldos y recuperación; integraciones y credenciales externas.

Ninguna instalación puede consultar, modificar ni eliminar datos de otra. La administración de cada instalación corresponde a usuarios autorizados de esa organización. El desarrollador **no** tiene acceso permanente e implícito a datos de producción; el soporte excepcional requiere autorización, controles, trazabilidad y acuerdo contractual.

**Consecuencias.** Cada instalación necesita despliegue, monitoreo, respaldo y mantenimiento; las migraciones se ejecutan por instalación; hace falta automatización para que el esfuerzo no crezca proporcionalmente al número de clientes; las integraciones se configuran por instalación.

**Prohibido:** asumir que compartir el código implica compartir base de datos, credenciales o administración. **No** volver a proponer una plataforma centralizada multiempresa como arquitectura predeterminada. Si se cree necesaria una excepción, se debe explicar el caso de uso, los riesgos y por qué no puede resolverse con instalaciones independientes.

**Identidad de animales entre instalaciones.** No existe identidad global operativa ni consulta automática entre clientes. Una transferencia futura de información sería un procedimiento explícito con autorización, validación, minimización de datos, trazabilidad y compatibilidad. Dentro de una instalación sí se mantiene trazabilidad del animal y se evitan duplicados.

---

### ADR-002 — Dos modalidades de alojamiento
**Estado:** Aceptada.

**Decisión.** SIRACOM admite ambas modalidades con la misma base de código:

1. **Alojamiento administrado por el cliente.**
2. **Alojamiento administrado por el proveedor de SIRACOM**, con una instalación independiente por cliente.

**Consecuencias.** La aplicación no depende de rutas, IP, servicios o recursos del entorno del desarrollador. Se requiere: configuración por variables de entorno y archivos apropiados; gestión segura de secretos; despliegues reproducibles; migraciones versionadas; respaldos y restauración; registro de errores y monitoreo; documentación de instalación, actualización y recuperación; separación entre configuración del producto y configuración del cliente. No se presupone un proveedor de nube único ni conexión permanente a una infraestructura central de SIRACOM.

**Pendiente dentro de este ADR:** responsable de respaldos, actualizaciones y recuperación en cada modalidad; requisitos mínimos de hardware y software; política de disponibilidad y recuperación ante incidentes; responsabilidades contractuales sobre datos; si algún cliente debe operar totalmente en red local.

---

### ADR-003 — Operación sin conexión (offline) y móvil
**Estado:** **Diferida.**

**Decisión vigente.** Por ahora **SIRACOM es una aplicación web conectada a Internet**. No se desarrolla modo offline ni aplicación móvil nativa en esta etapa. La interfaz web será *responsive* para poder usarse desde un celular con conexión.

**Escalabilidad futura.** El sistema debe poder crecer más adelante hacia una aplicación móvil y operación offline para campañas. Esa capacidad **no es requisito del diseño actual**, pero no se debe cerrar la puerta. Para ello se establecen estas **reglas de diseño obligatorias (costo bajo ahora, ahorro alto después):**

1. **API primero.** Toda la lógica de negocio vive en el backend. La interfaz web solo consume la API y no contiene reglas de negocio propias.
2. **Identificadores generados por el cliente (GUID/UUID)** en registros que a futuro podrían crearse en campo: atenciones, vacunaciones, consumos de inventario y similares.
3. **Operaciones idempotentes.** Reenviar la misma operación no debe crear una segunda atención, un segundo consumo de inventario ni una segunda aplicación de vacuna. Es necesario también en la web conectada, porque las redes fallan.
4. **Fecha de ocurrencia separada de la fecha de registro.** Un campo para cuándo ocurrió el hecho y otro para cuándo se guardó en el sistema.
5. **API versionada**, para que una futura app móvil conviva con distintas versiones del servidor.
6. **Interfaz web responsive.**

**Referencia para el futuro (no implementar ahora).** Si se retoma el offline, habrá que decidir: qué operaciones funcionan sin conexión; si se pueden crear animales nuevos o solo seleccionar descargados; PWA/IndexedDB frente a aplicación instalada (la PWA tiene límites de almacenamiento, sobre todo en iOS); cola de operaciones pendientes; resolución de conflictos sin sobrescrituras silenciosas; validación del servidor y flujo de **revisión humana** cuando un registro hecho en campo sea rechazado; seguridad del dispositivo; minimización y eliminación de datos locales; trazabilidad de dispositivo, momento y resultado de sincronización.

**Advertencia.** Si las campañas en zonas sin señal resultan ser un uso principal, el offline pasará a ser un proyecto propio. Las seis reglas anteriores reducen el costo, no lo eliminan.

---

### ADR-004 — Licencia de uso o entrega del código fuente
**Estado:** Aceptada como estrategia comercial flexible; condiciones específicas pendientes.

**Decisión.** Se contemplan dos modalidades contractuales: (1) licencia de uso del producto y (2) entrega del código fuente bajo condiciones pactadas. Los derechos de modificación, redistribución, explotación comercial, mantenimiento y actualización se definen por contrato. **No se asume** que todos los clientes reciban el código ni que tengan derechos de redistribución.

**Consecuencias.** El producto debe compilarse y desplegarse de forma reproducible. No se incluyen secretos, credenciales de producción ni datos reales en el código distribuido. Las dependencias de terceros se revisan según su licencia. Los módulos comerciales tienen límites técnicos claros. **No** se depende de un servidor central de SIRACOM para ejecutar funciones esenciales.

**Concepto a mantener separado:** *configuración funcional* (qué módulos usa una instalación) frente a *derechos contractuales* (qué puede hacer el cliente con el producto). Pendiente decidir si los módulos comerciales se habilitan solo por configuración, por licencia verificable localmente u otro mecanismo, sin comprometer la autonomía de la instalación.

---

### ADR-005 — Arquitectura modular y dependencias
**Estado:** Propuesta pendiente de aprobación formal.

**Propuesta.** Monolito modular en C# y .NET (ASP.NET Core), con PostgreSQL, y límites explícitos entre dominios. **Sin microservicios** salvo justificación técnica y operativa concreta. No crear proyectos o capas artificiales solo para aparentar modularidad.

**Regla de dependencias.**
- El núcleo de animales funciona sin depender de facturación.
- La gestión clínica no requiere caja ni módulo de ventas.
- Campañas, atención veterinaria e inventario se coordinan mediante contratos de aplicación definidos.
- **Lo comercial depende de lo clínico, nunca al revés.** Los módulos comerciales consumen servicios, atenciones e inventario mediante interfaces explícitas.
- Las funciones comerciales no imponen reglas a procesos clínicos que no las necesitan.
- La auditoría registra operaciones relevantes sin servir para eludir las reglas de cada módulo.

**Sugerencia para hacerla verificable:** pruebas de arquitectura automatizadas que fallen si se rompe el sentido de las dependencias.

**Pendiente:** estructura definitiva de proyectos; límites de módulos; estrategia de transacciones entre dominios; política de eventos internos; herramientas de autenticación, autorización y pruebas.

---

### ADR-006 — Fuente de verdad para procesos y diagramas
**Estado:** Propuesta pendiente de aprobación formal.

**Propuesta.** Las **fichas de proceso en Markdown** son la referencia principal. Los diagramas se generan a partir de ellas (con apoyo de IA) y se revisan contra ellas; no reemplazan reglas de negocio ni la ficha. Los diagramas generados por IA son propuestas: no se aceptan si omiten permisos, excepciones, estados o reglas.

**Contenido de una ficha:** identificador y objetivo; actores y permisos; disparador y precondiciones; flujo principal; flujos alternativos y excepciones; reglas de negocio; estados y transiciones; datos de entrada y resultados; efectos sobre otros módulos; auditoría requerida; criterios de aceptación y casos de prueba (normal, límite, error, acceso no autorizado); dependencias.

**Herramientas (pocas):**
- **GitHub + Markdown:** documentación oficial, decisiones y fichas.
- **Mermaid:** primera opción para diagramas versionables como texto en Git.
- **diagrams.net:** diagramas visuales de procesos y arquitectura; permite dibujar BPMN.
- **BPMN:** notación (no herramienta), solo donde aporte valor por eventos, responsables y excepciones.
- **dbdiagram.io:** modelo entidad-relación, **solo** cuando procesos y reglas estén definidos.

---

### ADR-007 — Docker para empaquetado y despliegue
**Estado:** Aceptada.

**Decisión.** SIRACOM se empaquetará y desplegará con **Docker**. El contrato de despliegue será una **imagen de contenedor de la aplicación más un archivo Docker Compose de referencia** que incluya PostgreSQL, volúmenes y configuración por variables de entorno.

**Obligaciones:**
- Datos de la base y archivos subidos viven en **volúmenes persistentes** con respaldo probado. Un contenedor sin volumen pierde sus datos al recrearse.
- Secretos fuera de la imagen y fuera del repositorio.
- Migraciones automatizadas con respaldo previo.
- Documentación de instalación, actualización y restauración.

**Cuidado.** Algunos clientes públicos pueden tener servidores Windows o personal sin experiencia en Docker. La arquitectura debe permitir una alternativa de instalación según el entorno, sin reescribir la aplicación.

---

### ADR-008 — Gestión del trabajo con Git y GitHub
**Estado:** Aceptada.

**Decisión.** Todo el trabajo se registra en el repositorio de GitHub existente mediante commits pequeños, con las reglas de la sección 0.2. El avance es incremental: primero contexto, luego documentos uno por uno, luego diseño, y recién después código. El token se maneja fuera del repositorio.

---

### ADR-009 — Ciclo de vida y metodología combinada del proyecto
**Estado:** Aceptada.

**Decisión.** Adoptar un ciclo de vida combinado de diez pasos que unifica el rigor de aislamiento, gobernanza y arquitectura de SIRACOM con el diseño funcional temprano de experiencia de usuario (UI/UX), la validación con usuarios reales y el plan integral de calidad y entrega continua.

**Ciclo combinado:**
1. **Visión, objetivos y alcance integral:** Definición del producto común, módulos y capacidades futuras sin recortes prematuros de MVP.
2. **Actores, responsabilidades, roles y permisos:** Matriz de accesos y responsabilidades institucionales y veterinarias.
3. **Módulos, límites y dependencias:** Acoplamiento unidireccional (lo comercial depende de lo clínico, nunca al revés) y contratos entre dominios.
4. **Procesos, reglas, estados y excepciones, junto con bocetos de interfaz (UI/UX):** Fichas de proceso en Markdown acompañadas de diseño de navegación, pantallas y formularios accesibles para personal municipal y veterinario, reconociendo que la interacción revela reglas y excepciones de negocio antes de tocar la base de datos.
5. **Validación de procesos y bocetos con usuarios reales:** Revisión temprana de flujos y pantallas con personal municipal y de clínicas veterinarias antes de consolidar el modelo de datos.
6. **Aislamiento por instalación, seguridad, datos personales y aspectos legales:** Garantía de base de datos y recursos independientes por cliente, cumplimiento de Ley 29733 (Perú), licencias de software y consideraciones de facturación electrónica.
7. **Modelo de dominio y base de datos:** Modelo conceptual, lógico y físico con persistencia en PostgreSQL, respetando los procesos y reglas validados.
8. **Arquitectura técnica y plan de calidad:** Definición de arquitectura de la aplicación (C# / .NET), contenedor Docker, pruebas automatizadas (aislamiento, seguridad, recuperación), integración continua, versiones, migraciones y respaldos probados.
9. **Implementación por módulos:** Construcción modular e incremental basada en dependencias técnicas, con criterios de aceptación estrictos y ajuste continuo del diseño.
10. **Validación, despliegue y operación:** Verificación integral con el sistema en funcionamiento, automatización de despliegue por instalación independiente y procedimientos de soporte y recuperación.

**Consecuencias.** 
- La arquitectura técnica y el plan de calidad se consolidan en la **Etapa 8** (antes de comenzar la implementación de código de la Etapa 9).
- El diseño de interfaz no se posterga a la fase técnica; se elabora en paralelo a los procesos para alimentar tempranamente el modelo de datos.
- Se mantiene el principio de que los diagramas, bocetos y fichas residen en Git como fuente de verdad en Markdown.

---

## 4. Principios técnicos generales

- Backend en **C# y .NET**; base de datos **PostgreSQL**.
- API preparada para clientes web y, a futuro, móviles.
- Autenticación, autorización y auditoría desde el diseño.
- Migraciones controladas y versionadas.
- Pruebas automatizadas, incluidas las de aislamiento, seguridad y recuperación.
- Despliegue reproducible y documentación operativa.
- **Sin complejidad innecesaria:** bibliotecas, patrones y estructura de proyectos se eligen justificándolos por necesidades reales.
- Prioridad: sistema coherente, seguro, mantenible y comercializable, que se instale y opere de forma independiente para cada cliente.

---

## 5. Metodología de diseño y ciclo combinado (orden obligatorio)

1. Visión, objetivos y alcance funcional integral.
2. Actores, responsabilidades, roles y permisos.
3. Módulos, límites y dependencias entre dominios.
4. Procesos, reglas, estados y excepciones, junto con bocetos de interfaz (UI/UX).
5. Validación de procesos y bocetos con usuarios reales (personal municipal y veterinario).
6. Aislamiento por instalación, seguridad, datos personales (Ley 29733) y aspectos legales.
7. Modelo de dominio y base de datos (conceptual, lógico y físico en PostgreSQL).
8. Arquitectura técnica y plan de calidad (pruebas, CI/CD, versionado, migraciones y respaldos).
9. Implementación por módulos con criterios de aceptación y ajuste de diseño.
10. Validación, despliegue y operación.

**Reglas del ciclo:**
- **No** diseñar la base de datos sin conocer los procesos que debe soportar ni haber validado los bocetos de interacción.
- **No** programar módulos aislados sin definir sus relaciones, responsabilidades y plan de pruebas.
- Las etapas se retroalimentan: el diseño de pantallas revela excepciones que afinan las fichas de proceso antes de congelar el modelo de datos.

### Orden sugerido de procesos a documentar

1. Registro de animal y responsable.
2. Agenda de citas.
3. Atención veterinaria.
4. Inventario veterinario.
5. Campaña institucional.
6. Vacunación y esterilización.
7. Adopciones.
8. Gestión comercial.

Estos procesos están conectados entre sí. No deben existir varios registros paralelos de la misma atención.

---

## 6. Decisiones abiertas

| Decisión pendiente | Por qué importa | Responsable propuesto |
|---|---|---|
| Modelo de alojamiento y operación (quién respalda, actualiza y recupera) | Define responsabilidades, respaldos y soporte. | Producto y arquitectura |
| Política de datos personales y retención (Ley 29733 en Perú) | Define permisos, conservación, eliminación y obligaciones contractuales. Sube de prioridad si el proveedor aloja datos de clientes. | Producto, seguridad y asesoría legal |
| Estrategia de actualizaciones y versionado | Define compatibilidad entre versión de aplicación y de base de datos, y cuántas versiones se soportan a la vez. | Arquitectura y operaciones |
| Licenciamiento de módulos | Define cómo se habilitan funciones y qué derechos recibe cada cliente. | Producto y asesoría contractual |
| Identificación de animales | Define duplicados, identificadores y reglas de trazabilidad. | Analista funcional |
| Facturación electrónica en Perú | Define alcance de integración, credenciales y responsabilidades (SUNAT, proveedores autorizados). | Producto, arquitectura y asesoría tributaria |
| Respaldos y recuperación | Define pérdida de datos admisible y tiempos de recuperación. | Arquitectura y operaciones |
| Seguridad y auditoría (incluye acceso de soporte) | Define controles, registros y acceso temporal autorizado. | Seguridad y arquitectura |
| Idioma y localización | Formatos y normativa; relevante si se vende fuera de Perú. | Producto |
| Alternativa a Docker para clientes sin ese entorno | Define opciones de instalación. | Arquitectura y operaciones |

La asignación de responsables es una propuesta organizativa, no una aprobación.

---

## 7. Despliegue, actualizaciones y soporte

Aunque cada instalación sea independiente, el producto debe poder mantenerse con eficiencia:

- Automatización de instalación y actualización.
- Migraciones versionadas con verificaciones previas y respaldo antes de cambios que modifiquen datos.
- Procedimientos de restauración probados.
- Compatibilidad entre versiones de aplicación y base de datos.
- Política de soporte para versiones antiguas y correcciones de seguridad.
- Registros de diagnóstico que minimicen datos personales.
- Exportación controlada de información técnica para soporte.
- Acceso temporal, autorizado y auditable cuando sea imprescindible intervenir.
- Inventario de versiones desplegadas **sin** recopilar datos operativos de los clientes en un panel central.

Nada de esto debe introducir un administrador global ni acceso implícito a datos de las organizaciones.

---

## 8. Estructura de documentación en el repositorio

```
siracom/
├── docs/
│   ├── 00-decisiones/          ← este documento (ADR)
│   ├── 01-vision-y-alcance/
│   ├── 02-actores-roles-permisos/
│   ├── 03-catalogo-de-modulos/
│   ├── 04-procesos/
│   │   ├── registro-animal/
│   │   ├── agenda-citas/
│   │   ├── atencion-veterinaria/
│   │   ├── inventario/
│   │   └── campanas/
│   ├── 05-modelo-de-dominio/
│   ├── 06-base-de-datos/
│   └── 07-arquitectura-tecnica/
└── src/                        ← vacío hasta aprobar la arquitectura
```

Los diagramas de diagrams.net se guardan junto a sus fichas; los Mermaid se versionan como `.mmd` o dentro del Markdown.

---

## 9. Próximo entregable

La **especificación de visión, alcance funcional y actores de SIRACOM**, junto con el ADR actualizado. Debe distinguir claramente:

1. Funcionalidades comunes.
2. Funcionalidades institucionales.
3. Funcionalidades veterinarias.
4. Funcionalidades comerciales.
5. Funcionalidades transversales de seguridad, auditoría, despliegue y operación.
6. Capacidades futuras diferidas (móvil y offline), claramente separadas del alcance actual.

Después se elaborarán las fichas de procesos, los diagramas y el modelo conceptual. El modelo de datos definitivo se hará cuando procesos y decisiones críticas estén suficientemente definidos.

---

## 10. Primeras acciones del agente al recibir este documento

1. Confirmar que entendió las reglas de la sección 0 (sin responder con código).
2. Verificar acceso al repositorio con el token entregado por canal seguro, sin escribirlo en ningún archivo.
3. Crear la estructura de carpetas de la sección 8 (con archivos `.gitkeep` donde estén vacías).
4. Guardar este documento en `docs/00-decisiones/ADR-SIRACOM.md`.
5. Agregar un `.gitignore` adecuado.
6. Hacer el primer commit (`docs: agrega ADR inicial y estructura de documentación`) y subirlo.
7. Esperar la siguiente instrucción antes de redactar cualquier otro documento.

---

## Criterio final

SIRACOM es un producto común, distribuible a múltiples clientes mediante **instalaciones independientes** y contenedores Docker. Cada instalación opera de forma autónoma, protege sus propios datos y se aloja según el contrato. El diseño funcional es integral; la implementación se organiza por dependencias; hoy es una aplicación web conectada, con una arquitectura que permite crecer hacia móvil y offline sin rehacer el modelo. Las decisiones abiertas permanecen identificadas hasta que exista justificación suficiente para cerrarlas.
