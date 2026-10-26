# SIRACOM — Especificación de Visión, Objetivos y Alcance Integral

**Documento:** Especificación Funcional de Visión y Alcance  
**Versión:** 0.1 (Borrador corregido)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y aprobación por Jeanpier)  
**Ubicación en repositorio:** `docs/01-vision-y-alcance/especificacion-vision-alcance.md`  
**Referencia arquitectónica:** ADR-001 a ADR-009 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))

---

## 1. Visión del Producto

### 1.1 Declaración de Visión
**SIRACOM** (Sistema Integral de Registro y Atención para Campañas y Operaciones Mascotas) es una plataforma de software diseñada para transformar la gestión institucional de bienestar animal, el control de zoonosis y la prestación de servicios médico-veterinarios, tanto en el sector público (municipalidades e instituciones gubernamentales) como en el sector privado (clínicas veterinarias y albergues/ONGs).

El sistema sustituye los registros fragmentados en cuadernos físicos, hojas de cálculo aisladas y sistemas dispersos por un ecosistema unificado, auditable y de alta trazabilidad que abarca desde la identificación del animal y su historial clínico hasta la gestión de abastecimiento, campañas masivas en campo y procesos institucionales de custodia en albergue y adopción.

### 1.2 Principio Rector y Modelo de Distribución
* **Producto común con núcleo unificado:** SIRACOM no se desarrolla como dos software bifurcados (público y privado). Es una única base de código modular construida con C# y .NET, con persistencia en PostgreSQL y empaquetamiento en contenedores Docker.
* **Instalaciones independientes (ADR-001):** Cada cliente cuenta con su propia instalación autónoma, con base de datos, almacenamiento de evidencias y configuración de usuarios y módulos totalmente independientes. No existe una base de datos multiempresa compartida ni un superadministrador central con acceso implícito o permanente a los datos de los clientes.
* **Desacoplamiento comercial-clínico (ADR-005):** El núcleo clínico veterinario y de registro animal es completamente autónomo y no depende de facturación, tarifas ni caja. La desactivación del módulo comercial, o la presencia de fallas operativas en la caja o en la emisión de comprobantes, **bajo ninguna circunstancia bloquea ni impide** registrar consultas médicas, tratamientos, diagnósticos, vacunaciones, desparasitaciones, cirugías ni campañas institucionales. Los módulos comerciales consumen información de atenciones e insumos mediante contratos explícitos, pero jamás imponen restricciones al acto médico o a la campaña pública gratuita.
* **Cliente de referencia para validación (Hipótesis de trabajo):** Se adopta provisionalmente a la **Municipalidad (Área de Zoonosis y Bienestar Animal)** como **hipótesis de trabajo** para contrastar el primer borrador, debido a que su operatoria cubre el espectro funcional más extenso (campañas masivas, consultorio, albergue temporal, adopciones, control de insumos y padrón de tenencia). Esta adopción es una referencia de diseño preliminar y no constituye un compromiso contractual ni funcional cerrado mientras la decisión **PEND-01** permanezca abierta. Se mantiene en todo momento la compatibilidad plena con el modelo de **Clínica Veterinaria Privada** mediante la activación/desactivación modular de submódulos comerciales (tarifas, cobros, caja).

---

## 2. Objetivos del Sistema

### 2.1 Objetivos Generales
1. **Garantizar la trazabilidad médica integral:** Consolidar una historia clínica única por animal que integre consultas, vacunaciones, desparasitaciones, intervenciones quirúrgicas y tratamientos realizados tanto en sede fija como en campañas públicas.
2. **Optimizar el control de abastecimiento y farmacia:** Administrar el inventario de medicamentos e insumos veterinarios con control estricto de lotes, vencimientos y consumo en tiempo real vinculado a cada atención médica o brigada de campaña.
3. **Transparentar la gestión institucional:** Proporcionar métricas confiables de cobertura de campañas (vacunación antirrábica, esterilizaciones), metas de bienestar animal y custodia responsable de animales alojados o rescatados.
4. **Preservar la soberanía y seguridad de los datos:** Asegurar el aislamiento físico y lógico de la información de cada entidad, con auditoría exhaustiva de modificaciones y estricto respeto a la normativa de protección de datos personales.

### 2.2 Objetivos Específicos
* Reducir tiempos de registro en campañas masivas mediante flujos de digitación ágil diseñados para personal de campo.
* Evitar la duplicidad de registros y la pérdida de historiales clínicos sin recurrir a fusiones automáticas arbitrarias.
* Controlar mermas, caducidades y desabastecimiento de insumos biológicos y farmacológicos mediante alertas preventivas basadas en FEFO (First Expired, First Out).
* Sistematizar el flujo completo de rescate, custodia física en albergue y adopción responsable con seguimiento verificable.
* Brindar flexibilidad comercial a clínicas y servicios tarifados mediante esquemas de comprobantes y tarifas desacoplados del flujo clínico.

---

## 3. Alcance Funcional Integral y Correspondencia con el ADR

El alcance se estructura sin recortes artificiales (sin un MVP restrictivo que mutile procesos esenciales), organizando las capacidades en 7 dominios operativos que corresponden directamente con los dominios funcionales establecidos en la Sección 1.1 del [ADR-SIRACOM](../00-decisiones/ADR-SIRACOM.md):

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        CORRESPONDENCIA DE DOMINIOS SIRACOM                             │
├──────────────────────────────────────┬─────────────────────────────────────────────────┤
│ Dominio en este Alcance              │ Dominio de Referencia en ADR-SIRACOM            │
├──────────────────────────────────────┼─────────────────────────────────────────────────┤
│ 1. Plataforma y Administración Local │ Dominio A: Plataforma y administración local    │
│ 2. Animales y Responsables           │ Dominio B: Animales y responsables              │
│ 3. Gestión Médica Veterinaria        │ Dominio C: Gestión veterinaria                  │
│ 4. Campañas Institucionales          │ Dominio D: Campañas y gestión institucional (1) │
│ 5. Farmacia e Inventario             │ Dominio E: Inventario y abastecimiento          │
│ 6. Albergue y Adopciones             │ Dominio D: Campañas y gestión institucional (2) │
│ 7. Gestión Comercial Configurable    │ Dominio F: Funcionalidades comerciales          │
└──────────────────────────────────────┴─────────────────────────────────────────────────┘
```
*Justificación de la división:* El Dominio D del ADR se desagrega en dos dominios operativos (4. Campañas y 6. Albergue/Adopciones) para reflejar con precisión técnica la diferencia entre la logística masiva, itinerante y de corta duración propia de las brigadas de campo, frente a la administración continua, física y legal propia del alojamiento en caniles, evaluación de adoptantes y seguimiento post-adopción.

### 3.1 Dominio 1 (ADR Dominio A): Plataforma, Administración Local y Auditoría
* **Gestión técnica de usuarios:** Altas, bajas, reseteo de claves y asignación de roles predefinidos dentro de la instalación local.
* **Configuración institucional:** Datos de la entidad (nombre, logotipo, dirección, sedes/consultorios locales).
* **Parámetros maestros:** Catálogo configurable de especies (canino y felino como base inicial), razas, tipos de procedimientos, vías de administración de medicamentos y causales de atención.
* **Pistas de auditoría:** Registro inmutable de transacciones relevantes (fecha, hora, usuario, IP, operación realizada, valores anteriores y nuevos).
* **Gestión de almacenamiento:** Repositorio de evidencias (fotografías del animal, actas firmadas, recetas escaneadas) vinculado a volúmenes persistentes protegidos.

### 3.2 Dominio 2 (ADR Dominio B): Animales y Responsables (Identificación y Trazabilidad)
* **Ficha de identificación animal:** Registro de especie (configurable), raza, sexo, condición reproductiva (entero, esterilizado), edad cronológica o estimada, peso inicial, señas particulares, color, pelaje y fotografía referencial.
* **Identificadores físicos:** Código de microchip (ISO), tatuaje o placa/código municipal cuando aplique, con validación de unicidad en la instalación.
* **Ficha de responsables / tenedores:** Registro de datos del ciudadano (documento de identidad, nombres, apellidos, teléfono, correo electrónico, dirección domiciliaria, sector/distrito).
* **Trazabilidad y no duplicidad:**
  - Vinculación histórica entre animales y responsables (adopción, cambio de tenedor, actualización de domicilio).
  - Regla estricta de no fusión: jamás consolidar automáticamente dos animales por coincidencia fonética de nombre o por pertenecer al mismo responsable. Las coincidencias sospechosas se alertan para verificación humana del operador.
* **Regla de anulación:** Las anulaciones o desactivaciones de registros de animales o de responsables **exigen motivo justificado y conservan historial completo**. No existe borrado físico que elimine expedientes del historial de la base de datos.

### 3.3 Dominio 3 (ADR Dominio C): Gestión Médica Veterinaria
* **Agenda y citas:** Programación de citas para consultorio fijo (consultas generales, cirugías programadas, controles).
* **Atención y triaje:** Registro de constantes vitales (peso, temperatura, frecuencia cardíaca, condición corporal).
* **Historia clínica cronológica:** Registro estructurado de motivo de consulta, anamnesis, examen clínico, diagnóstico presuntivo/definitivo y plan terapéutico.
* **Identificación del profesional actuante:** Todo acto clínico, diagnóstico, cirugía o prescripción registra de manera obligatoria la identidad del Médico Veterinario responsable, incluyendo su número de colegiatura profesional.
* **Inmutabilidad de registros cerrados:** Una vez cerrada y firmada la atención médica veterinaria, el registro **no admite edición ni sobrescritura**. Cualquier corrección o aclaración posterior se efectúa exclusivamente mediante notas rectificatorias o adendas médicas fechadas y auditadas que preservan el texto original.
* **Prescripción y recetas:** Generación de receta médica veterinaria con especificación de fármacos, dosis, posología y duración.
* **Procedimientos preventivos:** Registro de vacunaciones (con laboratorio y número de lote) y desparasitaciones, calculando fechas sugeridas de revacunación.
* **Procedimientos quirúrgicos:** Registro de actos quirúrgicos (esterilizaciones/castraciones, cirugías generales), protocolo anestésico, cirujano actuante e incidencias post-operatorias.

### 3.4 Dominio 4 (ADR Dominio D.1): Campañas Institucionales y Brigadas de Campo
* **Planificación y programación:** Configuración de campaña (nombre, objetivo, tipo: antirrábica, esterilización masiva, empadronamiento), fecha de inicio y término, zona geográfica y metas estimadas.
* **Asignación de recursos:** Designación de personal/brigadistas y asignación de lotes de biológicos/insumos despachados desde el almacén.
* **Ejecución y registro rápido:** Interfaz optimizada para digitación ágil en campo (animal intervenido, responsable, biológico aplicado), sincronizando de inmediato con el historial clínico del animal.
* **Cierre y balance:** Liquidación de campaña, balance de insumos utilizados frente a sobrantes devueltos al almacén y consolidación estadística.

### 3.5 Dominio 5 (ADR Dominio E): Farmacia, Abastecimiento e Inventario
* **Catálogo de insumos y fármacos:** Medicamentos, vacunas, anestésicos, material de curación, alimentos y microchips.
* **Control multialmacén:** Almacén central, botiquín de consultorio y stock temporal en custodia de brigadas de campaña.
* **Trazabilidad por lote y vencimiento:** Ingreso con número de lote y fecha de vencimiento; salidas y dispensación bajo principio FEFO (primero en vencer, primero en salir).
* **Consumo automático:** Descuento directo de stock derivado de atenciones clínicas o aplicaciones en campañas.
* **Ajustes y transferencias:** Registro formal de mermas, pérdidas o transferencias entre depósitos con justificación obligatoria.
* **Alertas preventivas:** Notificaciones visuales de stock mínimo crítico y productos próximos a caducar (30/60/90 días).

### 3.6 Dominio 6 (ADR Dominio D.2): Albergue, Custodia Temporal y Adopciones Responsables
Este dominio distingue formalmente dos procesos con dinámicas y requerimientos jurídicos diferentes:

* **3.6.1 Custodia y estancia en albergue (Alojamiento físico):**
  - Registro de ingreso de animales rescatados, en abandono, maltrato o decomiso institucional; causa de ingreso y estado de salud inicial.
  - Asignación y control de ocupación física de espacios (caniles, gateras, áreas de cuarentena).
  - Bitácora diaria de estancia: registro no médico de alimentación, socialización, paseos y evolución de comportamiento.
  - Articulación con el Dominio Clínico para tratamientos médicos y alta sanitaria de salida.

* **3.6.2 Evaluación y formalización de adopción (Gestión de adoptantes):**
  - Registro de postulantes a adopción y evaluación socio-habitacional de idoneidad.
  - Formalización del proceso mediante emisión del Acta de Compromiso de Tenencia Responsable.
  - Asignación del animal al adoptante aprobado y actualización de estado a "Adoptado" vinculando la ficha histórica del nuevo responsable.
  - Cronograma y registro de visitas o llamadas de seguimiento post-adopción con evidencias fotográficas.

### 3.7 Dominio 7 (ADR Dominio F): Gestión Comercial Configurable
* **Catálogo de tarifas y servicios:** Precios configurables para consultas, cirugías, exámenes o medicamentos en establecimientos con cobro.
* **Punto de venta y caja:** Generación de órdenes de cobro vinculadas a la atención médica o venta de mostrador; apertura, cierre y arqueo diario de caja.
* **Emisión de comprobantes internos:** Emisión de notas de venta/recibos internos con registro del medio de pago (efectivo, tarjeta, transferencia).
* **Exportación contable:** Reportes de ventas e ingresos por tipo de servicio en formatos estándar (Excel/CSV) para conciliación contable.
* **Principio de independencia estricta:** Este módulo es 100% optativo. Su desactivación en una municipalidad o la caída del sistema de caja en una clínica jamás interrumpe la prestación ni el registro de los actos veterinarios.

---

## 4. Capacidades Futuras Diferidas (ADR-003)

Siguiendo las directrices del ADR-003, las siguientes capacidades **no forman parte del diseño de implementación vigente**, pero se respetan principios arquitectónicos preventivos para habilitarlas a futuro sin reescribir el modelo:

| Capacidad Diferida | Descripción | Principio Preventivo Aplicado Hoy |
|---|---|---|
| **Portal Ciudadano Autoservicio** | Interfaz web para que los dueños de mascotas consulten carnés digitales de vacunas, agenden citas en línea o postulen a adopciones. | **API Primero:** La lógica de negocio reside en el backend; el cliente web actual consume los mismos servicios que utilizará el futuro portal. |
| **Operación Offline en Campaña** | Uso de la aplicación en campo sin señal de Internet y con sincronización diferida. | **GUID e Idempotencia:** Identificadores únicos generados en cliente para eventos clínicos y de inventario; operaciones idempotentes; fecha de ocurrencia separada de fecha de registro. |
| **Aplicación Móvil Nativa** | Aplicación móvil dedicada para personal de campo o brigadistas. | **API Versionada y Web Responsive:** La interfaz web actual es adaptable a pantallas móviles y la API se diseña con control de versiones. |

*Nota:* No se diseñan endpoints, tablas de base de datos ni componentes de software para estas capacidades en la fase actual.

---

## 5. Criterio Provisional de Entidades Externas y Reportes

A la fecha de este borrador, los requerimientos de interacción externa no están confirmados normativamente, por lo que se adopta un **enfoque desacoplado y provisional**:

1. **Padrón Municipal / Registro de Identificación Animal:**
   - *Criterio provisional:* El registro de animales y microchips opera de forma **estrictamente interna por instalación**.
   - *Consideración:* Los campos estándar (código de microchip, datos del responsable, señas del animal) se almacenan listos para exportaciones estructuradas si en el futuro se requiere notificar a un registro distrital o nacional.
2. **Facturación Electrónica (SUNAT / PSE / OSE en Perú):**
   - *Criterio provisional:* Registro de comprobantes internos con exportación de reportes de cobro.
   - *Consideración:* Se contempla una abstracción en la capa comercial para que la integración con un proveedor de facturación electrónica sea un conector configurable opcional sin impacto en el núcleo clínico.
3. **Reportes Sanitarios Oficiales (Salud Pública / Zoonosis):**
   - *Criterio provisional:* Reportes estadísticos internos (vacunación antirrábica, accidentes por mordedura y esterilizaciones) exportables a formatos estándar (Excel/PDF) para entrega manual o tramitación documental ante autoridades de salud.

---

## 6. Sección de Pendientes con Condiciones de Cierre Explícitas

Las siguientes materias quedan formalmente registradas como **Pendientes**, incorporando para cada una su impacto y la condición concreta requerida para su resolución:

| Identificador | Asunto Pendiente | Impacto en el Diseño | Condición de Cierre Explícita | Responsable Propuesto |
|---|---|---|---|---|
| **PEND-01** | Confirmación formal del cliente prioritario para validación | Define la prioridad en la elaboración de los bocetos de pantalla y flujos de usuario (municipalidad vs. clínica privada). | Decisión documentada y comunicada por Jeanpier que ratifique a la Municipalidad o designe a la Clínica Privada como piloto formal. | Jeanpier / Producto |
| **PEND-02** | Obligatoriedad de formatos oficiales de salud pública (MINSA/DIRIS) | Determinará si se requieren plantillas de exportación con campos fijos normados para vigilancia epidemiológica. | Entrega de directivas sanitarias oficiales vigentes o pronunciamiento formal de que basta con reportes estadísticos internos exportables. | Analista Funcional / Enlace Municipal |
| **PEND-03** | Modalidad de integración de facturación electrónica | Definir si se requiere desarrollo de conector directo a API de PSE/OSE/SUNAT o si la exportación contable satisface la operación inicial. | Definición contractual del alcance de cobros con el primer cliente comercial o decisión formal de mantenerlo en exportación. | Arquitectura Técnica / Asesoría Contable |
| **PEND-04** | Especies secundarias y reglas clínicas particulares | Determinar qué parámetros clínicos específicos requerirán especies no convencionales cuando se activen en el catálogo. | Aprobación de la ficha técnica de parámetros clínicos por parte del especialista veterinario para especies distintas a caninos y felinos. | Médico Veterinario Asesor |
| **PEND-05** | Política de protección de datos personales y retención (Ley 29733) | Define textos de consentimiento informado, plazos de conservación de historias clínicas y datos de responsables, ejercicio de derechos ARCO y protocolos de respaldo por instalación (conforme a ADR-001, ADR-002 y Sección 6 del ADR). | Aprobación de la política institucional de tratamiento de datos personales y cláusulas de confidencialidad por la asesoría legal correspondiente. | Asesoría Legal / Seguridad |
