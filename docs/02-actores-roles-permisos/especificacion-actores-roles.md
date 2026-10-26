# SIRACOM — Especificación de Actores, Roles y Permisos

**Documento:** Especificación Funcional de Actores, Roles y Permisos  
**Versión:** 0.1 (Borrador corregido)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y aprobación por Jeanpier)  
**Ubicación en repositorio:** `docs/02-actores-roles-permisos/especificacion-actores-roles.md`  
**Referencia arquitectónica:** ADR-001, ADR-005, ADR-009 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))

---

## 1. Principios de Control de Acceso y Autorización

El modelo de seguridad y control de acceso de SIRACOM se rige por los siguientes principios arquitectónicos:

1. **Principio de menor privilegio y privacidad de datos:** Todo usuario accede exclusivamente a las funciones, pantallas y datos indispensables para su función. En particular, el personal técnico de TI no tiene acceso por defecto a datos médicos ni personales de ciudadanos.
2. **Aislamiento e independencia local (ADR-001):** Usuarios, credenciales, roles y permisos residen exclusivamente en la base de datos de cada cliente. No existen identificadores globales ni superadministradores centrales.
3. **Modularidad y activación condicional (ADR-005):** Los roles y permisos se habilitan de acuerdo con los módulos activos. Si una municipalidad no activa el módulo comercial, las opciones de caja, cobros y ventas no se muestran ni se configuran. Si una clínica privada no gestiona albergues ni campañas masivas, dichos submódulos permanecen deshabilitados sin afectar el núcleo clínico.
4. **Trazabilidad de actos clínicos y registros cerrados:** Todo acto clínico, diagnóstico, cirugía o prescripción registra de forma obligatoria al Médico Veterinario actuante (incluyendo su número de colegiatura profesional). Los registros clínicos formalmente cerrados **no admiten edición ni sobrescritura**; cualquier rectificación o aclaración posterior se efectúa exclusivamente mediante adendas médicas fechadas y auditadas que preservan íntegro el texto original.
5. **Inmutabilidad y anulación justificada:** Queda técnicamente inhabilitado el borrado físico de datos. La anulación de registros (que cubre de forma expresa **animales y responsables**, así como actos clínicos, inventario y cobros) exige consignar un motivo obligatorio y conserva el historial completo con el usuario y marca de tiempo.
6. **Auditoría inmutable de accesos y operaciones:** Toda operación sensible y consulta de fiscalización queda registrada en la bitácora de auditoría inmutable de la instalación.

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
│ (Usuarios de la org.) │           │    (Sin credenciales) │               │       (ADR-001)       │
└───────────┬───────────┘           └───────────┬───────────┘               └───────────┬───────────┘
            │                                   │                                       │
            │                                   ▼                                       ▼
            │                       ┌───────────────────────┐               ┌───────────────────────┐
            │                       │ • Ciudadanos / Dueños │               │ • Soporte Técnico del │
            │                       │ • Postulantes Adopción│               │   Proveedor (temporal,│
            │                       │ • Proveedores Insumos │               │   autorizado y        │
            │                       └───────────────────────┘               │   auditado)           │
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
│ ROLES PROPUESTOS (PENDIENTES DE DECISIÓN):    │
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
* **Proveedores de medicamentos e insumos:** Entidades comerciales registradas para trazabilidad de compras de inventario.

### 2.3 Actor Excepcional: Soporte Técnico del Proveedor (ADR-001)
* **Naturaleza excepcional:** El desarrollador o proveedor de SIRACOM **no tiene acceso permanente, implícito ni remoto** a la instalación ni a la base de datos de producción de ningún cliente.
* **Protocolo de intervención:** En caso de soporte correctivo o mantenimiento que no pueda resolverse por el Administrador TI local:
  1. Debe mediar una solicitud formal y autorización expresa de la jefatura del cliente.
  2. El Administrador TI local habilita una cuenta temporal de soporte con expiración automática de vigencia.
  3. Cada acción, consulta y comando ejecutado por soporte queda registrado de forma inalterable en la bitácora de auditoría de la instalación.
  4. La intervención se realiza bajo principio de estricta minimización de datos personales y clínicos.

---

## 3. Catálogo de Roles Base Estándar (8 Roles)

| Rol | Denominación | Propósito y Límites de Acceso |
|---|---|---|
| **ROL-01** | **Administrador del Sistema Local (TI)** | **Rol estrictamente técnico.** Gestiona cuentas de usuarios, asigna roles aprobados, configura sedes, parámetros de conectividad y monitorea respaldos y auditoría técnica. **Por defecto no tiene acceso a datos clínicos de pacientes ni a datos personales de responsables o ciudadanos** (menor privilegio). No interviene en actos médicos. |
| **ROL-02** | **Jefe / Supervisor de Área (Zoonosis / Bienestar)** | Supervisión y gestión global: visualiza métricas consolidadas, metas de campañas, reportes de gestión y autoriza excepciones operativas dentro de su área. |
| **ROL-03** | **Médico Veterinario (Colegiado)** | Responsabilidad médica legal: emite diagnósticos, prescribe medicamentos, realiza y registra cirugías y procedimientos clínicos, y valida atenciones cerradas con su número de colegiatura. |
| **ROL-04** | **Técnico / Asistente Veterinario** | Soporte clínico operativo: toma de constantes vitales en triaje (peso, temperatura), sujeción y aplicación de vacunas y desparasitaciones bajo indicación médica. |
| **ROL-05** | **Personal de Campo / Brigadista** | Operación ágil en campañas masivas: registro rápido en campo de animales y responsables y aplicación del biológico/insumo asignado. |
| **ROL-06** | **Recepcionista / Administrativo** | Atención en mostrador y ventanilla: registro y actualización de datos de contacto de responsables y animales, gestión de agenda y citas, y derivación a consultorio. |
| **ROL-07** | **Encargado de Farmacia / Almacén** | Control de existencias físicas: recepción de pedidos, registro de lotes y vencimientos, dispensación a consultorios o brigadas y registro de mermas y ajustes. |
| **ROL-08** | **Cajero / Cobranzas** | Gestión de ingresos (cuando el módulo comercial está habilitado): apertura y cierre de caja, cobro de órdenes generadas por atención o mostrador y emisión de notas/recibos de venta. |

---

## 4. Evaluación de Roles Propuestos (Pendientes de Decisión)

Los siguientes tres roles no forman parte del núcleo cerrado y se catalogan formalmente como **"Propuestos, pendientes de decisión"**. Para cada uno se analiza qué permisos exclusivos aportaría frente a los roles base y se plantea una alternativa de fusión o tratamiento como responsabilidad asignable:

### 4.1 Rol 09: Auditor (Solo Lectura) — Propuesto, pendiente de decisión
* **Permisos exclusivos que ningún otro rol tiene:** Acceso de lectura transversal y exportación a **todos** los módulos del sistema (historias clínicas, almacén, campañas, caja y registros de usuarios) sin permiso de creación, modificación ni anulación en ninguno de ellos. Además, incorpora la regla de seguridad de que **sus propias consultas y exportaciones quedan registradas en el log de auditoría del sistema** (auditoría del auditor), garantizando que no consulte expedientes sin dejar rastro.
* **Independencia operativa:** Totalmente desvinculado de la operación diaria. No puede registrar ni aprobar actos.
* **Propuesta de tratamiento / fusión:** Si la entidad es una clínica pequeña o una municipalidad sin requerimiento de fiscalización independiente (OCI), no se justifica crear un usuario con este rol permanente. Se propone mantenerlo como **rol configurable opcional**, activable únicamente cuando se requiera conceder acceso temporal a inspectores sanitarios o auditores externos.

### 4.2 Rol 10: Coordinador de Campaña — Propuesto, pendiente de decisión
* **Permisos específicos:** Planificación de campañas institucionales, asignación de brigadistas, retiro de lotes de insumos desde almacén para campo y liquidación del balance final de la jornada.
* **Análisis de exclusividad frente a roles existentes:** No posee permisos que no puedan ser ejercidos por el **Jefe de Área (ROL-02)** o un **Médico Veterinario (ROL-03)** con funciones de campo. Su creación como rol independiente solo se justificaría en municipalidades con gran volumen de campañas simultáneas que deleguen la jefatura operativa en un coordinador no médico.
* **Propuesta de tratamiento / fusión:** Se propone **no crearlo como rol independiente fijo**, sino tratar la "Coordinación de Campaña" como una **responsabilidad asignable** a usuarios con rol de Jefe de Área o Médico Veterinario, evitando dispersión y sobrecarga en la administración de perfiles.

### 4.3 Rol 11: Encargado de Albergue / Canil — Propuesto, pendiente de decisión
* **Permisos específicos:** Asignación física y control de ocupación de caniles/gateras, registro de bitácoras diarias de comportamiento y alimentación (no médicas), y recepción de postulaciones de adopción.
* **Análisis de exclusividad frente a roles existentes:** En organizaciones sin instalaciones de custodia física (clínicas privadas o municipalidades sin canil municipal), este rol es completamente irrelevante. En municipalidades con albergue, las tareas de custodia y bitácora no son médicas ni administrativas generales, justificando un perfil diferenciado de cuidador/evaluador.
* **Propuesta de tratamiento / fusión:** Se propone mantenerlo como **rol modular configurable**, habilitado únicamente en instalaciones que activen el submódulo de albergue físico. Si la entidad no cuenta con personal exclusivo para el canil, sus funciones de adopción se asignan al **Recepcionista / Administrativo (ROL-06)** y las de bitácora y bienestar al **Asistente Veterinario (ROL-04)**.

---

## 5. Matriz de Permisos por Dominio Funcional y Rol

### 5.1 Convención Formal de Acciones de Acceso
Para eliminar ambigüedades, se adopta una nomenclatura estricta de 7 acciones:
- **R (Read):** Ver / Consultar registros y pantallas.
- **C (Create):** Crear / Registrar nuevos registros.
- **U (Update):** Editar / Modificar registros abiertos o datos de contacto.
- **D (Disable/Annul):** Anular / Desactivar con motivo justificado obligatorio (sin borrado físico).
- **A (Approve):** Aprobar / Validar formalmente (cierre de historias clínicas, actas de adopción, balances de campaña).
- **X (Export):** Exportar reportes en formatos externos (Excel, PDF, CSV).
- **M (Manage):** Administrar parámetros de configuración, cuentas y permisos del sistema.
- **- :** Sin acceso a la función o pantalla.

### 5.2 Matriz Granular de Autorización

| Dominio Funcional | Admin TI (01) | Jefe Área (02) | Médico Vet. (03) | Asistente (04) | Brigada Campo (05) | Recepcionista (06) | Farmacia (07) | Cajero (08) | Auditor [Prop.] (09) | Coord. Camp. [Prop.] (10) | Enc. Albergue [Prop.] (11) |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Administración Técnica y Usuarios** | C-U-D-M | R | - | - | - | - | - | - | R | - | - |
| **Auditoría Técnica del Sistema** | R-X | R-X | - | - | - | - | - | - | R-X (*) | - | - |
| **Configuración Institucional y Parámetros** | C-U-M | R-U | R | R | R | R | R | - | R | R | R |
| **Animales y Responsables (Identificación)** | - (**) | R-U-X | C-R-U-D | C-R-U | C-R | C-R-U-D | R | R | R-X | R-U | C-R-U-D |
| **Citas y Agenda** | - | R-U-A | C-R-U | C-R-U | - | C-R-U-D | - | R | R | - | - |
| **Triaje y Constantes Vitales** | - | R | C-R-U | C-R-U | - | - | - | - | R | - | R-U |
| **Consulta y Diagnóstico Clínico** | - | R-X | C-R-U-A | R | - | - | - | - | R-X | - | R |
| **Prescripción de Medicamentos** | - | R | C-R-A | R | - | - | R | - | R | - | - |
| **Cirugías y Procedimientos Médicos** | - | R-X | C-R-U-A | R-U | - | - | - | - | R-X | - | - |
| **Vacunación y Desparasitación** | - | R-X | C-R-A | C-R-U | C-R | - | R | - | R-X | R-U | R-U |
| **Planificación de Campañas** | - | C-R-U-D-A | R | R | R | - | R | - | R-X | C-R-U-A | - |
| **Registro Rápido en Campaña** | - | R-X | C-R | C-R | C-R | - | R | - | R-X | C-R-X | - |
| **Catálogo de Farmacia e Insumos** | R-M | R | R | R | R | - | C-R-U-D | - | R | R | R |
| **Movimientos y Ajustes de Inventario** | - | R-X | - | - | - | - | C-R-U-D-A | - | R-X | R-U | - |
| **Dispensación a Consultorio y Campaña** | - | R | R | R | - | - | C-R-U-A | - | R | R-A | - |
| **Custodia en Albergue y Caniles** | - | R-U-A | R | R | - | - | - | - | R | - | C-R-U-D-A |
| **Bitácora Conductual y Alimentación** | - | R | R | R-U | - | - | - | - | R | - | C-R-U |
| **Evaluación y Postulación de Adopción** | - | R-U-A | R | - | - | R | - | - | R | - | C-R-U-A |
| **Seguimiento Post-Adopción** | - | R-X | R | - | - | R | - | - | R-X | - | C-R-U-X |
| **Catálogo de Tarifas y Servicios** | R-M | R-U | R | - | - | R | - | R | R | - | - |
| **Órdenes de Cobro y Comprobantes** | - | R-X | R | - | - | C-R | - | C-R-D-X | R-X | - | - |
| **Apertura, Cierre y Arqueo de Caja** | - | R-A-X | - | - | - | - | - | C-R-A-X | R-X | - | - |

*Aclaraciones de seguridad:*  
(*) **Auditor (ROL-09):** Sus consultas y descargas quedan registradas en la bitácora inmutable de la instalación.  
(**) **Administrador TI (ROL-01):** No tiene acceso a datos personales de ciudadanos ni a expedientes médicos por menor privilegio. En caso de requerir soporte sobre base de datos, actúa bajo el protocolo del actor de soporte temporal.  
(***) **Anulaciones (D):** Requieren motivo obligatorio registrado; no existe borrado físico en animales, responsables, actos médicos ni inventario.

---

## 6. Actores y Capacidades Diferidas (ADR-003)

1. **Rol Ciudadano / Tenedor Responsable (Portal Autoservicio Diferido):**
   - *Estado:* Diferido para fase posterior.
   - *Salvaguarda:* La API en el backend separará los endpoints de consumo interno administrativo de los endpoints de consulta de dueños de mascotas, asegurando autenticación restringida sin exponer datos de terceros cuando se construya el portal.
2. **Operador Móvil / Brigadista Offline (ADR-003):**
   - *Estado:* Diferido.
   - *Salvaguarda:* El modelo de datos incorpora identificadores generados por cliente (GUID), estampado de fecha de ocurrencia del hecho clínico y lógica de operaciones idempotentes para evitar duplicidades en sincronizaciones remotas futuras.

---

## 7. Sección de Pendientes con Condiciones de Cierre Explícitas

| Identificador | Asunto Pendiente | Impacto en el Diseño de Seguridad | Condición de Cierre Explícita | Responsable Propuesto |
|---|---|---|---|---|
| **PEND-ROL-01** | Política de autenticación: caducidad de contraseñas y doble factor (2FA) | Define si el 2FA es obligatorio para todos o solo para administradores y supervisores de área. | Aprobación de la política técnica de seguridad que defina longitud mínima de clave, rotación y exigencia de 2FA. | Arquitectura Técnica / Seguridad |
| **PEND-ROL-02** | Validez y mecanismo de firma de actos médicos veterinarios | Determinar si se requiere firma digital formal (certificado digital) o aceptación bajo credencial interna en el expediente. | Informe de asesoría legal sobre validez de historia clínica veterinaria digital y pronunciamiento del especialista médico. | Asesoría Legal / Médico Veterinario Asesor |
| **PEND-ROL-03** | Límites y contingencias en el rol de cajero | Definir límites máximos de caja chica y procedimiento de anulación de comprobantes de cobro con intervención o no de supervisor. | Definición de los procedimientos de control interno contable por el área administrativa o comercial del cliente. | Especialista Contable / Comercial |
| **PEND-ROL-04** | Protocolo de cambio y reasignación de dueños de animales | Definir el flujo de validación formal cuando un animal cambia de responsable (cesión voluntaria, extravío o fallecimiento del tenedor). | Elaboración del procedimiento funcional de transferencia de custodia y validación de requisitos de identidad. | Analista Funcional / Asesoría Legal |
| **PEND-ROL-05** | Decisión formal sobre los roles propuestos (Auditor, Coordinador de Campaña, Encargado de Albergue) | Define si se consolidan como roles independientes o si Coordinador de Campaña se trata como responsabilidad asignable y Albergue como rol modular condicional. | Decisión formal y documentada por Jeanpier antes de cerrar el Catálogo de Módulos (Etapa 3 del ciclo combinado). | Jeanpier / Producto |
