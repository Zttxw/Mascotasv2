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
4. **Flujo clínico en cuatro fases y responsabilidad profesional:** El acto clínico se estructura de forma explícita en cuatro etapas sucesivas:
   - *Fase 1: Registro operativo:* Triaje preliminar, pesaje y constantes vitales que pueden ser registrados por el Asistente Veterinario (ROL-04) o el Médico Veterinario.
   - *Fase 2: Validación profesional exclusiva:* Examen físico detallado, diagnósticos, protocolos médicos y prescripción farmacológica, ejecutados y validados de forma exclusiva por el Médico Veterinario colegiado (ROL-03).
   - *Fase 3: Cierre clínico:* Marcado formal de la atención como **cerrada** por el Médico Veterinario responsable con su colegiatura (decisión de diseño sujeta a verificación legal bajo **PEND-08**), momento a partir del cual el registro **no admite edición en la aplicación** (distinguiéndose esta regla funcional de interfaz de las garantías técnicas de seguridad de bitácoras de **PEND-07**) y se gatillan los consumos automáticos de insumos generados por el sistema.
   - *Fase 4: Rectificación posterior:* Un registro cerrado **no admite edición en la aplicación**. Cualquier aclaración o corrección posterior se efectúa exclusivamente mediante notas rectificatorias o adendas médicas fechadas y auditadas que preservan íntegro el texto original. Si el Médico Veterinario que cerró la atención original ya no labora en la institución o no se encuentra disponible, la adenda médica solo podrá ser registrada por un Médico Veterinario titular formalmente asignado al caso o por la Jefatura de Área (siendo profesional médico veterinario), consignando obligatoriamente el motivo justificado de la intervención, la referencia explícita al acto clínico original y registrando la identidad de ambos profesionales.
   - *Delegación en procedimientos preventivos:* Cuando una vacunación o desparasitación es aplicada por un Asistente Veterinario o un Brigadista de Campo, el sistema registra obligatoriamente dos identidades: el **Médico Veterinario colegiado responsable** (garante sanitario de la atención o campaña masiva) y el **operador o brigadista actuante** que ejecutó materialmente la inoculación. Las condiciones de delegación y presencia física exigida se rigen por **PEND-13**.
5. **Regla de anulación operativa y ciclo de vida de datos:** No existe borrado físico operativo en la base de datos. Las inactivaciones o anulaciones de registros exigen consignar motivo justificado obligatorio y conservan el historial completo con usuario y marca de tiempo. La conservación, rectificación, anonimización o eventual eliminación definitiva de datos de animales y responsables seguirá la política de protección y ciclo de vida de datos personales validada en **PEND-05**, sin asumir una promesa de conservación indefinida absoluta.
6. **Regla de anulación de animales con historial:** Si un animal ya posee atenciones médicas, cirugías, vacunaciones o participación en campañas, **su anulación exige aprobación obligatoria de la Jefatura de Área (ROL-02)**. Puede ser solicitada formalmente por el Médico Veterinario (ROL-03) o el Recepcionista (ROL-06) mediante solicitud justificada (`S`), impidiendo la desaparición no autorizada de historiales clínicos.
7. **Segregación de bitácoras de auditoría:**
   - *Bitácora técnica de sistema:* Registra eventos de infraestructura (inicios/cierres de sesión, bloqueos de cuenta, caídas de servicios, cambios de parámetros de red/servidor). **No contiene contenido clínico ni datos personales de ciudadanos.** Es consultable por el Administrador TI (ROL-01) para soporte técnico, por el Auditor (ROL-09) para fiscalización técnica de accesos y por la Jefatura de Área (ROL-02) para supervisión de disponibilidad del servicio.
   - *Bitácora de negocio:* Registra eventos de dominio (modificaciones de historias clínicas, consultas de expedientes, solicitudes y aprobaciones de anulación, dispensación de insumos, cierres de caja y asignación de roles). Es **estrictamente confidencial** y accesible únicamente para la Jefatura de Área (ROL-02) y el Auditor (ROL-09) dentro de su ámbito autorizado. El personal técnico de TI no tiene acceso a esta bitácora.
   - *Requisito de seguridad por validar:* Los mecanismos de protección contra alteración de bitácoras, cobertura de eventos, retención, excepciones y salvaguardas técnicas constituyen requisitos de seguridad por validar en el diseño técnico, conforme a **PEND-07**.
8. **Gobernanza de roles sensibles y seguridad de cuentas:**
   - Se clasifican formalmente como **roles sensibles**: Administrador TI (01), Jefe de Área (02), Médico Veterinario (03), Encargado de Farmacia/Almacén (07), Cajero (08) y Auditor (09).
   - **Regla del sistema:** Ningún rol sensible puede activarse ni asignarse a un usuario sin la aprobación formal registrada de la autoridad competente. Está **estrictamente prohibida la autoasignación de roles**: ningún usuario puede elevar sus propios privilegios.
   - **Reinicio seguro de contraseñas (TI sin acceso a claves):** El personal de TI **no debe conocer ni generar claves temporales legibles**. El restablecimiento de acceso se efectúa mediante un enlace seguro de un solo uso enviado directamente al correo o canal oficial del titular, o mediante la aprobación obligatoria de un segundo rol autorizado (Jefatura de Área) que autoriza la generación de un código de acceso de un solo uso que el titular ingresa directamente. El sistema fuerza el cambio inmediato de contraseña en el primer acceso. **Este evento de reinicio de credencial se registra obligatoriamente en la bitácora de negocio** (además del registro técnico de TI), notificando de inmediato al usuario titular. Los parámetros de complejidad, expiración del enlace y 2FA se rigen por la política técnica de autenticación (**PEND-17**).
9. **Segregación de funciones y exportaciones masivas:**
   - La Jefatura de Área (ROL-02) **no exporta la bitácora que registra sus propios actos**. Toda exportación masiva de auditoría de negocio o datos institucionales solicitada por Jefatura exige aprobación previa de la Alta Dirección / Gerencia General o suplente de nivel superior.
   - El Auditor (ROL-09) accede a bitácoras por finalidad y ámbito autorizado, y sus exportaciones masivas exigen motivo obligatorio y quedan auditadas.
   - *Suplencia y organizaciones pequeñas:* Se contempla la figura de un **suplente designado** para autorizaciones en ausencia de la Jefatura. Para clínicas o dependencias pequeñas con personal reducido, se prevé un **modo de organización pequeña configurable** donde la concentración de roles opera bajo asunción de riesgo aceptado y suscrito por escrito por el titular de la organización, conforme a **PEND-16**.

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
│ • ROL-09: Auditor (Solo lectura por ámbito)   │
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
* **Objetivo de diseño sujeto a validación:** El soporte técnico del proveedor no tiene acceso permanente ni implícito a los datos de producción de la instalación de ningún cliente. Aun en la modalidad donde el proveedor aloja la infraestructura (ADR-002), el acceso operativo a la aplicación y a los datos de negocio **deberá residir** exclusivamente bajo control de la organización cliente, siendo este un objetivo de diseño arquitectónico sujeto a las garantías técnicas de **PEND-07**.
* **Protocolo de intervención técnica:** En caso de requerirse soporte extraordinario de mantenimiento o resolución de incidentes:
  1. Debe mediar una solicitud formal y autorización expresa de la jefatura responsable del cliente.
  2. El Administrador TI local habilita una credencial temporal de soporte técnico con expiración horaria automática.
  3. Cada comando, consulta y acción técnica ejecutada por el personal de soporte queda registrada en la bitácora técnica de la instalación.
  4. Si la intervención de soporte afecta o manipula datos de negocio o registros de la aplicación, **dichas acciones también quedan registradas en la bitácora de negocio**.
  5. La intervención se realiza bajo principio de estricta minimización de datos personales y clínicos.

### 2.4 Entidad de Gobierno: Alta Dirección
* **Naturaleza en SIRACOM:** La Alta Dirección no es un rol operativo del día a día, sino una **entidad de gobernanza externa al flujo clínico y comercial**. Puede materializarse de dos formas configurables según la escala y naturaleza del cliente:
  1. *Como aprobación externa registrada como evidencia:* Acto administrativo formal (resolución de alcaldía, memorándum de gerencia municipal, disposición de la dirección médica) adjunto en formato digital como evidencia documental que respalda la acción sensible en la instalación.
  2. *Como cuenta institucional con privilegios exclusivos de autorización de gobierno:* Usuario institucional sin acceso operativo clínico ni de caja, con facultades exclusivas para autorizar la designación del Auditor (**PEND-15**), aprobar exportaciones masivas de la Jefatura y autorizar la activación de parámetros sensibles de la instalación.
* **Quién la ejerce:**
  - En una **municipalidad:** La Gerencia Municipal, Gerencia de Servicios a la Ciudad o la instancia jerárquica superior de la que depende Zoonosis/Bienestar Animal.
  - En una **organización pequeña o clínica privada:** El Director Médico, Propietario o Representante Legal de la empresa.
* **Cómo se registra su aprobación:** Mediante un flujo de validación formal en el sistema donde se vincula obligatoriamente el documento de respaldo (archivo digital o identificador de expediente administrativo), estampando en la bitácora de negocio la fecha, hora, identidad del autorizante y motivo del acto de gobierno.

---

## 3. Catálogo de Roles Base Estándar (8 Roles)

| Rol | Denominación | Propósito y Delimitación de Responsabilidades |
|---|---|---|
| **ROL-01** | **Administrador del Sistema Local (TI)** | **Rol estrictamente técnico de infraestructura.** Gestiona cuentas de usuarios, asigna roles previamente aprobados por Jefatura, configura sedes físicas, parámetros de red y conectividad, y monitorea respaldos y la bitácora técnica. **Por defecto no tiene acceso a datos clínicos de pacientes ni a datos personales de ciudadanos o responsables** (menor privilegio). No administra catálogos de negocio (farmacia, tarifas ni parámetros clínicos). |
| **ROL-02** | **Jefe / Supervisor de Área (Zoonosis / Bienestar)** | Supervisión estratégica y gobernanza: aprueba asignación de roles sensibles, aprueba solicitudes de anulación de animales con historial (`A`), autoriza ajustes/mermas de inventario, aprueba cierres de caja y emite reportes consolidados de gestión. No exporta la bitácora que registra sus propios actos; sus exportaciones masivas requieren autorización de Dirección General. Cuenta con suplente designado y modo organización pequeña según **PEND-16**. |
| **ROL-03** | **Médico Veterinario (Colegiado)** | Responsabilidad médica legal: en fase clínica realiza examen físico, diagnósticos, prescribe medicamentos, realiza cirugías y cierra formalmente la atención registrando su número de colegiatura (sujeto a **PEND-08**). Administra catálogos de parámetros clínicos (especies, razas, causales, vías). Actúa como garante sanitario en campañas masivas (sujeto a **PEND-13**). Puede solicitar formalmente la anulación justificada de animales (`S`). |
| **ROL-04** | **Técnico / Asistente Veterinario** | Soporte clínico operativo en Fase 1: toma de constantes vitales en triaje (peso, temperatura), sujeción física y datos preliminares en atención abierta. No emite diagnósticos, no prescribe ni edita actos quirúrgicos. Inocula vacunas y desparasitaciones bajo supervisión del Médico responsable registrado. |
| **ROL-05** | **Personal de Campo / Brigadista** | Operación en brigadas de campaña: registro rápido de animales y responsables en campo conectado e inoculación de biológicos bajo la supervisión del Médico Veterinario garante de campaña registrado. |
| **ROL-06** | **Recepcionista / Administrativo** | Atención al público y ventanilla: alta y actualización de datos de contacto de responsables y animales, gestión de agenda y citas, y derivación a consultorio. Puede registrar solicitudes formales de anulación justificada (`S`) dirigidas a Jefatura. |
| **ROL-07** | **Encargado de Farmacia / Almacén** | **Rol sensible.** Control de existencias físicas y costos: administración del catálogo de fármacos e insumos (por conocimiento especializado; sujeto a **PEND-14** para sustancias reguladas), recepción física de compras con lote, vencimiento y costo, transferencias, registro de movimientos en modo solo-agregar (contra-asientos), registro de entrega de dispensación física (`C-R`) y solicitud de ajustes/mermas (`S`). |
| **ROL-08** | **Cajero / Cobranzas** | **Rol sensible.** Gestión de ingresos en caja (módulo comercial activo): apertura y arqueo diario preliminar de caja, cobro de órdenes generadas por atención o mostrador y emisión de recibos internos de cobro. Las anulaciones de recibos requieren autorización de supervisor. Límites de caja y anulaciones son provisionales sujetos a **PEND-09**. |

---

## 4. Conjuntos de Permisos Configurables (Roles Propuestos, Pendientes de Decisión)

Los siguientes tres conjuntos de permisos no forman parte del estándar cerrado de roles y se catalogan formalmente como **"Propuestos, pendientes de decisión"** bajo **PEND-11**. Se detallan los permisos específicos que aportarían, justificando su asignación o tratamiento alternativo como responsabilidades asignables:

### 4.1 Rol 09: Auditor (Solo Lectura por Ámbito) — Propuesto, pendiente de decisión
* **Acceso por finalidad y ámbito autorizado:** El Auditor accede a los módulos y bitácoras exclusivamente dentro del marco de un mandato o comisión de auditoría formal.
* **Salvaguardas de seguridad y privacidad:**
  1. **Restricción de datos personales y clínicos:** Los datos identificables de ciudadanos se visualizan enmascarados/anonimizados salvo mandato expreso legal; la consulta de historias clínicas se limita a expedientes muestreados en el ámbito auditado.
  2. Totalmente independiente de la operación diaria (no registra, no edita, no anula ni aprueba actos).
  3. **Auditoría del auditor:** Sus consultas y descargas quedan registradas en el log de auditoría del sistema (requisito técnico de **PEND-07**).
  4. Las exportaciones masivas de datos exigen consignación obligatoria de motivo justificado en pantalla.
* **Mecanismo de aprobación independiente:** Para evitar conflictos de interés, la asignación y habilitación de este rol **no puede depender de la Jefatura de Área auditada**, sino de la Alta Dirección, Gerencia Municipal o Dirección Médica conforme a **PEND-15**.
* **Propuesta de tratamiento:** Mantenerlo como **conjunto de permisos configurable por instalación**, habilitado exclusivamente bajo mandato de control formal.

### 4.2 Rol 10: Coordinador de Campaña — Propuesto, pendiente de decisión
* **Permisos específicos:** Planificación de campañas masivas institucionales, designación de brigadas de campo, retiro de lotes de insumos desde almacén y liquidación del balance final de la jornada.
* **Delimitación estricta:** **No posee permisos clínicos.** No edita historiales clínicos, no inocula biológicos ni modifica registros médicos de vacunación (`U` clínico retirado).
* **Propuesta de tratamiento / fusión:** No se justifica como rol independiente fijo. Se propone tratarlo como una **responsabilidad asignable** a usuarios de nivel supervisor o médico durante la vigencia de una campaña, evitando dispersión en la administración de cuentas.

### 4.3 Rol 11: Encargado de Albergue / Canil — Propuesto, pendiente de decisión
* **Permisos específicos:**
  - *Creación de animales (`C`):* Justificado exclusivamente para el registro inicial de animales rescatados, en abandono o decomisados que ingresan a custodia sin registro previo en la instalación.
  - *Sin anulación directa (`D`):* No tiene permiso de borrado ni inactivación directa sobre animales; solo puede registrar solicitudes de baja o traslado (`S`) dirigidas a Jefatura.
  - *Datos de responsables limitados:* No administra el padrón de tenencia general; sus permisos sobre responsables se restringen a postulantes a adopción y donantes/denunciantes vinculados a expedientes de rescate.
  - *Delimitación clínica:* **No posee permisos de edición clínica** (`U` clínico retirado en triaje, constantes y vacunación). El estado médico y tratamientos de los animales alojados son registrados exclusivamente por el Médico Veterinario o Asistente clínico asignado. Albergue gestiona únicamente la bitácora conductual y alimentación no médica.
* **Propuesta de tratamiento:** Mantenerlo como **conjunto de permisos configurable por instalación**, habilitado únicamente en clientes con instalaciones de albergue físico. Si la entidad no cuenta con un cuidador exclusivo, las funciones de adopción se asignan al Recepcionista (ROL-06) y las de bitácora al Asistente Veterinario (ROL-04).

---

## 5. Matriz Preliminar de Permisos por Dominio Funcional y Rol

> **Aviso de alcance preliminar (Paso 4 del ciclo combinado ADR-009):**  
> La siguiente matriz de permisos es de carácter **preliminar**. Su delimitación final de campos y validaciones se consolidará y validará durante la redacción de las **fichas de proceso** (Paso 4), donde cada pantalla, estado y excepción de negocio quedará formalmente establecido.

### 5.1 Convención Formal de Acciones de Acceso
- **R (Read):** Ver / Consultar registros y pantallas.
- **C (Create):** Crear / Registrar nuevos registros.
- **U (Update):** Editar / Modificar registros abiertos o datos de contacto.
- **D (Disable/Annul):** Anular / Inactivar con motivo justificado obligatorio (sin borrado físico).
- **S (Solicitar):** Registrar una solicitud formal de anulación, ajuste o baja que requiere aprobación de un rol superior.
- **A (Approve):** Aprobar / Validar formalmente (cierre médico, actas de adopción, ajustes de stock, cierres de caja, anulaciones con historial).
- **X (Export):** Exportar reportes en formatos externos (Excel, PDF, CSV) con motivo obligatorio en descargas masivas.
- **M (Manage):** Administrar parámetros técnicos de configuración y cuentas del sistema.
- **- :** Sin acceso a la función o pantalla.

### 5.2 Matriz Granular de Autorización Preliminar

| Dominio Funcional / Submódulo | Admin TI (01) | Jefe Área (02) | Médico Vet. (03) | Asistente (04) | Brigada Campo (05) | Recepcionista (06) | Farmacia (07) | Cajero (08) | Auditor [Prop.] (09) | Coord. Camp. [Prop.] (10) | Enc. Albergue [Prop.] (11) |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Administración Técnica y Cuentas** | C-U-D-M | R-A (1) | - | - | - | - | - | - | R (2) | - | - |
| **Bitácora Técnica de Sistema** | R-X | R (3) | - | - | - | - | - | - | R-X (3) | - | - |
| **Bitácora de Negocio (Auditoría)** | - (4) | R-X (5) | - | - | - | - | - | - | R-X (6) | - | - |
| **Configuración Institucional (Técnica)** | C-U-M | R-U | R | R | R | R | R | - | R | R | R |
| **Parámetros Clínicos (Especies/Razas)**| - (7) | R-U-A | C-R-U-M | R | R | R | R | - | R | R | R |
| **Animales (Identificación)** | - (8) | R-U-D-A-X| C-R-U-S(9)| C-R-U | C-R | C-R-U-S(9)| R | R | R-X (6) | R-U | C-R-U-S(10)|
| **Responsables (Datos de Contacto)** | - (8) | R-U-D-X | C-R-U | C-R-U | C-R | C-R-U-D | R | R | R-X (6) | R-U | C-R-U (10)|
| **Citas y Agenda** | - | R-U-A | C-R-U | C-R-U | - | C-R-U-D | - | R | R | - | - |
| **Triaje y Constantes (Fase 1: Operativa)**| - | R | C-R-U | C-R-U(11)| - | - | - | - | R | - | R (12) |
| **Consulta y Diagnóstico (Fase 2-3)** | - | R-X | C-R-U-A(13)| R (11) | - | - | - | - | R-X (6) | - | R |
| **Prescripción de Medicamentos** | - | R | C-R-A(13)| R | - | - | R | - | R | - | - |
| **Cirugías y Procedimientos Médicos** | - | R-X | C-R-U-A(13)| R (11) | - | - | - | - | R-X | - | - |
| **Vacunación y Desparasitación** | - | R-X | C-R-A | C-R (14)| C-R (14)| - | R | - | R-X | R (12) | R (12) |
| **Planificación de Campañas** | - | C-R-U-D-A | R | R | R | - | R | - | R-X | C-R-U-A | - |
| **Registro Rápido en Campaña** | - | R-X | C-R | C-R | C-R (14)| - | R | - | R-X | C-R-X | - |
| **Catálogo de Farmacia e Insumos** | - (7) | R-U-A | R | R | R | - | C-R-U-D-M| - | R | R | R |
| **Órdenes de Compra (Administración)** | - | R-U-A-X | - | - | - | - | C-R-S-X | - | R-X | R | - |
| **Recepción Física de Mercadería** | - | R-X | - | - | - | - | C-R-X (15)| - | R-X | R | - |
| **Movimientos de Stock (Solo-agregar)** | - | R-X | - | - | - | - | C-R-X (16)| - | R-X | R-A | - |
| **Consumos Automáticos por Atención** | - | R-X | - | - | - | - | R-X | - | R-X | - | - |
| **Ajustes y Mermas de Inventario** | - | R-A (17)| - | - | - | - | C-R-S(17)| - | R-X | - | - |
| **Dispensación a Consultorio y Campaña**| - | R-A (18)| R-A (18)| R | - | - | C-R (18)| - | R | R-A | - |
| **Custodia en Albergue y Caniles** | - | R-U-A | R | R | - | - | - | - | R | - | C-R-U-A |
| **Bitácora Conductual y Alimentación** | - | R | R | R-U | - | - | - | - | R | - | C-R-U(19)|
| **Evaluación y Postulación Adopción** | - | R-U-A | R | - | - | R | - | - | R | - | C-R-U-A |
| **Seguimiento Post-Adopción** | - | R-X | R | - | - | R | - | - | R-X | - | C-R-U-X |
| **Catálogo de Tarifas y Servicios** | - (7) | C-R-U-M | R | - | - | R | - | R | R | - | - |
| **Órdenes de Cobro y Recibos Internos**| - | R-A-X(20)| R | - | - | C-R | - | C-R-S-X(20)| R-X | - | - |
| **Apertura, Cierre y Arqueo de Caja** | - | R-A-X(21)| - | - | - | - | - | C-R-X(21)| R-X | - | - |

**Reglas de seguridad y aclaraciones de la matriz:**
* **(1) Aprobación de roles sensibles:** La Jefatura de Área aprueba formalmente la activación de roles sensibles (01, 02, 03, 07, 08). Prohibida la autoasignación.
* **(2) Aprobación de Auditor:** El Auditor no es aprobado por Jefatura sino por Alta Dirección / Gerencia Municipal conforme a **PEND-15**.
* **(3) Consulta de bitácora técnica:** TI la consulta para monitoreo de red/servidor; Jefatura y Auditor la consultan para verificar caídas y accesos anómalos, sin contenido clínico.
* **(4) Bitácora de negocio restringida:** TI no accede a la bitácora de negocio por estricto menor privilegio y confidencialidad.
* **(5) Exportación de bitácora por Jefatura:** Jefatura no exporta la bitácora que registra sus propios actos; exportaciones masivas requieren autorización de Dirección General. Suplente y organizaciones pequeñas según **PEND-16**.
* **(6) Auditor por finalidad:** Acceso por mandato específico, datos identificables enmascarados salvo requerimiento legal, consultas auditadas y descargas masivas con motivo obligatorio.
* **(7) M retirado a TI:** Parámetros clínicos administrados por Médico Veterinario (03); farmacia por Encargado de Farmacia (07); tarifas por Jefatura (02). TI solo administra parámetros técnicos de red/servidor.
* **(8) Menor privilegio de TI:** Sin acceso a datos personales de ciudadanos ni a fichas de animales.
* **(9) Solicitud de anulación de animales:** Médico Veterinario y Recepcionista registran solicitud justificada (`S`); la aprobación (`A`) y ejecución lógica es exclusiva de Jefatura.
* **(10) Albergue - Ingreso y responsables limitados:** Albergue tiene `C` de animales para registrar ingresos de rescatados/abandonados; no tiene `D` directo (solo `S`); sobre responsables solo registra adoptantes y denunciantes vinculados.
* **(11) Asistente Veterinario en consulta:** Registra triaje, pesaje y constantes en fase 1; en fase 2 solo lectura (`R`); no emite diagnósticos ni prescribe fármacos ni edita cirugías.
* **(12) Retiro de edición clínica a no clínicos:** Coordinador de Campaña y Encargado de Albergue no editan datos clínicos (`U` clínico retirado en triaje, constantes y vacunas).
* **(13) Actos médicos exclusivos:** Diagnósticos, cirugías, recetas y cierre de atención son exclusivos del Médico Veterinario con colegiatura (sujeto a **PEND-08**). Una vez cerrada, la atención no admite edición en la aplicación (distinguiéndose de las garantías técnicas de bitácoras de **PEND-07**); rectificaciones por adendas auditadas conforme a la regla 1.4.
* **(14) Vacunación delegada:** Inoculación por Asistente o Brigadista registra al Médico colegiado responsable garante de campaña (sujeto a **PEND-13**).
* **(15) Recepción de mercadería:** Registro físico de ingreso al almacén con lote, vencimiento y costo de adquisición.
* **(16) Inventario solo-agregar:** Movimientos sin `U`. Correcciones por contra-asiento justificado (`C`). Consumos automáticos generados por el sistema tras actos clínicos cerrados o campañas.
* **(17) Ajustes y mermas:** Farmacia registra la solicitud (`C-R-S`); aprobación obligatoria por Jefatura (`A`), rol distinto al que registra.
* **(18) Dispensación física:** Farmacia registra la entrega física (`C-R`); autorización de despacho por Médico Veterinario o Jefatura (`A`).
* **(19) Bitácora de albergue:** Solo conducta y alimentación no médica. Tratamientos médicos registrados por personal veterinario.
* **(20) Recibos internos y anulación:** Cajero emite recibos internos; su anulación solo puede solicitarse (`S`), requiriendo aprobación de supervisor (`A`) hasta resolver **PEND-09**.
* **(21) Cierre de caja:** Cajero genera arqueo preliminar (`C`); aprobación obligatoria de cierre por supervisor (`A`).

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
* **PEND-05:** Política institucional de protección de datos personales, ciclo de vida y retención (Ley 29733).
* **PEND-07:** Requisitos de seguridad de bitácoras y garantías técnicas de aislamiento (ADR-001/ADR-002).
* **PEND-08:** Validez y mecanismo de formalización de cierre de actos médicos veterinarios y exigibilidad de colegiatura.
* **PEND-09:** Límites operativos de caja chica, arqueo y protocolo de anulación de cobros.
* **PEND-10:** Procedimiento formal de reasignación y transferencia de dueños de animales.
* **PEND-11:** Decisión formal sobre roles propuestos (Auditor, Coordinador de Campaña, Encargado de Albergue).
* **PEND-13:** Validez legal y condiciones de delegación de actos veterinarios en campañas y presencia exigida del Médico garante.
* **PEND-14:** Marco de control, custodia y dispensación de medicamentos veterinarios de uso regulado.
* **PEND-15:** Mecanismo institucional y autoridad competente para designación y aprobación del rol Auditor.
* **PEND-16:** Esquema de operación y asunción de riesgo para organizaciones pequeñas o consultorios unipersonales.
* **PEND-17:** Política técnica de autenticación, caducidad de contraseñas y doble factor (2FA).
