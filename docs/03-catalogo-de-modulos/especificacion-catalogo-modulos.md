# SIRACOM — Catálogo de Módulos, Límites y Dependencias

**Documento:** Especificación Funcional de Módulos, Límites y Dependencias del Sistema  
**Versión:** 0.1 (Borrador inicial)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y validación por Jeanpier)  
**Ubicación en repositorio:** `docs/03-catalogo-de-modulos/especificacion-catalogo-modulos.md`  
**Referencia arquitectónica:** ADR-001, ADR-005, ADR-009, ADR-010 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))  
**Registro de decisiones pendientes:** [Registro Unificado de Pendientes](../00-decisiones/pendientes.md)

---

## 1. Principios de Modularidad y Reglas de Acoplamiento

El diseño modular de SIRACOM se fundamenta en los siguientes principios arquitectónicos derivados del ADR-001 (instalaciones independientes), ADR-005 (desacoplamiento comercial-clínico) y ADR-010 (decisiones de producto confirmadas):

1. **Núcleo de Datos Maestro Permanente (Padrón Animal y Responsables):** El registro y la identificación de animales y sus responsables constituyen el núcleo central del sistema. **Siempre se encuentra activo** y no admite desactivación en ninguna instalación. Todos los módulos operativos (clínica, campañas, albergue, caja) se anclan a este padrón común.
2. **Dependencias Unidireccionales Estrictas:** Ningún módulo de gestión de soporte o comercial puede condicionar, bloquear ni interrumpir el funcionamiento de los módulos sustantivos o clínicos. En particular:
   - El módulo clínico depende del padrón de animales, pero **jamás depende de tarifas, cobros, facturación ni caja** (ADR-005).
   - El módulo de campañas masivas depende del padrón de animales, pero puede operar con o sin control de existencias de farmacia.
   - El módulo comercial consume órdenes generadas por la atención clínica o mostrador, pero la caída de caja o la desactivación de tarifas no impide brindar ni registrar actos veterinarios.
3. **Catálogo de Productos y Trazabilidad vs. Control Físico de Existencias:** Se establece una separación formal entre el **Catálogo Maestro de Productos, Biológicos y Lotes** (dato de trazabilidad de plataforma, accesible para registrar vacunas y medicamentos aplicados en clínica o campo) y el módulo de **Farmacia y Control Físico de Existencias** (gestión de almacenes, saldos numéricos, mermas, transferencias y consumos automáticos). De este modo, una entidad puede registrar la inoculación de una vacuna con su lote sin necesidad de activar el control de stock de almacén.
4. **Activación Modular Configurable por Cliente:** Cada instalación independiente activa únicamente los módulos contratados y necesarios para su modelo de operación (gestión municipal, clínica veterinaria privada o albergue/ONG), ocultando de la interfaz y deshabilitando de la API las funcionalidades no autorizadas o innecesarias.

> [!NOTE]
> **Relación con el Mecanismo de Activación y Licenciamiento (PEND-20):**  
> El presente catálogo define la frontera lógica, las dependencias y los contratos funcionales entre módulos. La especificación técnica del mecanismo mediante el cual se activan o bloquean estos módulos en cada instalación (archivos de licencia criptográfica local, banderas de configuración en base de datos o variables de entorno del contenedor Docker, preservando la autonomía local sin validaciones remotas que comprometan la disponibilidad) queda diferida y sujeta a resolución bajo el ítem **PEND-20** en la Etapa 8.

---

## 2. Decisión Arquitectónica: Citas y Agenda (¿Módulo Propio o Integrado a Clínica?)

### 2.1 Decisión Adoptada
**Se decide definir "Citas y Agenda" como un Módulo Funcional Propio e Independiente (MOD-02), articulado pero desacoplado de la Gestión Médica Veterinaria (MOD-03).**

### 2.2 Justificación
1. **Diferenciación de actores y ciclo de vida:** El agendamiento de citas, asignación de turnos y control de recepción es operado fundamentalmente por el **Recepcionista / Administrativo (ROL-06)** y personal de ventanilla, quienes no poseen ni requieren competencias ni permisos clínicos.
2. **Transversalidad a múltiples dominios:** Una agenda en SIRACOM no solo reserva consultas médicas veterinarias generales; se utiliza para:
   - Programación de turnos quirúrgicos y esterilizaciones (clínica).
   - Agendamiento de citas para entrevistas de postulación y evaluación socio-habitacional de adopción (albergue).
   - Turnos para trámites administrativos presenciales o empadronamiento masivo de mascotas (padrón).
3. **Degradación operativa limpia y flexibilidad:**
   - Si una clínica u hospital municipal atiende exclusivamente por **demanda espontánea o triaje de urgencia por orden de llegada**, el módulo de Agenda puede desactivarse por completo sin perturbar el registro de consultas médicas ni historiales clínicos.
   - Si una institución pequeña o albergue no cuenta con servicios clínicos fijos pero requiere coordinar citas para visitas de adoptantes o empadronamiento distrital, puede activar Agenda sin verse forzada a habilitar la historia clínica veterinaria.

---

## 3. Matriz General de Dependencias entre Módulos

La siguiente tabla establece formalmente los contratos de acoplamiento del sistema, definiendo qué módulos exige cada componente, cuáles son integraciones optativas y cuáles representan dependencias estrictamente prohibidas:

| Código Módulo | Módulo Funcional | Dependencia Obligatoria (Exige) | Integración Opcional (Si está activo) | Dependencia Prohibida (Nunca puede depender de) | Justificación Arquitectónica |
|---|---|---|---|---|---|
| **SRV-01** | **Servicios de Plataforma y Seguridad** | Ninguna (Es la base) | Ninguna | Cualquier módulo operativo de negocio | Infraestructura base, bitácoras segregadas, seguridad y archivos. No conoce lógica de negocio. |
| **MOD-01** | **Padrón de Animales y Responsables** | SRV-01 | MOD-06 (Adopciones), MOD-07 (Comercial) | MOD-02, MOD-03, MOD-04, MOD-05, MOD-07 | Núcleo maestro del sistema. El animal y su tenedor existen independientemente de que se enfermen, vacunen o compren servicios. |
| **MOD-02** | **Citas y Agenda de Turnos** | SRV-01, MOD-01 | MOD-03 (Clínica), MOD-06 (Albergue/Adopción) | MOD-05 (Farmacia), MOD-07 (Comercial) | La reserva de cupos requiere identificar al animal/responsable, pero la agenda no debe condicionarse al cobro previo ni al stock de fármacos. |
| **MOD-03** | **Gestión Médica Veterinaria (Clínica)** | SRV-01, MOD-01 | MOD-02 (Agenda), MOD-05 (Farmacia/Stock), MOD-07 (Comercial) | **MOD-07 (Comercial / Caja / Tarifas)** | **Regla de oro ADR-005:** La atención médica y su registro inalterable jamás se detienen por falta de pago, bloqueo de caja o ausencia de tarifas. |
| **MOD-04** | **Campañas Institucionales y Brigadas** | SRV-01, MOD-01 | MOD-05 (Farmacia/Stock), MOD-08 (Reportes) | **MOD-07 (Comercial / Caja)** | Las campañas de salud pública operan en campo y son esencialmente institucionales/gratuitas; no dependen de módulos comerciales. |
| **MOD-05** | **Farmacia y Control Físico de Stock** | SRV-01 | MOD-03 (Clínica), MOD-04 (Campañas), MOD-07 (Comercial) | MOD-02, MOD-06 | Administra existencias físicas de almacén. Consume eventos de cierre clínico o balances de campaña si están activos para registrar bajas automáticas. |
| **MOD-06** | **Albergue, Custodia y Adopciones** | SRV-01, MOD-01 | MOD-03 (Clínica), MOD-02 (Agenda) | MOD-04 (Campañas), MOD-07 (Comercial) | La custodia de animales rescatados y su adopción operan sobre el padrón. El soporte médico lo brinda la clínica sin subordinar el albergue a ella. |
| **MOD-07** | **Gestión Comercial Configurable (POS/Caja)**| SRV-01 | MOD-01 (Animales), MOD-03 (Clínica), MOD-05 (Farmacia) | **Ningún módulo clínico o sanitario puede depender de MOD-07** | Consume órdenes de pago de actos clínicos o vende productos de mostrador. Si vende con descuento de stock, se integra con MOD-05; si MOD-05 no está activo, vende servicios o conceptos tarifados. |
| **MOD-08** | **Reportes Consolidados e Indicadores** | SRV-01 | Lee datos de todos los módulos que se encuentren activos | Ningún módulo puede depender de MOD-08 para operar | Módulo desacoplado de inteligencia operativa y epidemiológica. Es 100% de solo lectura. Su caída o desactivación no afecta la operación transaccional. |

---

## 4. Perfiles de Activación Modular por Tipo de Organización

SIRACOM provee combinaciones modulares predefinidas y verificadas para agilizar el despliegue según el modelo institucional del cliente:

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              PERFILES PRECONFIGURADOS DE INSTALACIÓN                                   │
├──────────────────────────────────────┬─────────────────────────────────────────────────────────────────┤
│ Perfil de Instalación                │ Módulos Activos por Defecto y Opciones                          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **1. Municipalidad**                 │ • **Activos por defecto:** MOD-01 (Padrón), MOD-02 (Agenda),    │
│    *(Cliente de referencia           │   MOD-03 (Clínica/Preventiva), MOD-04 (Campañas de Campo),      │
│     confirmado - ADR-010)*           │   MOD-05 (Farmacia/Insumos) y MOD-06 (Albergue/Adopciones).     │
│                                      │ • **Opcional / Configurable:** MOD-07 (Comercial/Caja) según si │
│                                      │   la ordenanza cobra tasas TUPA o es 100% subvencionada.        │
│                                      │   MOD-08 (Reportes) activo para vigilancia epidemiológica.      │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **2. Clínica Veterinaria Privada**   │ • **Activos por defecto:** MOD-01 (Padrón), MOD-02 (Agenda),    │
│                                      │   MOD-03 (Clínica y Cirugía), MOD-05 (Farmacia y Compras),      │
│                                      │   MOD-07 (Comercial, Tarifas y Caja) y MOD-08 (Reportes).       │
│                                      │ • **Desactivados por defecto:** MOD-06 (Albergue).              │
│                                      │ • **Opcional activable:** MOD-04 (Campañas de vacunación/castra-│
│                                      │   ción privada en campo, desactivada inicialmente).             │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **3. Albergue / Organización ONG**   │ • **Activos por defecto:** MOD-01 (Padrón), MOD-02 (Agenda de   │
│                                      │   visitas), MOD-06 (Custodia y Adopciones), MOD-05 (Botiquín de │
│                                      │   insumos básico) y MOD-08 (Indicadores de rescate).            │
│                                      │ • **Desactivados por defecto:** MOD-04 (Campañas masivas) y     │
│                                      │   MOD-07 (Comercial). MOD-03 (Clínica) activable si cuenta con  │
│                                      │   tópico veterinario propio.                                    │
└──────────────────────────────────────┴─────────────────────────────────────────────────────────────────┘
```

---

## 5. Especificación Detallada de Módulos del Sistema

---

### MÓDULO 00: Servicios Transversales de Plataforma y Seguridad (SRV-01)
* **Propósito:** Proveer la infraestructura base de autenticación, control de accesos, auditoría estricta segregada, almacenamiento seguro de documentos de gobierno y soporte transversal de parametrización de marca desacoplada (ADR-010).
* **Submódulos:**
  1. *Autenticación y Sesiones:* Control de credenciales, bloqueo de cuentas y soporte de segundo factor (2FA) conforme a **PEND-17**.
  2. *Gobernanza de Roles y Permisos:* Asignación de 8 roles base y paquetes configurables (Auditor, Coordinador de Campaña, Albergue), con prohibición estricta de autoasignación.
  3. *Bitácora Técnica Segregada:* Registro de eventos de servidor, accesos de red y caídas, consultable por TI, Jefatura y Auditor sin acceso a datos clínicos ni personales.
  4. *Bitácora de Negocio Restringida:* Registro inmutable de eventos de dominio, adendas, anulaciones y cierres de caja, inaccesible para TI.
  5. *Custodia de Evidencias y Archivos:* Almacenamiento seguro de evidencias de aprobación de Alta Dirección, expedientes y actas.
  6. *Motor de Parametrización de Marca:* Configuración centralizada de la denominación institucional del producto (ADR-010).
  7. *Catálogo Nuclear de Biológicos, Fármacos y Lotes:* Trazabilidad básica de nombres de medicamentos y lotes para uso transversal, independiente del control físico de almacén.
* **Actores participantes:** Administrador TI Local (ROL-01), Jefe de Área (ROL-02), Auditor (ROL-09), Alta Dirección (Gobernanza).
* **Datos que posee (Fuente de verdad):** Cuentas de usuario, credenciales cifradas, sesiones activas, asignaciones de roles, registros de logs técnicos y de negocio, catálogo nuclear de productos/lotes y configuración de marca.
* **Datos que consume:** Ninguno (es la base del sistema).
* **Dependencias:** Ninguna.
* **Comportamiento cuando está desactivado:** **No admite desactivación.** Es el núcleo técnico operativo indispensable para la ejecución del contenedor.

---

### MÓDULO 01: Padrón de Animales y Responsables (MOD-01)
* **Propósito:** Gestionar de forma única, auditable y centralizada la identidad, características biométricas, señas particulares, historial de vida y datos de contacto de los tenedores y animales intervenidos por la institución.
* **Submódulos:**
  1. *Identificación Animal:* Ficha biográfica (especie, raza, sexo, edad estimada o fecha de nacimiento, color, pelaje, señas particulares, fotografía), código de microchip validado (PEND-21) y estado vital (vivo, fallecido).
  2. *Padrón de Responsables / Tenedores:* Datos personales de ciudadanos tenedores (documento de identidad, nombres, domicilio, teléfonos, correo), consentimiento informado de datos personales y registro de derechos ARCO conforme a **PEND-05**.
  3. *Trazabilidad de Custodia y Vínculos:* Historial de transferencias de titularidad, reasignaciones voluntarias y registros de extravío conforme a **PEND-10**.
* **Actores participantes:** Recepcionista / Administrativo (ROL-06), Médico Veterinario (ROL-03), Técnico Asistente (ROL-04), Brigadista de Campo (ROL-05), Jefe de Área (ROL-02), Encargado de Albergue (ROL-11).
* **Datos que posee (Fuente de verdad):** Identificador único del animal (GUID), número de microchip/placa, ficha de vida animal, datos del tenedor responsable, vínculos jurídicos de tenencia y consentimiento de protección de datos personales.
* **Datos que consume:** Catálogo de especies y razas de SRV-01.
* **Dependencias:** SRV-01 (Plataforma y Seguridad).
* **Comportamiento cuando está desactivado:** **No admite desactivación en ninguna instalación.** Es el núcleo maestro del que dependen todas las funciones sustantivas y administrativas del software.

---

### MÓDULO 02: Citas y Agenda de Turnos (MOD-02)
* **Propósito:** Planificar, organizar y controlar la disponibilidad de cupos, profesionales y consultorios para atenciones médicas, intervenciones quirúrgicas programadas, trámites administrativos de empadronamiento o entrevistas de adopción.
* **Submódulos:**
  1. *Calendario y Disponibilidad de Recursos:* Configuración de turnos de consultorios fijos, horarios de atención y profesionales asignados.
  2. *Reserva y Confirmación de Citas:* Asignación de cupo vinculado al animal y responsable registrado, con indicación del motivo de la cita (consulta, vacuna, cirugía, trámite, adopción).
  3. *Sala de Espera y Turnos de Llegada:* Registro de presentación del paciente en ventanilla y control de cola para derivación a triaje o consultorio.
* **Actores participantes:** Recepcionista / Administrativo (ROL-06), Médico Veterinario (ROL-03), Asistente Veterinario (ROL-04).
* **Datos que posee (Fuente de verdad):** Calendarios de atención, reservas de citas (fecha, hora, estado: programada, confirmada, atendida, cancelada, inasistencia), motivos de cita y registros de cola en sala de espera.
* **Datos que consume:** Ficha del animal y responsable de MOD-01; identidad de profesionales veterinarios de SRV-01.
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La institución atiende exclusivamente por demanda espontánea. Los pacientes que llegan a ventanilla pasan de forma directa al registro de triaje o consulta en MOD-03 sin reserva previa de turno.

---

### MÓDULO 03: Gestión Médica Veterinaria (MOD-03)
* **Propósito:** Registrar cronológica e inalterablemente el historial clínico de los animales atendidos en consultorio o quirófano fijo, garantizando el cumplimiento del flujo profesional en 4 fases, la exigibilidad de colegiatura del profesional actuante y la trazabilidad de prescripciones farmacológicas y quirúrgicas.
* **Submódulos:**
  1. *Triaje Preliminar y Constantes Vitales (Fase 1: Operativa):* Pesaje, temperatura, frecuencia cardíaca/respiratoria, tiempo de llenado capilar y condición corporal (registrado por Asistente ROL-04 o Médico ROL-03).
  2. *Consulta Médica, Diagnósticos y Evolución (Fase 2: Validación Profesional):* Anamnesis, exploración física por sistemas, diagnósticos presuntivos y definitivos, y planes terapéuticos exclusivos del Médico Veterinario (ROL-03).
  3. *Prescripción y Emisión de Recetas:* Emisión estructurada de recetas médicas con denominación de fármaco, posología, vía y duración del tratamiento.
  4. *Procedimientos Preventivos en Sede:* Registro de vacunaciones (laboratorio, lote nuclear) y desparasitaciones, calculando fechas sugeridas de revacunación.
  5. *Actos Quirúrgicos y Hospitalización:* Protocolos pre-quirúrgicos, consentimientos informados de anestesia, detalle de intervención quirúrgica (esterilizaciones, cirugías de tejidos blandos) e incidencias.
  6. *Cierre Clínico y Adendas Médicas (Fases 3 y 4):* Bloqueo formal de la atención en la aplicación con firma/colegiatura profesional (PEND-08) y registro de adendas posteriores fechadas por el médico tratante o titular sustituto justificado (PEND-27).
* **Actores participantes:** Médico Veterinario colegiado (ROL-03, profesional exclusivo), Técnico / Asistente Veterinario (ROL-04, apoyo operativo en Fase 1), Jefe de Área (ROL-02, supervisión y suplencia).
* **Datos que posee (Fuente de verdad):** Episodios clínicos, constantes de triaje, diagnósticos, recetas emitidas, protocolos quirúrgicos, registros de vacunación clínica, estado de cierre de atención y adendas médicas auditadas.
* **Datos que consume:** Identidad de animal y responsable de MOD-01; catálogo de lotes y productos de SRV-01; citas programadas de MOD-02 (opcional).
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La instalación no registra historias clínicas ni atención médica individualizada. Típico de centros que operan únicamente como albergues puros de custodia o puestos administrativos de empadronamiento distrital.

---

### MÓDULO 04: Campañas Institucionales y Brigadas de Campo (MOD-04)
* **Propósito:** Planificar, desplegar y ejecutar intervenciones masivas de salud pública en campo (vacunación antirrábica masiva, desparasitación canina/felina, esterilizaciones comunitarias itinerantes y empadronamiento barrial) mediante interfaces optimizadas para digitación ágil en campo conectado (ADR-003).
* **Submódulos:**
  1. *Planificación y Asignación de Brigadas:* Definición de campaña (nombre, meta sanitaria, fechas, polígono territorial/barrio), designación del Médico Veterinario garante (PEND-13) y acreditación de brigadistas de campo (ROL-05).
  2. *Asignación y Retiro de Insumos de Campaña:* Registro de dotación de biológicos y microchips despachados hacia la brigada, con prevención estricta de autoaprobación por parte del Coordinador de Campaña conforme a **PEND-28**.
  3. *Registro Rápido de Campo Conectado:* Captura ágil y en un solo paso del animal, tenedor y biológico o procedimiento aplicado (con soporte de contingencia ante caídas de señal bajo fichas de digitación diferida según **PEND-06**).
  4. *Liquidación, Balance y Cierre de Campaña:* Cuadre de dosis aplicadas frente a frascos devueltos intactos o mermas/roturas en campo corregidas por contra-asiento justificado con aprobación de supervisor.
* **Actores participantes:** Personal de Campo / Brigadista (ROL-05), Coordinador de Campaña (ROL-10, asignado a ROL-02 o ROL-03), Médico Veterinario Garante (ROL-03), Jefe de Área (ROL-02).
* **Datos que posee (Fuente de verdad):** Registro maestro de campañas, cronograma territorial, actas de apertura y cierre de jornada, registros rápidos de intervención en campo, balance de insumos asignados vs. aplicados.
* **Datos que consume:** Padrón animal y responsables de MOD-01; catálogo de productos y lotes de SRV-01; lotes y existencias de MOD-05 (si Farmacia está activa; si Farmacia está inactiva, consume únicamente el catálogo nuclear de SRV-01 sin validar saldos de almacén).
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La instalación no gestiona brigadas de campo ni operativos sanitarios masivos. Las vacunaciones o cirugías solo pueden realizarse dentro del consultorio fijo en MOD-03.

---

### MÓDULO 05: Farmacia, Abastecimiento y Control de Existencias (MOD-05)
* **Propósito:** Controlar rigurosamente los movimientos físicos, saldos disponibles, caducidades y consumos de medicamentos, vacunas, anestésicos, material médico y microchips, garantizando el principio de movimientos solo-agregar (contra-asientos) y trazabilidad por lote y vencimiento (FEFO).
* **Submódulos:**
  1. *Control de Almacenes Físicos y Saldos:* Gestión multialmacén (Almacén Central, Botiquín de Consultorio, Custodia Temporal de Brigada de Campaña) y kardex valorizado.
  2. *Movimientos de Inventario (Solo-agregar):* Ingresos físicos, traslados entre sub-almacenes y correcciones mediante contra-asientos justificados (sin edición destructiva ni borrado de transacciones).
  3. *Consumos Automáticos por Acto Clínico o Campaña:* Descuento directo de stock gatillado de forma automática por el sistema ante el marcado de un acto clínico cerrado en MOD-03 o la liquidación de campaña en MOD-04.
  4. *Gestión de Ajustes, Vencimientos y Mermas:* Registro de mermas físicas, roturas y caducidades con flujo obligatorio de aprobación por rol independiente (Jefatura de Área ROL-02).
  5. *Abastecimiento, Proveedores y Compras con Costos (Submódulo Activable):* Catálogo de proveedores registrados, registro de órdenes de compra institucionales y recepción física de mercadería con registro obligatorio de costos de adquisición unitarios y totales.
* **Actores participantes:** Encargado de Farmacia / Almacén (ROL-07), Jefe de Área (ROL-02, aprobación de mermas y compras), Médico Veterinario (ROL-03, solicitud de despacho a consultorio), Coordinador de Campaña (ROL-10, recepción de dotación).
* **Datos que posee (Fuente de verdad):** Catálogo ampliado de insumos, ubicaciones físicas, kardex de transacciones de stock, saldos por lote y fecha de vencimiento, órdenes de compra y costos históricos de adquisición.
* **Datos que consume:** Catálogo nuclear de fármacos/lotes de SRV-01; atenciones cerradas de MOD-03 (para bajas de medicamentos aplicados); balance de liquidación de MOD-04 (para descargas de campo).
* **Dependencias:** SRV-01.
* **Comportamiento cuando está desactivado:** El sistema opera sin control cuantitativo de existencias físicas en almacén. En MOD-03 y MOD-04 se continúa registrando el nombre del medicamento, dosis y número de lote aplicado al animal (garantizando la trazabilidad sanitaria nuclear), pero el sistema no valida saldos ni genera descuentos automáticos de inventario.

---

### MÓDULO 06: Albergue, Custodia Temporal y Adopciones (MOD-06)
* **Propósito:** Administrar integralmente el ingreso, alojamiento físico, bienestar no médico y formalización de adopciones responsables de animales rescatados, en abandono o decomisados institucionalmente. Se estructura en dos submódulos funcionalmente separables:
* **Submódulos:**
  1. *Submódulo 6.1: Custodia Física y Alojamiento (Caniles):*
     - Registro de ingreso y procedencia (abandono en vía pública, rescate por maltrato, decomiso, entrega voluntaria).
     - Control de ocupación de espacios físicos (caniles individuales, gateras colectivas, pabellones de cuarentena y aislamiento).
     - Bitácora diaria de estancia: registro no clínico de pautas alimentarias, socialización, paseos y evolución conductual.
     - *[Nota de alcance / A confirmar]:* Soporte funcional para la figura de "Hogares Temporales Externos" (familias de acogida que custodian al animal antes de su adopción definitiva, sin ocupar canil físico municipal).
  2. *Submódulo 6.2: Formalización de Adopciones Responsables:*
     - Registro y empadronamiento de postulantes a adopción.
     - Ficha de evaluación socio-habitacional y verificación de aptitud del adoptante.
     - Emisión y suscripción del Acta de Compromiso de Tenencia Responsable.
     - Transición de custodia en MOD-01 vinculando al nuevo titular y cierre de estancia en albergue.
     - Planificación y registro de visitas o llamadas de fiscalización y seguimiento post-adopción con registro fotográfico.
* **Actores participantes:** Encargado de Albergue / Canil (ROL-11), Jefe de Área (ROL-02, autorización de bajas/adopciones), Recepcionista (ROL-06, atención de postulantes en caso de no existir ROL-11), Asistente Veterinario (ROL-04, apoyo en bitácora).
* **Datos que posee (Fuente de verdad):** Registro de ingresos a custodia, asignación de caniles, bitácoras conductuales, postulaciones a adopción, actas contractuales de compromiso y evidencias de visitas post-adopción.
* **Datos que consume:** Identidad del animal y ficha de vida de MOD-01; atenciones clínicas y altas sanitarias veterinarias de MOD-03; agenda de citas para entrevistas de MOD-02 (opcional).
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La instalación no gestiona custodia física ni procesos institucionales de rescate y adopción. Los animales registrados en MOD-01 corresponden exclusivamente a mascotas particulares con tenedor conocido.

---

### MÓDULO 07: Gestión Comercial Configurable (MOD-07)
* **Propósito:** Proveer capacidades de punto de venta (POS), cobro de tarifas de servicios veterinarios, venta de mostrador de farmacia/accesorios y arqueo diario de caja en establecimientos comerciales o dependencias públicas tarifadas (TUPA), operando con total desacoplamiento del flujo clínico (ADR-005).
* **Submódulos:**
  1. *Catálogo de Tarifas y Servicios:* Configuración de precios, tasas o conceptos de cobro por atenciones, vacunas, cirugías, desparasitaciones o certificados.
  2. *Órdenes de Cobro y Facturación Interna:* Generación de recibos internos o notas de cobro vinculadas a una atención médica previa generada en MOD-03 o por derivación directa.
  3. *Venta de Mostrador Desacoplada (POS Directo):* Emisión de órdenes de cobro para venta directa de productos o servicios sueltos sin necesidad de que exista una atención clínica ni un animal registrado (con asociación opcional al animal/responsable si se encuentra en ventanilla).
  4. *Arqueo y Cierre Diario de Caja:* Registro de turno, fondo inicial de caja chica, registro de pagos por medio (efectivo, POS/tarjeta, transferencias bancarias), solicitud de arqueo preliminar y aprobación obligatoria de cierre por supervisor (ROL-02).
  5. *Protocolo de Anulación Controlada:* Solicitud obligatoria justificada de anulación de recibos internos (`S`), requiriendo aprobación formal de supervisor (`A`) bajo **PEND-09**.
* **Actores participantes:** Cajero / Cobranzas (ROL-08), Jefe / Supervisor de Área (ROL-02, aprobación obligatoria de arqueo, cierre y anulaciones).
* **Datos que posee (Fuente de verdad):** Catálogo de tarifas vigentes, órdenes de cobro, recibos internos emitidos, cierres de turno de caja, cuadre de medios de pago y solicitudes/aprobaciones de anulación.
* **Datos que consume:** Servicios prestados en atenciones clínicas de MOD-03 (consumo de solo lectura para liquidación); catálogo de insumos de MOD-05 (si está activo para ventas con baja de stock de mostrador).
* **Dependencias:** SRV-01. Integración opcional con MOD-01 (si se asocia a un animal), MOD-03 (si cobra una consulta) y MOD-05 (si descuenta existencias de farmacia al vender).
* **Comportamiento cuando está desactivado:** Todos los servicios institucionales se brindan sin registro de cobro ni gestión de caja. Ninguna pantalla del sistema solicita medios de pago ni muestra botones de liquidación comercial.

---

### MÓDULO 08: Reportes Consolidados e Indicadores de Gestión (MOD-08)
* **Propósito:** Consolidar, sintetizar y presentar reportes estadísticos, tableros de control e indicadores de gestión epidemiológica, cobertura de salud pública y rendimiento operativo, operando de forma 100% desacoplada en modo de solo lectura.
* **Submódulos:**
  1. *Vigilancia Epidemiológica y Zoonosis:* Estadísticas de vacunación antirrábica distrital, registro de accidentes por mordedura de animales potencialmente transmisores de rabia y casos de esterilización masiva para control poblacional (con formatos estandarizados exportables a Excel/PDF para entrega a autoridades de salud pública bajo **PEND-02**).
  2. *Métricas de Campañas Masivas:* Porcentaje de cumplimiento de metas por zona barrial, rendimiento de brigadas y ratios de cobertura territorial.
  3. *Indicadores de Bienestar y Albergue:* Tiempos promedio de permanencia en caniles, tasa de adopción de animales rescatados, tasa de retorno de extraviados y porcentaje de efectividad en seguimiento post-adopción.
  4. *Métricas Operativas y Financieras:* Resumen consolidado de ingresos por tipo de servicio, reporte de rotación de fármacos, valorización de inventario y pérdidas por mermas/vencimientos.
* **Actores participantes:** Jefe de Área (ROL-02), Auditor (ROL-09), Administrador TI (ROL-01, solo métricas técnicas), Alta Dirección (Gobernanza institucional).
* **Datos que posee (Fuente de verdad):** No genera datos transaccionales propios. Almacena consultas de agregación, definiciones de plantillas de reporte e históricos consolidados en memoria de cálculo.
* **Datos que consume:** Lee de forma no bloqueante datos de MOD-01, MOD-03, MOD-04, MOD-05, MOD-06 y MOD-07 (exclusivamente de aquellos módulos que se encuentren activados en la instalación).
* **Dependencias:** SRV-01.
* **Comportamiento cuando está desactivado:** Cada módulo individual conserva y sigue generando sus reportes operativos inmediatos (listados, fichas individuales y resúmenes diarios). Lo que se desactiva es la vista analítica consolidada y los tableros de control gerenciales y epidemiológicos agregados.

---

## 6. Remisión al Registro Unificado de Pendientes

Las decisiones abiertas e incertidumbres funcionales que interactúan con el catálogo de módulos se gestionan formalmente en:

👉 **[Registro Unificado de Decisiones Pendientes (docs/00-decisiones/pendientes.md)](../00-decisiones/pendientes.md)**

En particular, aplican a esta especificación los ítems:
* **PEND-02:** Obligatoriedad de formatos oficiales de salud pública (MINSA / DIRIS en Perú) para MOD-08.
* **PEND-03:** Modalidad de integración de facturación electrónica y comprobantes tributarios para MOD-07.
* **PEND-05:** Política institucional de protección de datos personales, ciclo de vida y retención (Ley 29733) para MOD-01.
* **PEND-06:** Procedimiento operativo de contingencia en campañas ante ausencia de conectividad (digitación diferida) para MOD-04.
* **PEND-07:** Requisitos técnicos de seguridad de bitácoras y garantías de aislamiento para SRV-01.
* **PEND-08:** Validez y mecanismo de formalización de cierre de actos médicos veterinarios y colegiatura para MOD-03.
* **PEND-09:** Límites operativos de caja chica, arqueo y protocolo de anulación de cobros para MOD-07.
* **PEND-10:** Procedimiento formal de reasignación y transferencia de dueños de animales para MOD-01.
* **PEND-14:** Marco de control, custodia y dispensación de medicamentos veterinarios regulados para MOD-05.
* **PEND-15:** Mecanismo institucional de designación del rol Auditor para SRV-01.
* **PEND-16:** Esquema de operación y asunción de riesgo para organizaciones pequeñas para SRV-01 y MOD-07.
* **PEND-17:** Política técnica de contraseñas y doble factor de autenticación (2FA) para SRV-01.
* **PEND-20:** Mecanismo de licenciamiento y activación modular por instalación (definición técnica diferida).
* **PEND-21:** Criterios de identificación animal, microchip y prevención de duplicados para MOD-01.
* **PEND-25:** Alcance y gobernanza de los "parámetros sensibles" autorizados por la Alta Dirección (a resolver antes de aprobar este catálogo).
* **PEND-27:** Asignación de médico titular de reemplazo para adendas clínicas en consultorios unipersonales para MOD-03.
* **PEND-28:** Prevención de autoaprobación del Coordinador de Campaña en el retiro de insumos para MOD-04 y MOD-05.
