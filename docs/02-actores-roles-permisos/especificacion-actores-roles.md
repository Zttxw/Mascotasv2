# SIRACOM — Especificación de Actores, Roles y Permisos

**Documento:** Especificación Funcional de Actores, Roles y Permisos  
**Versión:** 0.1 (Borrador corregido)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y aprobación por Jeanpier)  
**Ubicación en repositorio:** `docs/02-actores-roles-permisos/especificacion-actores-roles.md`  
**Referencia arquitectónica:** ADR-001, ADR-005, ADR-009 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))  
**Registro de decisiones pendientes:** [Registro Unificado de Pendientes](../00-decisiones/pendientes.md)

---

## 1. Principios de Control de Acceso y Autorización

El modelo de seguridad y control de acceso de SIRACOM se rige por los siguientes principios arquitectónicos:

1. **Principio de menor privilegio y privacidad de datos:** Todo usuario accede exclusivamente a las funciones, pantallas y datos indispensables para su labor. En particular, el Administrador TI local es un rol de soporte técnico de infraestructura y **por defecto no tiene acceso a datos clínicos de pacientes ni a datos personales de ciudadanos**.
2. **Aislamiento e independencia local (ADR-001):** Usuarios, credenciales, roles y permisos residen exclusivamente en la base de datos de cada cliente. No existen identificadores globales ni superadministradores centrales.
3. **Modularidad y activación condicional (ADR-005):** Los roles y permisos se habilitan de acuerdo con los módulos activos. Si una municipalidad no activa el módulo comercial, las opciones de caja, cobros y ventas no se muestran ni se configuran. Si una clínica privada no gestiona albergues ni campañas masivas, dichos submódulos permanecen deshabilitados sin afectar el núcleo clínico.
4. **Trazabilidad de actos clínicos y responsabilidad profesional:** Todo acto clínico, diagnóstico, cirugía o prescripción registra de forma obligatoria al Médico Veterinario actuante (incluyendo su número de colegiatura profesional). Cuando un procedimiento preventivo (vacunación o desparasitación) es inoculado por un Asistente Veterinario o un Brigadista de Campo, el sistema registra obligatoriamente dos identidades:
   - El **Médico Veterinario colegiado responsable** (garante sanitario de la atención, lote o campaña institucional).
   - El **operador o brigadista actuante** que ejecutó materialmente la inoculación o digitación.
5. **Inmutabilidad de registros cerrados:** Una vez marcada como **cerrada** la atención médica veterinaria, el registro **no admite edición ni sobrescritura**. Cualquier rectificación o aclaración posterior se efectúa exclusivamente mediante adendas médicas fechadas y auditadas que preservan íntegro el texto original.
6. **Inmutabilidad y anulación justificada:** Queda técnicamente inhabilitado el borrado físico de datos en toda la base de datos. La anulación de registros (que cubre de forma expresa **animales y responsables**, así como actos clínicos, inventario y cobros) exige consignar un motivo obligatorio y conserva el historial completo con usuario y marca de tiempo.
7. **Regla de anulación de animales con historial:** Si un animal ya posee atenciones médicas, cirugías, vacunaciones o participación en campañas, **su anulación exige aprobación obligatoria de la Jefatura de Área (ROL-02)**. Puede ser solicitada formalmente por el Médico Veterinario (ROL-03) o el Recepcionista (ROL-06) con causa justificada, impidiendo la desaparición no autorizada de historiales clínicos.
8. **Segregación de bitácoras de auditoría:**
   - **Bitácora técnica de sistema:** Registra eventos de infraestructura (inicios/cierres de sesión, bloqueos de cuenta, caídas de servicios, cambios de parámetros de red/servidor). **No contiene contenido clínico ni datos personales de ciudadanos.** Es visible para el Administrador TI (ROL-01).
   - **Bitácora de negocio:** Registra eventos de dominio (modificaciones de historias clínicas, consultas de expedientes, solicitudes y aprobaciones de anulación, dispensación de insumos, cierres de caja y asignación de roles). Es **estrictamente confidencial** y accesible únicamente para la Jefatura de Área (ROL-02) y el Auditor (ROL-09). El personal técnico de TI no tiene acceso a esta bitácora.
9. **Gobernanza y asignación de roles sensibles:** La creación, elevación de privilegios o revocación de cuentas con roles sensibles (Administrador TI, Jefe de Área, Médico Veterinario, Cajero y Auditor) **requiere aprobación formal de la Jefatura de Área (ROL-02)**. Está **estrictamente prohibida la autoasignación de roles**: ningún usuario puede modificar sus propios privilegios. Toda alteración de roles queda registrada de forma inalterable en la bitácora de negocio.

---

## 2. Clasificación de Actores del Sistema

```
                               ┌────────────────────────────────┐
                               │       ACTORES DE SIRACOM       │
                               └───────────────┬────────────────┘
                                               │
       ┌───────────────────────────────────────┼───────────────────────────────────────┐
       ▼                                       ▼                                       ▼
┌───────────────────────┐           ┌───────────────────────┐               ┌───────────────────────┐
│    ACTORES INTERNOS   │           │   SUJETOS REGISTRADOS │               │   ACTOR EXCEPCIONAL   │
│ (Usuarios de la org.) │           │    (Sin credenciales) │               │   (ADR-001 / ADR-002) │
└───────────┬───────────┘           └───────────┬───────────┘               └───────────┬───────────┘
            │                                   │                                       │
            │                                   ▼                                       ▼
            │                       ┌───────────────────────┐               ┌───────────────────────┐
            │                       │ • Ciudadanos / Dueños │               │ • Soporte Técnico del │
            │                       │ • Postulantes Adopción│               │   Proveedor           │
            │                       │ • Proveedores Insumos │               │   (autorizado,        │
            │                       └───────────────────────┘               │   temporal y auditado)│
            ▼                                                               └───────────────────────┘
┌───────────────────────────────────────────────┐
│ ROLES BASE ESTÁNDAR:                          │
│ • ROL-01: Administrador TI Local              │
│ • ROL-02: Jefe / Supervisor de Área           │
│ • ROL-03: Médico Veterinario (Colegiado)      │
│ • ROL-04: Técnico / Asistente Veterinario     │
│ • ROL-05: Personal de Campo / Brigadista      │
│ • ROL-06: Recepcionista / Administrativo      │
│ • ROL-07: Encargado de Farmacia / Almacén     │
│ • ROL-08: Cajero / Cobranzas                  │
│                                               │
│ CONJUNTOS DE PERMISOS CONFIGURABLES:          │
│ (ROLES PROPUESTOS, PENDIENTES DE DECISIÓN)    │
│ • ROL-09: Auditor (Solo lectura)              │
│ • ROL-10: Coordinador de Campaña              │
│ • ROL-11: Encargado de Albergue / Canil       │
└───────────────────────────────────────────────┘
```

### 2.1 Actores Internos (Personal Operativo y de Gestión)
Usuarios pertenecientes a la entidad (municipalidad o clínica veterinaria) que inician sesión con credenciales individuales y ejercen roles asignados según su función laboral.

### 2.2 Sujetos Registrados (Sin Acceso al Sistema en esta Fase)
Personas registradas por los operadores cuya información es custodiada por la institución, sin contar con credenciales de usuario:
* **Ciudadanos / Tenedores de mascotas:** Propietarios o responsables registrados en atenciones o empadronamientos.
* **Postulantes a adopción:** Ciudadanos evaluados en el proceso de adopción de animales rescatados.
* **Proveedores de medicamentos e insumos:** Entidades comerciales registradas para trazabilidad de compras y costos de inventario.

### 2.3 Actor Excepcional: Soporte Técnico del Proveedor (ADR-001 y ADR-002)
* **Naturaleza excepcional:** El desarrollador o proveedor de SIRACOM **no tiene acceso permanente ni implícito** a los datos de producción de la instalación de ningún cliente. Aun en la modalidad donde el proveedor aloja la infraestructura (ADR-002), el acceso operativo a la aplicación y a los datos de negocio reside exclusivamente bajo control de la organización cliente.
* **Protocolo de intervención técnica:** En caso de requerirse soporte extraordinario de mantenimiento o resolución de incidentes:
  1. Debe mediar una solicitud formal y autorización expresa de la jefatura responsable del cliente.
  2. El Administrador TI local habilita una credencial temporal de soporte técnico con expiración horaria automática.
  3. Cada comando, consulta y acción técnica ejecutada por el personal de soporte queda registrada de forma inalterable en la bitácora técnica de la instalación.
  4. La intervención se realiza bajo principio de estricta minimización de datos personales y clínicos.
* **Garantía técnica de aislamiento:** Las salvaguardas técnicas sobre la bitácora inalterable y la segregación de accesos en entornos alojados por el proveedor quedan sujetas a la resolución del ítem **PEND-07**.

---

## 3. Catálogo de Roles Base Estándar (8 Roles)

| Rol | Denominación | Propósito y Delimitación de Responsabilidades |
|---|---|---|
| **ROL-01** | **Administrador del Sistema Local (TI)** | **Rol estrictamente técnico de infraestructura.** Gestiona cuentas de usuarios, asigna roles previamente aprobados por Jefatura, configura sedes físicas, parámetros de red y conectividad, y monitorea respaldos y la bitácora técnica. **Por defecto no tiene acceso a datos clínicos de pacientes ni a datos personales de ciudadanos o responsables** (menor privilegio). No administra catálogos de negocio (farmacia, tarifas ni parámetros clínicos). |
| **ROL-02** | **Jefe / Supervisor de Área (Zoonosis / Bienestar)** | Supervisión estratégica y gobernanza: aprueba asignación de roles sensibles, aprueba anulaciones de animales con historial, autoriza ajustes/mermas de inventario, valida cierres de caja, aprueba exportaciones masivas con motivo justificado y emite reportes consolidados de gestión. |
| **ROL-03** | **Médico Veterinario (Colegiado)** | Responsabilidad médica legal: emite diagnósticos, prescribe medicamentos, realiza cirugías y procedimientos clínicos, administra catálogos de parámetros clínicos (especies, razas, causales, vías), y cierra atenciones registrando su número de colegiatura. Actúa como garante sanitario en campañas masivas. |
| **ROL-04** | **Técnico / Asistente Veterinario** | Soporte clínico operativo: toma de constantes vitales en triaje (peso, temperatura), sujeción física, y aplicación de vacunas y desparasitaciones bajo la supervisión o protocolo del Médico Veterinario responsable registrado. |
| **ROL-05** | **Personal de Campo / Brigadista** | Operación en brigadas de campaña: registro rápido de animales y responsables en campo conectado e inoculación de biológicos bajo la supervisión del Médico Veterinario garante de campaña registrado. |
| **ROL-06** | **Recepcionista / Administrativo** | Atención al público y ventanilla: alta y actualización de datos de contacto de responsables y animales, gestión de agenda y citas, y derivación a consultorio. Puede solicitar formalmente la anulación de registros a Jefatura. |
| **ROL-07** | **Encargado de Farmacia / Almacén** | Control de existencias físicas y costos: administración del catálogo de fármacos e insumos (por conocimiento especializado), registro de proveedores y compras, recepción con lote y vencimiento, transferencias, registro de movimientos en modo solo-agregar (contra-asientos) y solicitud de ajustes/mermas. |
| **ROL-08** | **Cajero / Cobranzas** | Gestión de ingresos en caja (módulo comercial activo): apertura y arqueo diario preliminar de caja, cobro de órdenes generadas por atención o mostrador y emisión de recibos internos de cobro. Los límites máximos de caja y protocolos de anulación de comprobantes son provisionales y sujetos a **PEND-09**. |

---

## 4. Conjuntos de Permisos Configurables (Roles Propuestos, Pendientes de Decisión)

Los siguientes tres conjuntos de permisos no forman parte del estándar cerrado de roles y se catalogan formalmente como **"Propuestos, pendientes de decisión"** bajo **PEND-11**. Se detallan los permisos específicos que aportarían, justificando su asignación o tratamiento alternativo como responsabilidades asignables:

### 4.1 Rol 09: Auditor (Solo Lectura) — Propuesto, pendiente de decisión
* **Permisos específicos:** Acceso de lectura transversal y exportación a **todos** los módulos del sistema (historias clínicas, movimientos de inventario, campañas, caja y bitácoras de auditoría) sin permiso de creación, modificación ni anulación en ninguno de ellos.
* **Salvaguardas de seguridad:**
  1. Totalmente independiente de la operación diaria (no registra ni aprueba actos).
  2. **Sus propias consultas y exportaciones quedan registradas de forma inmutable en el log de auditoría del sistema** (auditoría del auditor), impidiendo inspecciones no trazadas.
  3. Las exportaciones masivas de datos exigen la consignación obligatoria de un motivo justificado en pantalla.
* **Propuesta de tratamiento / fusión:** Se propone mantenerlo como un **conjunto de permisos configurable por instalación**, habilitado exclusivamente cuando la organización requiera dar acceso a inspectores sanitarios u órganos de control institucional (OCI).

### 4.2 Rol 10: Coordinador de Campaña — Propuesto, pendiente de decisión
* **Permisos específicos:** Planificación de campañas masivas institucionales, designación de brigadas de campo, retiro de lotes de insumos desde el almacén y liquidación del balance final de insumos y cobertura de la jornada.
* **Análisis de exclusividad y propuesta de tratamiento:** No posee permisos que no puedan ser ejercidos por el **Jefe de Área (ROL-02)** o por un **Médico Veterinario (ROL-03)** con funciones de campo. Su creación como rol independiente fijo sobrecargaría innecesariamente la matriz de perfiles. Se propone tratarlo como una **responsabilidad asignable** a usuarios existentes de nivel supervisor o médico durante la vigencia de una campaña.

### 4.3 Rol 11: Encargado de Albergue / Canil — Propuesto, pendiente de decisión
* **Permisos específicos:** Asignación física y control de ocupación de caniles/gateras, registro de bitácoras diarias de comportamiento y alimentación (no médicas), y recepción/evaluación de postulaciones de adopción.
* **Análisis de exclusividad y propuesta de tratamiento:** En clínicas privadas o municipalidades sin instalaciones de custodia física, este conjunto de permisos es innecesario. Se propone mantenerlo como un **conjunto de permisos configurable por instalación**, activable únicamente si se habilita el submódulo de albergue físico. Si la entidad no cuenta con un cuidador exclusivo, las funciones de adopción se asignan al **Recepcionista (ROL-06)** y las de bitácora diaria al **Asistente Veterinario (ROL-04)**.

---

## 5. Matriz de Permisos por Dominio Funcional y Rol

### 5.1 Convención Formal de Acciones de Acceso
- **R (Read):** Ver / Consultar registros y pantallas.
- **C (Create):** Crear / Registrar nuevos registros.
- **U (Update):** Editar / Modificar registros abiertos o datos de contacto.
- **D (Disable/Annul):** Anular / Desactivar con motivo justificado obligatorio (sin borrado físico).
- **A (Approve):** Aprobar / Validar formalmente (cierre médico, actas de adopción, ajustes de stock, cierres de caja).
- **X (Export):** Exportar reportes en formatos externos (Excel, PDF, CSV) con motivo obligatorio en descargas masivas.
- **M (Manage):** Administrar parámetros de configuración, cuentas y permisos técnicos.
- **- :** Sin acceso a la función o pantalla.

### 5.2 Matriz Granular de Autorización

| Dominio Funcional | Admin TI (01) | Jefe Área (02) | Médico Vet. (03) | Asistente (04) | Brigada Campo (05) | Recepcionista (06) | Farmacia (07) | Cajero (08) | Auditor [Prop.] (09) | Coord. Camp. [Prop.] (10) | Enc. Albergue [Prop.] (11) |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Administración Técnica y Cuentas** | C-U-D-M | R-A (1) | - | - | - | - | - | - | R | - | - |
| **Bitácora Técnica de Sistema** | R-X | R | - | - | - | - | - | - | R-X | - | - |
| **Bitácora de Negocio (Auditoría)** | - (2) | R-X (3) | - | - | - | - | - | - | R-X (3)(4)| - | - |
| **Configuración Institucional (Técnica)** | C-U-M | R-U | R | R | R | R | R | - | R | R | R |
| **Parámetros Clínicos (Especies/Razas)**| - (5) | R-U-A | C-R-U-M | R | R | R | R | - | R | R | R |
| **Animales y Responsables (Fichas)** | - (6) | R-U-D-A-X| C-R-U (7)| C-R-U | C-R | C-R-U (7)| R | R | R-X (3)| R-U | C-R-U-D |
| **Citas y Agenda** | - | R-U-A | C-R-U | C-R-U | - | C-R-U-D | - | R | R | - | - |
| **Triaje y Constantes Vitales** | - | R | C-R-U | C-R-U | - | - | - | - | R | - | R-U |
| **Consulta y Diagnóstico Clínico** | - | R-X (3)| C-R-U-A(8)| R | - | - | - | - | R-X (3)| - | R |
| **Prescripción de Medicamentos** | - | R | C-R-A | R | - | - | R | - | R | - | - |
| **Cirugías y Procedimientos Médicos** | - | R-X (3)| C-R-U-A(8)| R-U | - | - | - | - | R-X (3)| - | - |
| **Vacunación y Desparasitación** | - | R-X (3)| C-R-A | C-R (9)| C-R (9)| - | R | - | R-X (3)| R-U | R-U |
| **Planificación de Campañas** | - | C-R-U-D-A | R | R | R | - | R | - | R-X (3)| C-R-U-A | - |
| **Registro Rápido en Campaña** | - | R-X | C-R | C-R | C-R (9)| - | R | - | R-X | C-R-X | - |
| **Catálogo de Farmacia e Insumos** | - (5) | R-U-A | R | R | R | - | C-R-U-D-M| - | R | R | R |
| **Proveedores y Órdenes de Compra** | - | R-A-X | - | - | - | - | C-R-U-X | - | R-X | R | - |
| **Movimientos de Stock (Solo-agregar)** | - | R-X (3)| - | - | - | - | C-R-X(10)| - | R-X (3)| R-A | - |
| **Ajustes y Mermas de Inventario** | - | R-A (11)| - | - | - | - | C-R (11)| - | R-X (3)| - | - |
| **Dispensación a Consultorio/Campaña**| - | R | R | R | - | - | C-R-A | - | R | R-A | - |
| **Custodia en Albergue y Caniles** | - | R-U-A | R | R | - | - | - | - | R | - | C-R-U-D-A |
| **Bitácora Conductual y Alimentación** | - | R | R | R-U | - | - | - | - | R | - | C-R-U |
| **Evaluación y Postulación Adopción** | - | R-U-A | R | - | - | R | - | - | R | - | C-R-U-A |
| **Seguimiento Post-Adopción** | - | R-X (3)| R | - | - | R | - | - | R-X (3)| - | C-R-U-X |
| **Catálogo de Tarifas y Servicios** | - (5) | C-R-U-M | R | - | - | R | - | R | R | - | - |
| **Órdenes de Cobro y Recibos Internos**| - | R-X (3)| R | - | - | C-R | - | C-R-D-X(12)| R-X (3)| - | - |
| **Apertura, Cierre y Arqueo de Caja** | - | R-A-X(13)| - | - | - | - | - | C-R-X(13)| R-X | - | - |

**Reglas de seguridad y aclaraciones de la matriz:**
* **(1) Aprobación de roles sensibles:** La Jefatura de Área (ROL-02) debe aprobar formalmente la asignación de cuentas con roles sensibles. Queda prohibida la autoasignación.
* **(2) Bitácora de negocio:** El Administrador TI (ROL-01) no tiene acceso a la bitácora de negocio para preservar la privacidad de datos clínicos y personales.
* **(3) Exportaciones masivas (X):** Toda exportación masiva de datos por parte de Jefatura o Auditor exige consignar obligatoriamente un motivo en pantalla y genera registro de auditoría.
* **(4) Auditoría del auditor:** Cada consulta, lectura o exportación efectuada por el Auditor (ROL-09) queda registrada inalterablemente en el log de auditoría.
* **(5) Retiro de administración a TI:** TI no gestiona catálogos de negocio (farmacia, tarifas ni parámetros clínicos). Fueron asignados a Farmacia (ROL-07), Jefatura (ROL-02) y Médico Veterinario (ROL-03) por competencia funcional.
* **(6) Menor privilegio de TI:** El Administrador TI no tiene acceso a visualizar fichas de animales ni datos personales de ciudadanos.
* **(7) Anulación de animales con historial:** Médico Veterinario y Recepcionista pueden registrar la solicitud de anulación con motivo justificado, pero su aprobación y ejecución lógica corresponde exclusivamente a la Jefatura (ROL-02). No hay borrado físico.
* **(8) Inmutabilidad de atenciones cerradas:** El Médico Veterinario cierra la atención con su número de colegiatura. El registro cerrado no admite edición; rectificaciones se realizan mediante adendas auditadas.
* **(9) Vacunación delegada y campañas:** Cuando inocula Asistente o Brigadista, se registra obligatoriamente al Médico Veterinario colegiado responsable garante de la atención o campaña.
* **(10) Inventario modo solo-agregar:** Los movimientos de inventario no admiten modificación (sin U). Correcciones se realizan mediante contra-asientos justificados.
* **(11) Ajustes y mermas:** Farmacia registra la propuesta de ajuste (C); la aprobación formal (A) corresponde obligatoriamente a Jefatura (rol distinto al que registra).
* **(12) Recibos internos:** Emitidos por Cajero para control administrativo, sin validez tributaria directa (sujeto a PEND-03). Límites y contingencias de caja provisionales sujetos a PEND-09.
* **(13) Cierre de caja:** Cajero genera arqueo y cierre preliminar (C); la aprobación formal (A) corresponde obligatoriamente a un supervisor (Jefe de Área ROL-02).

---

## 6. Requerimientos de Seguridad para Capacidades Futuras (ADR-003)

Para salvaguardar la arquitectura sin introducir desarrollos prematuros en la fase actual, la futura implementación de capacidades diferidas deberá observar los siguientes lineamientos:

1. **Futuro Portal Ciudadano:**
   - La arquitectura del sistema **deberá** garantizar el desacoplamiento de interfaces, de modo que las futuras consultas de los ciudadanos se atiendan mediante servicios backend protegidos con autenticación restringida, sin exponer datos de terceros ni conceder acceso a pantallas administrativas internas.
2. **Futura Operación Offline en Campaña:**
   - La arquitectura **deberá** soportar la generación de identificadores únicos (GUID) en el cliente para eventos creados en campo, estampado temporal dual (fecha de ocurrencia del acto médico separada de la fecha de registro en base de datos) y procesamiento idempotente para evitar registros duplicados durante la sincronización remota diferida.

*Nota:* No se diseñan endpoints, tablas ni componentes de software para estas capacidades en la fase actual.

---

## 7. Remisión al Registro Unificado de Pendientes

Las decisiones abiertas, vacíos de información y aspectos por definir de esta especificación se gestionan formalmente en:

👉 **[Registro Unificado de Decisiones Pendientes (docs/00-decisiones/pendientes.md)](../00-decisiones/pendientes.md)**

En particular, aplican a esta especificación los ítems:
* **PEND-05:** Política institucional de protección de datos personales y retención (Ley 29733).
* **PEND-07:** Garantías técnicas de bitácora inalterable y cumplimiento de aislamiento (ADR-001) bajo modalidad de alojamiento por proveedor (ADR-002).
* **PEND-08:** Validez y mecanismo de formalización de cierre de actos médicos veterinarios.
* **PEND-09:** Límites operativos de caja chica, arqueo y protocolo de anulación de cobros.
* **PEND-10:** Procedimiento formal de reasignación y transferencia de dueños de animales.
* **PEND-11:** Decisión formal sobre roles propuestos (Auditor, Coordinador de Campaña, Encargado de Albergue).
