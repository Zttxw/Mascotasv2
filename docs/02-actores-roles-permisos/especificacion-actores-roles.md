# SIRACOM — Especificación de Actores, Roles y Permisos

**Documento:** Especificación Funcional de Actores, Roles y Permisos  
**Versión:** 0.1 (Borrador)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y aprobación por Jeanpier)  
**Ubicación en repositorio:** `docs/02-actores-roles-permisos/especificacion-actores-roles.md`  
**Referencia arquitectónica:** ADR-001, ADR-005, ADR-009 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))

---

## 1. Principios de Control de Acceso y Autorización

El modelo de control de acceso de SIRACOM se rige por los siguientes principios arquitectónicos:

1. **Principio de menor privilegio:** Todo usuario tiene acceso exclusivamente a las funciones, pantallas y datos indispensables para el desempeño de su labor operativa o clínica.
2. **Aislamiento por instalación (ADR-001):** Los usuarios, contraseñas, roles y permisos residen exclusivamente en la base de datos de cada cliente. No existen identificadores universales ni acceso administrativo cruzado entre distintas organizaciones.
3. **Modularidad y activación condicional (ADR-005):** La disponibilidad de roles y permisos se adapta a los módulos habilitados en la instalación. Si una municipalidad no activa el módulo comercial, las opciones de caja, cobros y ventas no se muestran ni se configuran. Si una clínica privada no opera albergues ni campañas masivas, dichos submódulos permanecen deshabilitados sin alterar la estabilidad del núcleo clínico.
4. **Trazabilidad y no repudio:** Toda operación sensible (modificación de historia clínica, anulación de actos médicos, aplicación de fármacos, ajustes de inventario y transacciones de caja) queda registrada en la bitácora de auditoría con identificación del usuario, marca de tiempo inmutable y detalle del cambio.

---

## 2. Clasificación de Actores del Sistema

```
                               ┌────────────────────────────────┐
                               │       ACTORES DE SIRACOM       │
                               └───────────────┬────────────────┘
                                               │
               ┌───────────────────────────────┴───────────────────────────────┐
               ▼                                                               ▼
   ┌───────────────────────┐                                       ┌───────────────────────┐
   │    ACTORES INTERNOS   │                                       │   SUJETOS REGISTRADOS │
   │ (Usuarios con acceso) │                                       │    (Sin credenciales) │
   └───────────┬───────────┘                                       └───────────┬───────────┘
               │                                                               │
  ┌────────────┴────────────┐                                                  │
  │ • Administrador TI      │                                                  ▼
  │ • Jefe de Zoonosis      │                                      ┌───────────────────────┐
  │ • Médico Veterinario    │                                      │ • Ciudadanos / Dueños │
  │ • Asistente Veterinario │                                      │ • Postulantes a       │
  │ • Brigadista de Campo   │                                      │   Adopción            │
  │ • Recepcionista         │                                      │ • Proveedores de      │
  │ • Encargado de Farmacia │                                      │   Insumos             │
  │ • Cajero                │                                      └───────────────────────┘
  │ • Auditor (Solo lectura)│
  │ • Coord. de Campaña     │
  │ • Encargado de Albergue │
  └─────────────────────────┘
```

### 2.1 Actores Internos (Usuarios Operativos)
Personal perteneciente a la municipalidad, institución o clínica veterinaria que interactúa con el sistema mediante credenciales de autenticación individuales y roles asignados.

### 2.2 Sujetos Registrados (Sin Acceso al Sistema)
Personas naturales o jurídicas cuya información es registrada y custodiada en el sistema, pero que **no cuentan con usuario ni contraseña** en esta fase:
* **Ciudadanos / Tenedores de mascotas:** Propietarios o poseedores de animales atendidos o empadronados.
* **Postulantes a adopción:** Personas evaluadas para la tenencia de un animal rescatado.
* **Proveedores de medicamentos e insumos:** Entidades comerciales registradas para trazabilidad de compras y abastecimiento.

---

## 3. Catálogo de Roles Estándar (8 Roles Base)

Los siguientes roles constituyen el estándar predeterminado de la plataforma y pueden asignarse a uno o más usuarios según la estructura del cliente:

| Rol | Denominación | Propósito Principal |
|---|---|---|
| **ROL-01** | **Administrador del Sistema Local (TI)** | Gestión técnica de la instalación: administración de cuentas de usuario, asignación de roles, configuración de sedes, parámetros generales y monitoreo de auditoría técnica. No interviene en actos médicos. |
| **ROL-02** | **Jefe / Supervisor de Área (Zoonosis / Bienestar)** | Supervisión estratégica: consulta de estadísticas globales, monitoreo de metas de campañas, aprobación de solicitudes especiales y generación de reportes consolidados de gestión. |
| **ROL-03** | **Médico Veterinario (Colegiado)** | Responsabilidad médica: emisión de diagnósticos, prescripción de recetas médicas, ejecución y registro de cirugías/procedimientos, y validación clínica de atenciones. |
| **ROL-04** | **Técnico / Asistente Veterinario** | Soporte clínico: pesaje, toma de constantes vitales en triaje, sujeción, aplicación de vacunas y desparasitaciones bajo prescripción/supervisión médica. |
| **ROL-05** | **Personal de Campo / Brigadista** | Operación en brigadas: registro ágil de animales y responsables durante campañas masivas descentralizadas, registrando la aplicación directa del biológico o insumo. |
| **ROL-06** | **Recepcionista / Administrativo** | Atención en ventanilla: alta y actualización de datos de responsables y animales, agendamiento de citas y derivación al consultorio o sala de espera. |
| **ROL-07** | **Encargado de Farmacia / Almacén** | Control de existencias: recepción de medicamentos e insumos, registro de lotes y fechas de vencimiento, transferencias entre almacenes/consultorios y ajustes de inventario. |
| **ROL-08** | **Cajero / Cobranzas** | Gestión de ingresos (cuando el módulo comercial está activo): apertura y cierre de caja, cobro de órdenes generadas por atención médica o mostrador y emisión de notas/recibos de venta. |

---

## 4. Evaluación de Roles Adicionales Solicitados

Conforme a las directrices de alcance, se analizan y formalizan tres roles complementarios para robustecer la gobernanza:

### 4.1 Rol 09: Auditor (Solo Lectura)
* **Evaluación y Justificación:** **Aprobado para inclusión.** En entornos municipales (Órgano de Control Institucional - OCI, inspectores de sanidad) y en clínicas con asesoría contable/legal externa, es indispensable otorgar acceso a expedientes, reportes y bitácoras de auditoría sin conferir privilegios de edición o borrado.
* **Alcance funcional:** Consulta global de historiales clínicos, movimientos de almacén, balances de campañas y registros de auditoría de usuarios. Botones de creación, edición, anulación o eliminación bloqueados en toda la interfaz.

### 4.2 Rol 10: Coordinador de Campaña
* **Evaluación y Justificación:** **Aprobado para inclusión.** Durante campañas masivas de zoonosis o esterilización en campo, la figura operativa que lidera no siempre es el Jefe de Área ni un médico en consulta. Este rol organiza la logística: define puntos de atención, brigadistas participantes, retira lotes de insumos del almacén y ejecuta el balance de cierre de la jornada.
* **Alcance funcional:** Planificación de campañas, asignación de metas, recepción de lotes de insumos para campo, supervisión del avance de digitación y emisión del acta de liquidación de campaña.

### 4.3 Rol 11: Encargado de Albergue / Canil
* **Evaluación y Justificación:** **Aprobado para inclusión.** La dinámica de un albergue o centro de rescate municipal difiere sustancialmente de la consulta externa. Exige control diario de ocupación de caniles, bitácoras de comportamiento, manejo de postulaciones de adopción y seguimiento domiciliario post-adopción.
* **Alcance funcional:** Registro de ingresos por rescate/abandono, asignación y liberación de espacios físicos (caniles/gateras), actualización de bitácoras de alimentación/conducta, registro de evaluaciones a adoptantes y cronograma de seguimiento post-adopción.

---

## 5. Matriz de Permisos por Dominio Funcional y Rol

**Convención de permisos:**
- **C** = Crear / Registrar
- **R** = Leer / Consultar
- **U** = Modificar / Actualizar
- **D** = Anular / Desactivar (con justificación auditada; no borrado físico)
- **E** = Ejecutar operación especializada (ej. autorizar cierre, recetar, dispensar)
- **-** = Sin acceso al módulo o acción

| Dominio Funcional | Admin TI (01) | Jefe Área (02) | Médico Vet. (03) | Asistente (04) | Brigada Campo (05) | Recepcionista (06) | Farmacia (07) | Cajero (08) | Auditor (09) | Coord. Campaña (10) | Enc. Albergue (11) |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Administración Local y Usuarios** | CRUD | R | - | - | - | - | - | - | R | - | - |
| **Auditoría del Sistema** | R | R | - | - | - | - | - | - | R | - | - |
| **Configuración Institucional / Catálogos** | CRUD | RU | R | R | R | R | R | - | R | R | R |
| **Animales y Responsables (Identificación)** | R | RU | CRU | CRU | CR | CRU | R | R | R | CRU | CRU |
| **Citas y Agenda** | - | RU | CRU | CRU | - | CRUD | - | R | R | - | - |
| **Triaje y Constantes Vitales** | - | R | CRU | CRU | - | - | - | - | R | - | RU |
| **Consulta y Diagnóstico Clínico** | - | R | CRU | R | - | - | - | - | R | - | R |
| **Prescripción de Medicamentos** | - | R | CRUE | R | - | - | R | - | R | - | - |
| **Cirugías y Procedimientos Médicos** | - | R | CRUE | RU | - | - | - | - | R | - | - |
| **Vacunación y Desparasitación** | - | R | CRU | CRU | CR | - | R | - | R | CRU | RU |
| **Planificación de Campañas** | - | CRUD | R | R | R | - | R | - | R | CRUE | - |
| **Registro Rápido en Campaña** | - | R | CR | CR | CR | - | R | - | R | CR | - |
| **Catálogo de Farmacia y Medicamentos** | R | R | R | R | R | - | CRUD | - | R | R | R |
| **Movimientos y Ajustes de Inventario** | - | R | - | - | - | - | CRUE | - | R | R | - |
| **Dispensación / Despacho a Consultorio** | - | R | R | R | - | - | CRUE | - | R | R | - |
| **Gestión de Albergue y Ocupación Caniles** | - | RU | R | R | - | - | - | - | R | - | CRUE |
| **Bitácora Conductual y Alimentación** | - | R | R | RU | - | - | - | - | R | - | CRU |
| **Evaluación y Registro de Adopciones** | - | RU | R | - | - | R | - | - | R | - | CRUE |
| **Seguimiento Post-Adopción** | - | R | R | - | - | R | - | - | R | - | CRU |
| **Catálogo de Tarifas y Servicios** | R | RU | R | - | - | R | - | R | R | - | - |
| **Órdenes de Cobro y Facturación** | - | R | R | - | - | CR | - | CRUE | R | - | - |
| **Apertura, Cierre y Arqueo de Caja** | - | R | - | - | - | - | - | CRUE | R | - | - |

*Nota sobre permisos:* El borrado físico de registros clínicos y transaccionales está técnicamente deshabilitado; toda anulación requiere justificación obligatoria y preserva el estado histórico como registro inactivo/anulado.

---

## 6. Actores y Capacidades Diferidas (ADR-003)

Para preservar la arquitectura sin introducir sobrecostos inmediatos, se establecen los siguientes lineamientos sobre actores diferidos:

1. **Rol Ciudadano / Tenedor Responsable (Portal Web Diferido):**
   - *Estado:* Diferido para fase posterior.
   - *Salvaguarda:* Los contratos de la API en el backend separarán los endpoints administrativos de los endpoints de consulta pública/propietario, permitiendo autenticar a ciudadanos mediante tokens de acceso restringidos cuando se construya el portal.
2. **Operador Móvil / Brigadista Offline (ADR-003):**
   - *Estado:* Diferido.
   - *Salvaguarda:* El modelo de datos incorpora identificadores generados por cliente (GUID), estampado de fecha de ocurrencia del hecho clínico y lógica de operaciones idempotentes para evitar duplicidades en sincronizaciones remotas futuras.

---

## 7. Sección de Pendientes y Decisiones Abiertas

Las siguientes materias quedan formalmente marcadas como **Pendientes** en este borrador:

| Identificador | Asunto Pendiente | Impacto en el Diseño de Seguridad | Responsable Propuesto |
|---|---|---|---|
| **PEND-ROL-01** | Política de autenticación: caducidad de contraseñas y doble factor (2FA) | Define si el 2FA es obligatorio para todos o solo para administradores y jefaturas. | Arquitectura Técnica / Seguridad |
| **PEND-ROL-02** | Validez y mecanismo de firma de actos médicos veterinarios | Determinar si se requiere firma digital formal (certificado digital) o aceptación bajo credencial interna en el expediente. | Asesoría Legal / Médico Veterinario Asesor |
| **PEND-ROL-03** | Límites y contingencias en el rol de cajero | Definir límites máximos de caja chica y procedimiento de anulación de comprobantes de cobro con intervención o no de supervisor. | Especialista Contable / Comercial |
| **PEND-ROL-04** | Criterio de reasignación y desvinculación de responsables de animales | Definir el flujo de validación formal cuando un animal cambia de dueño (cesión voluntaria, extravío o fallecimiento). | Analista Funcional / Asesoría Legal |
