# SIRACOM — Especificación de Visión, Objetivos y Alcance Integral

**Documento:** Especificación Funcional de Visión y Alcance  
**Versión:** 0.1 (Borrador)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y aprobación por Jeanpier)  
**Ubicación en repositorio:** `docs/01-vision-y-alcance/especificacion-vision-alcance.md`  
**Referencia arquitectónica:** ADR-001 a ADR-009 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))

---

## 1. Visión del Producto

### 1.1 Declaración de Visión
**SIRACOM** (Sistema Integral de Registro y Atención para Campañas y Operaciones Mascotas) es una plataforma de software diseñada para transformar la gestión institucional de bienestar animal, el control de zoonosis y la prestación de servicios médico-veterinarios, tanto en el sector público (municipalidades e instituciones gubernamentales) como en el sector privado (clínicas veterinarias y albergues/ONGs).

El sistema sustituye los registros fragmentados en cuadernos físicos, hojas de cálculo aisladas y sistemas dispersos por un ecosistema unificado, auditable y de alta trazabilidad que abarca desde la identificación del animal y su historial clínico hasta la gestión de abastecimiento, campañas masivas en campo y procesos institucionales de adopción y custodia.

### 1.2 Principio Rector y Modelo de Distribución
* **Producto común con núcleo unificado:** SIRACOM no se desarrolla como dos software bifurcados (público y privado). Es una única base de código modular construida con C# y .NET, con persistencia en PostgreSQL y empaquetamiento en contenedores Docker.
* **Instalaciones independientes (ADR-001):** Cada cliente cuenta con su propia instalación autónoma, con base de datos, almacenamiento de evidencias y configuración de usuarios y módulos totalmente independientes. No existe una base de datos multiempresa compartida ni un superadministrador central con acceso a los datos de los clientes.
* **Desacoplamiento comercial-clínico (ADR-005):** El núcleo clínico veterinario y de registro animal es completamente autónomo y no depende de facturación ni caja. Los módulos comerciales consumen información de atenciones e insumos mediante contratos explícitos, pero nunca imponen reglas restrictivas al acto médico o a la campaña pública gratuita.
* **Cliente de referencia prioritario para validación:** Se establece a la **Municipalidad (Área de Zoonosis y Bienestar Animal)** como el arquetipo de referencia para validar el primer borrador, por ser el entorno que cubre el espectro operativo más amplio (campañas masivas, atención en consultorio, albergue temporal, adopciones, control de insumos y padrón de tenencia). Simultáneamente, se valida la compatibilidad con el modelo de **Clínica Veterinaria Privada** mediante la activación de submódulos comerciales (tarifas, cobros, caja).

---

## 2. Objetivos del Sistema

### 2.1 Objetivos Generales
1. **Garantizar la trazabilidad médica integral:** Consolidar una historia clínica única por animal que integre consultas, vacunaciones, desparasitaciones, intervenciones quirúrgicas y tratamientos realizados tanto en sede fija como en campañas públicas.
2. **Optimizar el control de abastecimiento y farmacia:** Administrar el inventario de medicamentos e insumos veterinarios con control estricto de lotes, vencimientos y consumo en tiempo real vinculado a cada atención médica o brigada de campaña.
3. **Transparentar la gestión institucional:** Proporcionar métricas confiables de cobertura de campañas (vacunación antirrábica, esterilizaciones), metas de bienestar animal y custodia responsable de animales rescatados o en albergue.
4. **Preservar la soberanía y seguridad de los datos:** Asegurar el aislamiento físico/lógico de la información de cada entidad, con auditoría exhaustiva de modificaciones y cumplimiento de principios de protección de datos personales.

### 2.2 Objetivos Específicos
* Reducir tiempos de atención en campañas masivas mediante flujos de registro optimizados para digitación ágil.
* Evitar la duplicidad de registros y la pérdida de historiales clínicos sin recurrir a fusiones automáticas arbitrarias.
* Controlar mermas, caducidades y desabastecimiento de insumos biológicos y farmacológicos mediante alertas preventivas.
* Sistematizar el flujo completo de rescate, custodia en albergue y adopción responsable con seguimiento verificable.
* Brindar flexibilidad comercial a clínicas y servicios tarifados mediante esquemas de comprobantes y tarifas desacoplados del flujo clínico.

---

## 3. Alcance Funcional Integral (Vigente en Fase de Diseño)

El alcance se estructura sin recurrir a recortes artificiales (sin MVP que excluya procesos vitales), organizando las capacidades en dominios funcionales:

```
┌────────────────────────────────────────────────────────────────────────┐
│                      SIRACOM - NÚCLEO MODULAR                          │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Plataforma, Parámetros y Auditoría Local                            │
│ 2. Animales y Responsables (Identificación y Trazabilidad)             │
│ 3. Gestión Médica Veterinaria (Citas, Triaje, Consulta, Quirúrgico)     │
│ 4. Campañas Institucionales y Brigadas de Campo                        │
│ 5. Farmacia, Abastecimiento e Inventario (Lotes y Vencimientos)        │
│ 6. Albergue, Rescate, Custodia y Adopciones Responsables               │
│ 7. Gestión Comercial Configurable (Tarifas, Cobros, Caja)             │
└────────────────────────────────────────────────────────────────────────┘
```

### 3.1 Dominio A: Plataforma, Administración Local y Auditoría
* **Gestión de usuarios y credenciales:** Altas, bajas, reseteo de claves y asignación de perfiles de roles dentro de la instalación local.
* **Configuración institucional:** Datos de la entidad (nombre, logotipo, dirección, parámetros de atención, sedes/consultorios locales).
* **Parámetros maestros:** Catálogo configurable de especies (canino, felino como base), razas, tipos de procedimientos, vías de administración de medicamentos, causales de atención.
* **Pistas de auditoría:** Registro inmutable de transacciones relevantes (fecha, hora, usuario, IP, operación realizada, valores anteriores y nuevos).
* **Gestión de almacenamiento:** Repositorio de evidencias (fotografías del animal, actas firmadas, exámenes de laboratorio, recetas escaneadas) vinculado a volúmenes persistentes protegidos.

### 3.2 Dominio B: Animales y Responsables
* **Ficha de identificación animal:** Registro de especie (configurable), raza, sexo, condición reproductiva (fértil, esterilizado), edad cronológica o estimada, peso inicial, señas particulares, color, pelaje y fotografía referencial.
* **Identificadores físicos:** Código de microchip (ISO), tatuaje o placa/código municipal cuando aplique, con validación de unicidad en la instalación.
* **Ficha de responsables / tenedores:** Registro de datos del ciudadano (documento de identidad, nombres, apellidos, teléfono, correo electrónico, dirección domiciliaria, pertenencia a sector/distrito).
* **Trazabilidad y no duplicidad:**
  - Vinculación histórica entre animales y responsables (adopción, cambio de domicilio, transferencia de custodia).
  - Regla estricta de no fusión: jamás consolidar automáticamente dos animales por similitud fonética o por pertenecer al mismo dueño. Las coincidencias sospechosas se alertan para verificación humana del operador.

### 3.3 Dominio C: Gestión Clínica Veterinaria
* **Agenda y citas:** Programación de citas para consultorio fijo (consultas generales, cirugías programadas, controles).
* **Atención y triaje:** Registro de constantes vitales (peso, temperatura corporal, frecuencia cardíaca, condición corporal).
* **Historia clínica cronológica:** Registro estructurado de motivo de consulta, anamnesis, hallazgos de examen clínico, diagnóstico presuntivo/definitivo y plan terapéutico.
* **Prescripción y recetas:** Generación de receta médica veterinaria con especificación de dosis, frecuencia y duración del tratamiento.
* **Procedimientos preventivos:** Registro de vacunaciones (con número de lote y laboratorio) y desparasitaciones internas/externas, calculando automáticamente la fecha de refuerzo sugerida.
* **Procedimientos quirúrgicos:** Registro de actos quirúrgicos (esterilizaciones/castraciones, cirugías menores/mayores), profesional responsable, protocolo anestésico e incidencias post-operatorias.

### 3.4 Dominio D: Campañas Institucionales y Brigadas
* **Planificación y programación:** Definición de campaña (nombre, objetivo, tipo: antirrábica, desparasitación masiva, esterilización, empadronamiento), fecha de inicio, fecha de cierre, ubicación/zona geográfica y metas proyectadas.
* **Asignación de recursos:** Designación de brigadistas/personal asignado y asignación de lotes de biológicos/insumos desde el almacén.
* **Ejecución y registro rápido:** Interfaz optimizada de digitación ágil para registro masivo en campo (animal atendido, responsable, insumo aplicado) asegurando sincronización con el historial clínico del animal.
* **Cierre y balance:** Liquidación de campaña, balance de insumos utilizados vs. sobrantes retornados al almacén, y consolidación estadística de animales intervenidos.

### 3.5 Dominio E: Farmacia, Abastecimiento e Inventario
* **Catálogo de insumos y fármacos:** Medicamentos, vacunas, anestésicos, material de curación, alimentos y microchips.
* **Control de existencias multialmacén:** Almacén central, farmacia de consultorio, botiquín de quirófano y stock asignado a brigadas de campaña.
* **Trazabilidad por lote y fecha de expiración:** Ingreso con número de lote, fecha de vencimiento y registro sanitario; salidas bajo criterio FEFO (First Expired, First Out / Primero en vencer, primero en salir).
* **Consumo automático:** Descuento directo del inventario derivado de atenciones clínicas o aplicaciones en campañas.
* **Ajustes y transferencias:** Registro justificado y auditado de mermas, pérdidas, roturas o transferencias entre almacenes/consultorios.
* **Alertas operativas:** Avisos visuales preventivos de stock mínimo crítico y productos próximos a expirar (30/60/90 días).

### 3.6 Dominio F: Albergue, Rescate, Custodia y Adopciones Responsables
* **Ingreso y registro de custodia:** Registro de ingreso de animales en abandono, maltrato o decomiso; causal de ingreso, estado clínico al llegar, fotografías de rescate y asignación de espacio/canil físico.
* **Bitácora de estadía:** Registro periódico de evolución de comportamiento, sociabilización, tratamientos médicos en curso y plan de alimentación.
* **Gestión de adoptantes y postulaciones:** Registro de postulantes a adopción, evaluación socio-habitacional y verificación de idoneidad.
* **Formalización de adopción:** Asignación del animal al adoptante aprobado, generación del Acta de Compromiso de Tenencia Responsable y cambio de estado a "Adoptado".
* **Seguimiento post-adopción:** Cronograma de visitas o llamadas de control post-adopción, registro de evidencias fotográficas del estado del animal y cierre de expediente.

### 3.7 Dominio G: Gestión Comercial Configurable (Para Clínicas o Servicios con Tarifa)
* **Catálogo de servicios y listas de precios:** Configuración de tarifas para consultas, cirugías, exámenes, vacunas o servicios estéticos.
* **Punto de venta y caja:** Generación de órdenes de cobro vinculadas a la atención médica o venta de mostrador, apertura y cierre de caja diaria con arqueo de valores.
* **Emisión de comprobantes internos:** Generación e impresión de notas de venta/recibos de cobro y registro de forma de pago (efectivo, transferencia, POS).
* **Exportación contable:** Reportes de ventas diarias, ingresos por tipo de servicio y cierres de caja en formatos estándar (Excel/CSV) para conciliación contable.

---

## 4. Capacidades Futuras Diferidas (ADR-003)

Siguiendo las directrices del ADR-003, las siguientes capacidades **no forman parte de la implementación vigente**, pero su diseño arquitectónico queda protegido para garantizar una evolución sin traumatismos:

| Capacidad Diferida | Descripción | Salvaguarda Arquitectónica Aplicada Hoy |
|---|---|---|
| **Portal Ciudadano Autoservicio** | Interfaz web para que los tenedores de mascotas consulten carnés digitales de vacunas, agenden citas en línea o postulen a adopciones. | **API Primero:** Toda la lógica reside en el backend; el personal interno usa la misma API que utilizará el portal futuro. |
| **Operación Offline en Campaña** | Uso del sistema en zonas rurales o de campaña sin conectividad a Internet con sincronización diferida. | **GUID en cliente e Idempotencia:** Identificadores generados por el cliente en eventos clínicos e inventario; operaciones idempotentes; fecha de ocurrencia separada de fecha de registro. |
| **Aplicación Móvil Nativa** | App móvil para brigadistas y personal de campo en Android/iOS. | **API Versionada y Web Responsive:** Interfaz web responsive usable en celulares y API preparada para múltiples clientes. |

---

## 5. Criterio Provisional de Entidades Externas y Reportes

A la fecha de este borrador, los requerimientos de articulación externa no están formalmente confirmados, por lo que se adopta un **enfoque provisional y desacoplado**:

1. **Padrón Municipal / Registro Nacional de Identificación Animal:**
   - *Criterio provisional:* El padrón opera de forma **estrictamente interna por instalación**.
   - *Consideración:* La base de datos mantendrá los campos estándar (código de microchip, datos del responsable, código municipal) listos para exportaciones estructuradas en caso de requerirse reportes a registros distritales o nacionales.
2. **Facturación Electrónica (SUNAT / PSE / OSE en Perú):**
   - *Criterio provisional:* Registro de comprobantes internos con exportación de reportes de cobro.
   - *Consideración:* Se diseña una interfaz de abstracción comercial para que, si un cliente lo requiere, se conecte un proveedor de facturación electrónica sin tocar la lógica clínica.
3. **Reportes Sanitarios Oficiales (Salud Pública / Zoonosis):**
   - *Criterio provisional:* Consolidación de reportes operativos internos (campañas antirrábicas, reportes de mordeduras y atenciones veterinarias) exportables a formatos estándar (Excel/PDF) para entrega manual a las autoridades correspondientes.

---

## 6. Sección de Pendientes y Decisiones Abiertas

Las siguientes materias quedan formalmente marcadas como **Pendientes** y no se asumirán resueltas hasta su validación explícita:

| Identificador | Asunto Pendiente | Impacto en el Diseño | Responsable Propuesto |
|---|---|---|---|
| **PEND-01** | Confirmación formal del cliente prioritario para el piloto | Define el orden de refinamiento fino de los flujos de pantalla. | Jeanpier / Producto |
| **PEND-02** | Obligatoriedad de formatos oficiales de salud pública (MINSA/DIRIS) | Determinará si se requieren plantillas de exportación con campos específicos normados. | Analista Funcional / Coordinación Municipal |
| **PEND-03** | Modalidad de integración de facturación electrónica | Definir si se requerirá conector directo a API de PSE o si la exportación contable satisface la necesidad inicial. | Arquitectura Técnica / Asesoría Contable |
| **PEND-04** | Especies secundarias y reglas clínicas particulares | Determinar qué parámetros clínicos específicos requerirán especies no convencionales cuando se habiliten en el catálogo. | Médico Veterinario Asesor |
