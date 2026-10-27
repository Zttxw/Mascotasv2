# SIRACOM — Catálogo de Módulos, Límites y Dependencias

**Documento:** Especificación Funcional de Módulos, Límites y Dependencias del Sistema  
**Versión:** 0.1 (Borrador corregido)  
**Fecha:** 26 de octubre de 2026  
**Estado:** **Borrador** (Pendiente de revisión y validación por Jeanpier)  
**Ubicación en repositorio:** `docs/03-catalogo-de-modulos/especificacion-catalogo-modulos.md`  
**Referencia arquitectónica:** ADR-001, ADR-005, ADR-009, ADR-010 ([ADR-SIRACOM.md](../00-decisiones/ADR-SIRACOM.md))  
**Registro de decisiones pendientes:** [Registro Unificado de Pendientes](../00-decisiones/pendientes.md)

---

## 1. Principios de Modularidad y Reglas de Acoplamiento

El diseño modular de SIRACOM se fundamenta en los principios arquitectónicos derivados del ADR-001 (instalaciones independientes), ADR-005 (desacoplamiento comercial-clínico) y ADR-010 (decisiones de producto confirmadas):

1. **Núcleo de Datos Maestro Permanente (Padrón Animal y Responsables):** El registro y la identificación de animales y sus responsables constituyen el núcleo central del sistema. **Siempre se encuentra activo** y no admite desactivación en ninguna instalación. Todos los módulos operativos (clínica, campañas, albergue, caja) se anclan a este padrón común.
2. **Dependencias Unidireccionales Estrictas:** Ningún módulo de gestión de soporte o comercial puede condicionar, bloquear ni interrumpir el funcionamiento de los módulos sustantivos o clínicos. En particular:
   - El módulo clínico depende del padrón de animales y del componente canónico de actos sanitarios, pero **jamás depende de tarifas, cobros, facturación ni caja** (ADR-005).
   - El módulo de campañas masivas depende del padrón de animales y del componente canónico de actos sanitarios, pero opera con o sin control de existencias de farmacia.
   - El módulo comercial consume hechos de órdenes de cobro generadas por la atención clínica, mostrador o campañas privadas, pero la caída de caja o la desactivación de tarifas no impide brindar ni registrar actos veterinarios o sanitarios.
3. **Separación de Capas: Catálogos Maestros, Identidad del Producto e Inventario:**
   - *Catálogos maestros comunes:* Especies, razas, causales comunes de anulación y vías de administración, administrados por el dominio médico-veterinario o administrativo funcional (mantenidos por Médico Veterinario ROL-03 o personal autorizado). No residen en la plataforma técnica SRV-01.
   - *Identidad mínima del producto sanitario:* Atributos indispensables (nombre normalizado, tipo de biológico/fármaco y especie de destino) para registrar y auditar un acto sanitario con trazabilidad consistente, aun cuando Farmacia (MOD-05) esté desactivada.
   - *Inventario propietario (MOD-05):* Gestión de existencias físicas, almacenes, lotes disponibles, caducidades, movimientos kardex, validación de stock y mermas.
   - Las opciones de diseño para producto y lote con MOD-05 inactivo se detallan en la Sección 1.1 y se gestionan bajo **PEND-30**.
4. **Activación Modular Configurable por Cliente:** Cada instalación independiente activa únicamente los módulos contratados y necesarios para su modelo de operación (gestión municipal, clínica veterinaria privada o albergue/ONG), ocultando de la interfaz y deshabilitando de la API las funcionalidades no autorizadas o innecesarias.
5. **Regla General de Escritura entre Módulos:** Un módulo solo puede modificar datos propiedad de otro módulo mediante un servicio o una operación que el módulo propietario exponga expresamente. Ningún módulo escribe directamente en estructuras internas o tablas propietarias de otro módulo. Esta regla aplica expresamente a MOD-06 (Albergue/Adopciones) frente a MOD-01 (Padrón Animal): cuando se formaliza una adopción, MOD-06 no actualiza por su cuenta los registros de titularidad del animal, sino que invoca la operación formal de cambio de titularidad por adopción expuesta por MOD-01 (`TransferirTitularidadPorAdopcion`), correspondiendo a MOD-01 validar los requisitos, registrar al nuevo tenedor y preservar la integridad del historial de tenencia.

> [!NOTE]
> **Relación con el Mecanismo de Activación y Licenciamiento (PEND-20):**  
> El presente catálogo define la frontera lógica, las dependencias y los contratos funcionales entre módulos. La especificación técnica del mecanismo mediante el cual se activan o bloquean estos módulos en cada instalación (archivos de licencia criptográfica local, banderas de configuración en base de datos o variables de entorno del contenedor Docker, preservando la autonomía local sin validaciones remotas que comprometan la disponibilidad) queda diferida y sujeta a resolución bajo el ítem **PEND-20** en la Etapa 8.

---

### 1.1 Evaluación de Opciones de Diseño: Identidad Mínima del Producto Sanitario y Lote sin Farmacia (MOD-05)

Cuando una instalación opera sin el módulo de Farmacia (MOD-05), los actos sanitarios (vacunaciones, desparasitaciones) deben registrar qué biológico se aplicó y con qué lote, sin simular inventario ni permitir que texto libre caótico degrade las estadísticas de salud pública (ej. vacunación antirrábica distrital). Se evalúan dos opciones técnicas:

* **Opción A: Catálogo Maestro de Nomenclatura Sanitaria (Desacoplado de Stock):**
  - *Descripción:* Catálogo predefinido y normalizado de biológicos y fármacos comunes (nombre genérico/comercial, tipo de biológico, especie destino). El operador selecciona el producto del catálogo y digita el número de lote rotulado en el frasco físico.
  - *Ventajas:* Nombres estandarizados; alta calidad para reportes y vigilancia epidemiológica (MINSA/DIRIS); integración limpia si en el futuro se activa MOD-05 (el inventario vincula sus ítems al producto maestro).
  - *Desventajas:* Sobrecarga inicial de configuración del catálogo para entidades pequeñas sin personal técnico.
  - *Duplicación potencial:* Mínima, al existir una única tabla de definiciones de producto.
  - *Complejidad de integración:* Media.
* **Opción B: Parámetros Predefinidos Restringidos con Captura de Lote al Vuelo:**
  - *Descripción:* Lista cerrada de tipos de procedimiento (ej. "Antirrábica Canina", "Séxtuple Canina", "Triple Felina") donde el operador ingresa laboratorio y lote en campos de texto estructurado restringido.
  - *Ventajas:* Despliegue inmediato sin configuración previa de catálogo de productos; máxima agilidad para brigadas de campo.
  - *Desventajas:* Menor granularidad en marcas comerciales; si se activa MOD-05 posteriormente, requiere conciliación manual para asociar registros pasados al stock.
  - *Duplicación potencial:* Media en cadenas de texto descriptivo.
  - *Calidad estadística:* Aceptable para metas globales de dosis por tipo de vacuna; deficiente si la autoridad sanitaria exige trazabilidad por laboratorio o retiro de lote defectuoso.
  - *Complejidad de integración:* Baja al inicio, alta al conciliar con inventario.

* **Reglas rectoras para ambas opciones:**
  1. Un texto libre sin normalizar no puede ser la única fuente para estadísticas sanitarias oficiales (ej. vacunación antirrábica).
  2. Con MOD-05 desactivado, el sistema **no simula disponibilidad ni afirma haber validado el lote contra un inventario inexistente**. El lote se registra exclusivamente como metadato de trazabilidad sanitaria proporcionado por el operador actuante.
  3. La selección final entre Opción A y B queda registrada como decisión abierta bajo **PEND-30**.

---

## 2. Propuesta Arquitectónica: Componente Canónico de Actos Sanitarios (CAS)

### 2.1 Naturaleza y Propósito (Propuesta bajo PEND-29)
Se propone la creación del **Componente Canónico de Actos Sanitarios (CAS)** como el repositorio canónico de eventos preventivos y procedimientos médicos (vacunaciones, desparasitaciones, cirugías/esterilizaciones, colocación de microchips).
- Una intervención aplicada a un animal posee un identificador global único (`IdActoSanitario`) y un origen explícito (`Origen`: "Consultorio" | "Campana").
- Una vacunación realizada en campaña masiva **no puede duplicarse ni fragmentarse** en dos registros clínicos por haber sido digitada en campo o consultada posteriormente en consultorio.
- Este componente permite que el módulo de Campañas (MOD-04) registre actos sanitarios oficiales aun cuando la Gestión Médica Veterinaria fija (MOD-03) se encuentre desactivada (caso típico de perfiles ONG o brigadas municipales de campo sin consultorio clínico).

### 2.2 Responsabilidad, Límites y Relación con MOD-01
- *Responsabilidad:* Custodiar la cronología inalterable de actos de intervención corporal o sanitaria sobre el animal.
- *Relación con MOD-01:* MOD-01 posee la identidad biográfica del animal (`IdAnimal`). La historia clínica o carné de salud del animal consulta a CAS mediante dicha referencia para exponer la línea de tiempo consolidada.
- *Límites:* CAS no almacena triajes completos por sistemas, diagnósticos patológicos complejos ni evoluciones médicas hospitalarias (propiedad de MOD-03). Tampoco administra caniles (MOD-06) ni existencias físicas (MOD-05).

### 2.3 Datos Mínimos Registrados con MOD-03 Desactivado
Cuando MOD-03 está desactivado, CAS registra como mínimo:
1. `IdActoSanitario` (GUID único canónico)
2. `IdAnimal` (referencia a MOD-01)
3. `Origen` ("Campana" | "PuestoPreventivo")
4. `FechaHoraOcurrencia`
5. `OperadorActuante` (identidad de quien inoculó: ROL-05 brigadista o ROL-04 asistente)
6. `VeterinarioResponsable` (identidad y colegiatura del Médico Veterinario colegiado garante, conforme a PEND-13)
7. `TipoProcedimiento` (ej. Vacunación Antirrábica, Desparasitación Interna, Implante de Microchip)
8. `IdentidadProducto` (nombre/tipo normalizado según PEND-30)
9. `LoteTexto` (código de lote del frasco)

### 2.4 Alternativas Descartadas y Justificación
- *Alternativa descartada 1: Duplicar registros en tablas separadas de MOD-03 y MOD-04.* Descartada porque fractura el historial sanitario del animal, duplica carnés y genera inconsistencias estadísticas ante fiscalizaciones de salud pública.
- *Alternativa descartada 2: Embeber el registro de actos exclusivamente dentro de MOD-03 (Clínica).* Descartada porque violaría la modularidad: obligaría a organizaciones tipo ONG o campañas municipales a activar y pagar/configurar todo el módulo clínico para registrar una jornada de vacunación.
- *Alternativa descartada 3: Ubicarlo dentro de SRV-01 (Plataforma).* Descartada porque SRV-01 es estrictamente infraestructura y seguridad; no debe incorporar conceptos de dosificación ni colegiatura veterinaria.
- *Alternativa descartada 4: Asignarlo por inercia al núcleo MOD-01 (Padrón Animal).* Descartada porque MOD-01 es el maestro de identidad demográfica del animal y su tenedor, no un motor de transacciones médicas operativas.

### 2.5 Dependencias del Componente
- MOD-03 y MOD-04 **dependen de CAS** para registrar y consultar eventos preventivos.
- CAS **no depende de MOD-03**. Opera sobre SRV-01 y MOD-01.
- *Estado:* Propuesta formalizada y sujeta a validación institucional bajo **PEND-29**.

---

## 3. Decisión Arquitectónica: Citas y Agenda (¿Módulo Propio o Integrado a Clínica?)

### 3.1 Decisión Adoptada
**Se define "Citas y Agenda de Turnos" como un Módulo Funcional Propio e Independiente (MOD-02), articulado pero desacoplado de la Gestión Médica Veterinaria (MOD-03).**

### 3.2 Justificación
1. **Diferenciación de actores y ciclo de vida:** El agendamiento de turnos, gestión de cancelaciones y recepción en sala de espera es operado por el **Recepcionista / Administrativo (ROL-06)**, quien no requiere privilegios clínicos.
2. **Transversalidad a múltiples dominios:** La agenda atiende consultas médicas, cirugías programadas, turnos de empadronamiento masivo y entrevistas de evaluación para adopción en albergue (MOD-06).
3. **Degradación operativa limpia:** Si una institución atiende por demanda espontánea por orden de llegada, MOD-02 se desactiva sin perturbar el registro de atenciones clínicas en MOD-03. A la inversa, una oficina de registro o albergue puede agendar citas sin necesidad de activar historias clínicas.

---

## 4. Matriz General de Dependencias y Hechos Publicados entre Módulos

La siguiente tabla establece formalmente los contratos de acoplamiento del sistema, divididos en cuatro columnas estrictas: dependencias obligatorias, dependencias opcionales, hechos que el módulo publica para consumo de otros, y dependencias prohibidas (ADR-005):

| Código | Módulo Funcional | (1) Dependencias Obligatorias (Exige) | (2) Dependencias Opcionales (Si están activos) | (3) Hechos que Publica (Consumibles por otros módulos) | (4) Dependencias Prohibidas (Nunca puede depender de) |
|---|---|---|---|---|---|
| **SRV-01** | **Servicios de Plataforma y Seguridad** | Ninguna (Es la base) | Ninguna | `UsuarioAutenticado`, `RolAsignado`, `AlertaTecnicaGenerada`, `EvidenciaRegistrada` | Cualquier módulo operativo de negocio (MOD-01 a MOD-08) |
| **MOD-01** | **Padrón de Animales y Responsables** | SRV-01 | Ninguna (El núcleo maestro no conoce módulos opcionales) | `AnimalRegistrado`, `FichaAnimalActualizada`, `ResponsableRegistrado`, `TitularidadTransferida`, `AnimalInactivadoLogicamente` | MOD-02, MOD-03, MOD-04, MOD-05, MOD-06, MOD-07, MOD-08 |
| **CAS** | **Componente Canónico de Actos Sanitarios (Prop.)** | SRV-01, MOD-01 | MOD-05 (para enlazar lote a inventario si está activo) | `ActoSanitarioRegistrado`, `ProcedimientoPreventivoAplicado` | MOD-03 (debe operar si clínica está inactiva), MOD-07 |
| **MOD-02** | **Citas y Agenda de Turnos** | SRV-01, MOD-01 | MOD-03 (asociar cita a servicio médico), MOD-06 (asociar a entrevista adopción) | `CitaProgramada`, `CitaConfirmada`, `PacienteEnSalaEspera`, `CitaAtendida`, `CitaCancelada` | MOD-05 (Farmacia), MOD-07 (Comercial / Caja) |
| **MOD-03** | **Gestión Médica Veterinaria (Clínica y Cirugía)** | SRV-01, MOD-01, CAS | MOD-02 (asociar atención a cita previa) | `AtencionIniciada`, `TriajeRegistrado`, `ActoClinicoCerrado` (detalla insumos y servicios para consumo externo), `AdendaClinicaRegistrada` | **MOD-07 (Comercial / Tarifas / Caja)**, MOD-05 (Farmacia para registrar el acto médico), MOD-06, MOD-08 |
| **MOD-04** | **Campañas Institucionales y Brigadas de Campo** | SRV-01, MOD-01, CAS | Ninguna obligatoria para operar | `CampanaIniciada`, `InoculacionRegistradaEnCampo`, `CampanaLiquidada` (con balance de insumos y recaudación si aplica) | MOD-07 (Comercial no condiciona campañas de salud pública), MOD-05 (no condiciona el registro en campo), MOD-08 |
| **MOD-05** | **Farmacia y Control Físico de Existencias** | SRV-01 | Consume hechos de MOD-03 (`ActoClinicoCerrado`), MOD-04 (`CampanaLiquidada`) y MOD-07 (`VentaMostradorCompletada`) | `StockActualizado`, `LoteRegistrado`, `AlertaStockCritico`, `AjusteInventarioAprobado`, `InsumoDespachadoABrigada` | MOD-02, MOD-03, MOD-04, MOD-06 (Farmacia no condiciona actos médicos ni rescates) |
| **MOD-06** | **Albergue, Custodia Temporal y Adopciones** | SRV-01, MOD-01 | MOD-02 (citas de entrevista), MOD-03 (altas sanitarias si está activo), MOD-05 (salidas de alimento si está activo) | `AnimalIngresadoAlbergue`, `CustodiaActualizada`, `AdopcionFormalizada` (solicita transferencia a MOD-01), `SeguimientoPostAdopcionRegistrado` | MOD-04 (Campañas), MOD-07 (Comercial no condiciona rescates ni adopciones) |
| **MOD-07** | **Gestión Comercial Configurable (POS / Caja)** | SRV-01 | MOD-01 (si asocia cobro a animal/dueño), MOD-05 (si vende ítems de stock). Consume hechos de MOD-03 y MOD-04 para cobros | `OrdenCobroEmitida`, `ReciboCobroGenerado`, `CierreCajaAprobado`, `ReciboAnulado` | **Ningún módulo clínico, preventivo ni de rescate (MOD-03, MOD-04, MOD-06) puede depender de MOD-07** |
| **MOD-08** | **Reportes Consolidados e Indicadores (Solo Lectura)** | SRV-01 | Consume en modo solo lectura datos agregados de los módulos activos (MOD-01 a MOD-07) | Ninguno (Es consumidor final de solo lectura) | Ningún módulo transaccional (MOD-01 a MOD-07) puede depender de MOD-08 |

*Nota sobre el mecanismo técnico:* No se impone un mecanismo técnico de comunicación (colas asíncronas, eventos en memoria o llamadas a servicios internos); la definición técnica de integración se mantiene abierta conforme a ADR-005.

---

## 5. Ciclo de Vida de los Módulos: Activación, Desactivación y Reactivación

El comportamiento operativo del sistema ante cambios de estado en los módulos se rige por las siguientes reglas arquitectónicas:

1. **Desactivación Funcional No Destructiva:** Desactivar un módulo **jamás elimina datos de la base de datos**. Toda la información transaccional e histórica permanece preservada en el esquema.
2. **Disponibilidad Restringida durante la Desactivación:**
   - Las opciones de menú, formularios de captura y endpoints operativos de escritura del módulo desactivado se ocultan de la interfaz y se bloquean en la API.
   - Las consultas históricas sobre registros de dicho módulo se restringen a perfiles autorizados en modo de solo lectura, según reglas de acceso a definir en el diseño técnico.
3. **Reactivación e Integridad Referencial:** Al reactivarse un módulo, los datos históricos recuperan su plena operatividad, respetando la integridad referencial y las asignaciones de roles vigentes.
4. **Regla de Bloqueo de Desactivación por Dependencia:** El sistema **impide la desactivación** de un módulo si dicha acción deja insatisfecha una dependencia obligatoria de otro módulo que permanezca activo en la instalación.
   - *Ejemplo:* MOD-01 (Padrón Animal) no puede ser desactivado bajo ninguna circunstancia mientras existan módulos de negocio activos.
   - *Ejemplo:* CAS no puede desactivarse si MOD-03 o MOD-04 están activos.
5. **Validación Previa e Identificación de Procesos en Curso:** Antes de ejecutar la desactivación de un módulo, el sistema analiza dependencias y alerta sobre procesos abiertos (ej. atenciones médicas sin cerrar en MOD-03, campañas sin balance final en MOD-04 o cajas sin cierre diario en MOD-07), exigiendo su conclusión o resolución formal previa.
6. **Comportamiento en Reportes Consolidados (MOD-08):** Los reportes históricos preservan las estadísticas correspondientes a los periodos en que el módulo estuvo activo, indicando claramente la fecha de cese de actividad para evitar interpretaciones erróneas de "ausencia de datos".
7. *Estado:* Criterios registrados como base funcional; las reglas de transición detalladas se gestionan bajo **PEND-36**.

---

## 6. Perfiles de Activación Modular por Tipo de Organización

SIRACOM provee combinaciones modulares predefinidas y verificadas para agilizar el despliegue según el modelo institucional del cliente:

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              PERFILES PRECONFIGURADOS DE INSTALACIÓN                                   │
├──────────────────────────────────────┬─────────────────────────────────────────────────────────────────┤
│ Perfil de Instalación                │ Módulos Activos por Defecto y Opciones                          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **1. Municipalidad**                 │ • **Activos por defecto:** MOD-01 (Padrón), CAS (Actos Sanit.), │
│    *(Cliente de referencia           │   MOD-02 (Agenda), MOD-03 (Clínica/Preventiva), MOD-04 (Campa- │
│     confirmado - ADR-010)*           │   ñas de Campo), MOD-05 (Farmacia) y MOD-06 (Albergue).         │
│                                      │ • **Opcional / Configurable:** MOD-07 (Comercial/Caja) según si │
│                                      │   la ordenanza cobra tasas o derechos, a verificar con la muni- │
│                                      │   cipalidad (PEND-34). MOD-08 (Reportes) activo para vigilancia.│
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **2. Clínica Veterinaria Privada**   │ • **Activos por defecto:** MOD-01 (Padrón), CAS (Actos Sanit.), │
│                                      │   MOD-02 (Agenda), MOD-03 (Clínica y Cirugía), MOD-05 (Farma-   │
│                                      │   cia y Compras), MOD-07 (Comercial y Caja) y MOD-08 (Reportes).│
│                                      │ • **Desactivados por defecto:** MOD-06 (Albergue).              │
│                                      │ • **Opcional activable:** MOD-04 (Campañas privadas con cobro o │
│                                      │   jornadas a costo social, desactivadas inicialmente).          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ **3. Albergue / Organización ONG**   │ • **Activos por defecto:** MOD-01 (Padrón), CAS (Actos Sanit.), │
│                                      │   MOD-02 (Agenda visitas), MOD-06 (Custodia y Adopciones),      │
│                                      │   MOD-05 (Botiquín básico) y MOD-08 (Reportes de rescate).      │
│                                      │ • **Desactivados por defecto:** MOD-04 (Campañas masivas) y     │
│                                      │   MOD-07 (Comercial). MOD-03 (Clínica) activable si cuenta con  │
│                                      │   tópico veterinario propio (si MOD-03 está inactivo, opera con │
│                                      │   ficha visual de ingreso en MOD-06 según PEND-32).             │
└──────────────────────────────────────┴─────────────────────────────────────────────────────────────────┘
```

---

## 7. Especificación Detallada de Módulos del Sistema

---

### MÓDULO 00: Servicios Transversales de Plataforma y Seguridad (SRV-01)
* **Correspondencia arquitectónica:** Corresponde directamente al **Dominio 1 (Plataforma y Administración Local)** de la Especificación de Visión y al **Dominio A (Plataforma y administración local)** de la Sección 1.1 del [ADR-SIRACOM](../00-decisiones/ADR-SIRACOM.md).
* **Propósito:** Proveer la infraestructura base técnica de autenticación, control de accesos, auditoría estricta segregada, almacenamiento seguro de documentos de gobierno, notificaciones técnicas y parametrización de marca desacoplada (ADR-010), manteniéndose completamente libre de lógica de negocio o catálogos médicos.
* **Submódulos:**
  1. *Configuración Institucional y Sedes:* Parámetros generales de la entidad cliente, sedes físicas y zonas horarias.
  2. *Gestión Técnica de Usuarios y Sesiones:* Altas, bajas, control de contraseñas y sesiones activas.
  3. *Gobernanza Técnica de Accesos (RBAC):* Asignación de los 8 roles base estándar y activación técnica de perfiles configurables (Perfil 09, Perfil 10, Perfil 11), sin autoasignación. Soporte de 2FA conforme a **PEND-17**.
  4. *Bitácora Técnica Segregada:* Registro de eventos de servidor, accesos de red y caídas, consultable por TI, Jefatura y Auditor sin acceso a datos clínicos ni personales.
  5. *Servicio de Bitácora de Negocio:* Servicio técnico de persistencia inalterable en base de datos (cuyo registro no admite edición en la aplicación, distinguiéndose de las garantías técnicas de bitácoras de **PEND-07**) para eventos de dominio emitidos por los módulos de negocio (inaccesible para TI).
  6. *Custodia de Evidencias y Archivos Digitales:* Almacenamiento blob/archivos seguro para evidencias de aprobación de Alta Dirección, expedientes y actas.
  7. *Notificaciones Técnicas y Alertas del Sistema:* Servicio transversal para avisos internos de infraestructura y eventos técnicos de sistema.
  8. *Monitoreo Técnico, Respaldos y Recuperación:* Diagnóstico de estado del servidor y ejecución técnica de copias de seguridad de volúmenes Docker y PostgreSQL.
  9. *Desacoplamiento de Marca Institucional:* Parámetro centralizado de denominación del producto conforme a ADR-010.
* **Actores participantes:** Administrador TI Local (ROL-01), Jefe de Área (ROL-02), Auditor (Perfil 09), Alta Dirección (Gobernanza).
* **Datos que posee (Fuente de verdad):** Cuentas de usuario, credenciales cifradas, sesiones activas, asignaciones de roles, registros de logs técnicos y de negocio, configuración de marca y archivos adjuntos.
* **Datos que consume:** Ninguno (es la base del sistema).
* **Dependencias:** Ninguna.
* **Comportamiento cuando está desactivado:** **No admite desactivación.** Es el núcleo técnico operativo indispensable para la ejecución del contenedor.

---

### MÓDULO 01: Padrón de Animales y Responsables (MOD-01)
* **Propósito:** Gestionar de forma única, auditable y centralizada la identidad, características biométricas, señas particulares, historial de vida y datos de contacto de los tenedores y animales intervenidos por la institución.
* **Submódulos:**
  1. *Identificación Animal:* Ficha biográfica (especie, raza, sexo, edad estimada o fecha de nacimiento, color, pelaje, señas particulares, fotografía), código de microchip validado (PEND-21) y estado vital (vivo, fallecido).
  2. *Padrón de Responsables / Tenedores:* Datos personales de ciudadanos tenedores (documento de identidad, nombres, domicilio, teléfonos, correo), consentimiento informado de datos personales y registro de derechos ARCO conforme a **PEND-05**.
  3. *Trazabilidad de Custodia y Vínculos:* Historial de transferencias de titularidad y reasignaciones voluntarias conforme a **PEND-10**. Expone la operación formal `TransferirTitularidadPorAdopcion` para consumo exclusivo de MOD-06.
* **Actores participantes:** Recepcionista / Administrativo (ROL-06), Médico Veterinario (ROL-03), Técnico Asistente (ROL-04), Brigadista de Campo (ROL-05), Jefe de Área (ROL-02), Encargado de Albergue (Perfil 11).
* **Datos que posee (Fuente de verdad):** Identificador único del animal (GUID), número de microchip/placa, ficha de vida animal, datos del tenedor responsable, vínculos jurídicos de tenencia y consentimiento de protección de datos personales.
* **Datos que consume:** Catálogo de especies y razas mantenido a nivel funcional.
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
* **Datos que consume:** Ficha del animal y responsable de MOD-01; identidad de profesionales de SRV-01.
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La institución atiende exclusivamente por demanda espontánea. Los pacientes que llegan a ventanilla pasan de forma directa al registro de triaje o consulta en MOD-03 sin reserva previa de turno.

---

### MÓDULO 03: Gestión Médica Veterinaria (MOD-03)
* **Propósito:** Registrar cronológicamente el historial clínico de los animales atendidos en consultorio o quirófano fijo (cuyo registro cerrado no admite edición en la aplicación, distinguiéndose de las garantías técnicas de **PEND-07**), garantizando el flujo profesional en 4 fases, la colegiatura del profesional actuante y la trazabilidad de prescripciones farmacológicas y quirúrgicas.
* **Submódulos:**
  1. *Triaje Preliminar y Constantes Vitales (Fase 1: Operativa):* Pesaje, temperatura, frecuencia cardíaca/respiratoria, tiempo de llenado capilar y condición corporal (registrado por Asistente ROL-04 o Médico ROL-03).
  2. *Consulta Médica, Diagnósticos y Evolución (Fase 2: Validación Profesional):* Anamnesis, exploración física por sistemas, diagnósticos presuntivos y definitivos, y planes terapéuticos exclusivos del Médico Veterinario (ROL-03).
  3. *Prescripción y Emisión de Recetas:* Emisión estructurada de recetas médicas con denominación de fármaco, posología, vía y duración del tratamiento.
  4. *Procedimientos Preventivos y Registro Canónico:* Registra vacunaciones y desparasitaciones sobre el Componente Canónico de Actos Sanitarios (CAS), calculando fechas sugeridas de revacunación.
  5. *Actos Quirúrgicos y Hospitalización:* Protocolos pre-quirúrgicos, consentimientos informados de anestesia, detalle de intervención quirúrgica (esterilizaciones, cirugías generales) e incidencias.
  6. *Cierre Clínico y Adendas Médicas (Fases 3 y 4):* Marcado de cierre en la aplicación con firma/colegiatura profesional (PEND-08), momento en el cual el registro no admite edición en la aplicación, publicando el evento `ActoClinicoCerrado`. Registro de adendas médicas posteriores fechadas por el médico tratante o titular sustituto justificado (PEND-27).
* **Actores participantes:** Médico Veterinario colegiado (ROL-03, profesional exclusivo), Técnico / Asistente Veterinario (ROL-04, apoyo operativo en Fase 1), Jefe de Área (ROL-02, supervisión y suplencia).
* **Datos que posee (Fuente de verdad):** Episodios clínicos, constantes de triaje, diagnósticos, recetas emitidas, protocolos quirúrgicos, estado de cierre de atención y adendas médicas auditadas.
* **Datos que consume:** Identidad de animal y responsable de MOD-01; citas de MOD-02 (opcional); registra actos en CAS.
* **Dependencias:** SRV-01, MOD-01 y CAS.
* **Comportamiento cuando está desactivado:** La instalación no registra historias clínicas ni atención médica individualizada.

---

### MÓDULO 04: Campañas Institucionales y Brigadas de Campo (MOD-04)
* **Propósito:** Planificar, desplegar y ejecutar intervenciones masivas de salud pública en campo (vacunación antirrábica masiva, desparasitación canina/felina, esterilizaciones comunitarias itinerantes y empadronamiento barrial) mediante interfaces optimizadas para digitación ágil en campo conectado (ADR-003).
* **Submódulos:**
  1. *Planificación y Asignación de Brigadas:* Definición de campaña (nombre, meta sanitaria, fechas, polígono territorial/barrio), designación del Médico Veterinario garante (PEND-13) y acreditación de brigadistas de campo (ROL-05).
  2. *Asignación y Retiro de Insumos de Campaña:* Registro de dotación de biológicos y microchips despachados hacia la brigada, con prevención estricta de autoaprobación por parte del Coordinador de Campaña conforme a **PEND-28**.
  3. *Registro Rápido de Campo Conectado:* Captura ágil y en un solo paso del animal, tenedor y acto sanitario sobre el Componente Canónico CAS (con soporte de contingencia ante caídas de señal bajo fichas de digitación diferida según **PEND-06**).
  4. *Liquidación, Balance y Cierre de Campaña:* Cuadre de dosis aplicadas frente a frascos devueltos intactos o mermas/roturas en campo corregidas por contra-asiento justificado con aprobación de supervisor, publicando el evento `CampanaLiquidada`.
* **Actores participantes:** Personal de Campo / Brigadista (ROL-05), Coordinador de Campaña (Perfil 10, asignado a ROL-02 o ROL-03), Médico Veterinario Garante (ROL-03), Jefe de Área (ROL-02).
* **Datos que posee (Fuente de verdad):** Registro maestro de campañas, cronograma territorial, actas de apertura y cierre de jornada, registros rápidos de intervención en campo, balance de insumos asignados vs. aplicados.
* **Datos que consume:** Padrón animal y responsables de MOD-01; registra actos en CAS; consume dotación de MOD-05 si Farmacia está activa (si MOD-05 está inactivo, opera registrando actos en CAS sin validar existencias numéricas).
* **Dependencias:** SRV-01, MOD-01 y CAS.
* **Comportamiento cuando está desactivado:** La instalación no gestiona brigadas de campo ni operativos sanitarios masivos. Las vacunaciones o cirugías solo pueden realizarse dentro del consultorio fijo en MOD-03.

---

### MÓDULO 05: Farmacia, Abastecimiento y Control de Existencias (MOD-05)
* **Propósito:** Controlar rigurosamente los movimientos físicos, saldos disponibles, caducidades y consumos de medicamentos, vacunas, anestésicos, material médico y microchips, garantizando el principio de movimientos solo-agregar (contra-asientos) y trazabilidad por lote y vencimiento (FEFO).
* **Submódulos:**
  1. *Control de Almacenes Físicos y Saldos:* Gestión multialmacén (Almacén Central, Botiquín de Consultorio, Custodia Temporal de Brigada de Campaña) y kardex valorizado.
  2. *Movimientos de Inventario (Solo-agregar):* Ingresos físicos, traslados entre sub-almacenes y correcciones mediante contra-asientos justificados (sin edición destructiva ni borrado de transacciones).
  3. *Consumos Automáticos por Acto Clínico o Campaña:* Descuento directo de stock gatillado de forma automática por el sistema al consumir los eventos `ActoClinicoCerrado` de MOD-03 o `CampanaLiquidada` de MOD-04.
  4. *Gestión de Ajustes, Vencimientos y Mermas:* Registro de mermas físicas, roturas y caducidades con flujo obligatorio de aprobación por rol independiente (Jefatura de Área ROL-02).
  5. *Abastecimiento, Proveedores y Compras con Costos (Submódulo Activable):* Catálogo de proveedores registrados, registro de órdenes de compra institucionales y recepción física de mercadería con registro obligatorio de costos de adquisición unitarios y totales.
* **Actores participantes:** Encargado de Farmacia / Almacén (ROL-07), Jefe de Área (ROL-02, aprobación de mermas y compras), Médico Veterinario (ROL-03, solicitud de despacho a consultorio), Coordinador de Campaña (Perfil 10, recepción de dotación).
* **Datos que posee (Fuente de verdad):** Catálogo de inventario, ubicaciones físicas, kardex de transacciones de stock, saldos por lote y fecha de vencimiento, órdenes de compra y costos históricos de adquisición.
* **Datos que consume:** Eventos `ActoClinicoCerrado` de MOD-03 (para bajas de insumos usados); eventos `CampanaLiquidada` de MOD-04 (para descargas de dotación); eventos `VentaMostradorCompletada` de MOD-07 (si se venden ítems de stock).
* **Dependencias:** SRV-01.
* **Comportamiento cuando está desactivado:** El sistema opera sin control cuantitativo de existencias físicas en almacén. En CAS, MOD-03 y MOD-04 se continúa registrando el nombre del producto, dosis y número de lote aplicado al animal (garantizando la trazabilidad sanitaria), pero el sistema no valida saldos ni genera descuentos automáticos de inventario.

---

### MÓDULO 06: Albergue, Custodia Temporal y Adopciones (MOD-06)
* **Propósito:** Administrar integralmente el ingreso, alojamiento físico o temporal, bienestar no médico y formalización de adopciones responsables de animales rescatados, en abandono o decomisados institucionalmente. Se estructura en dos submódulos funcionalmente separables:
* **Submódulos:**
  1. *Submódulo 6.1: Custodia Física, Alojamiento y Hogares Temporales:*
     - Registro de ingreso y procedencia (abandono en vía pública, rescate por maltrato, decomiso, entrega voluntaria).
     - *Ficha básica de recepción y estado físico aparente al ingreso:* En instalaciones con MOD-03 inactivo (perfil ONG), se registra una evaluación visual no médica (condición corporal aparente, pelaje, lesiones visibles, aseo), sin diagnósticos veterinarios reservados a ROL-03, conforme a **PEND-32**.
     - Control de ocupación de espacios físicos (caniles individuales, gateras colectivas, pabellones de cuarentena).
     - *Modalidad de Hogares Temporales Externos (Propuesta PEND-33):* Registro y supervisión de familias de acogida que custodian al animal de forma externa antes de su adopción definitiva.
     - *Bitácora diaria de estancia y alimentación:* Registro no clínico de paseos, socialización y pauta de alimentación en texto estructurado. Si MOD-05 está activo, la salida de alimento puede vincularse opcionalmente a un consumo de stock; si MOD-05 está inactivo, opera como bitácora operativa sin afectar kardex (**PEND-35**).
  2. *Submódulo 6.2: Formalización de Adopciones Responsables:*
     - Registro y empadronamiento de postulantes a adopción.
     - Ficha de evaluación socio-habitacional y verificación de aptitud del adoptante.
     - Emisión y suscripción del Acta de Compromiso de Tenencia Responsable.
     - Solicitud formal de cambio de titularidad invocando la operación `TransferirTitularidadPorAdopcion` expuesta por MOD-01, preservando la propiedad de datos de MOD-01.
     - Planificación y registro de visitas o llamadas de fiscalización y seguimiento post-adopción con registro fotográfico.
* **Actores participantes:** Encargado de Albergue / Canil (Perfil 11), Jefe de Área (ROL-02, autorización de bajas/adopciones), Recepcionista (ROL-06, atención de postulantes en caso de no existir Perfil 11), Asistente Veterinario (ROL-04, apoyo en bitácora).
* **Datos que posee (Fuente de verdad):** Registro de ingresos a custodia, asignación de caniles u hogares temporales, bitácoras conductuales y alimentarias, postulaciones a adopción, actas contractuales de compromiso y evidencias de visitas post-adopción.
* **Datos que consume:** Identidad del animal y ficha de vida de MOD-01; atenciones clínicas y altas sanitarias veterinarias de MOD-03 (si está activo); agenda de citas para entrevistas de MOD-02 (opcional); insumos de alimento de MOD-05 (opcional).
* **Dependencias:** SRV-01 y MOD-01.
* **Comportamiento cuando está desactivado:** La instalación no gestiona custodia física ni procesos institucionales de rescate y adopción. Los animales registrados en MOD-01 corresponden exclusivamente a mascotas particulares con tenedor conocido.

---

### MÓDULO 07: Gestión Comercial Configurable (MOD-07)
* **Propósito:** Proveer capacidades de punto de venta (POS), cobro de tarifas de servicios veterinarios, venta de mostrador de farmacia/accesorios y arqueo diario de caja en establecimientos comerciales o dependencias públicas que cobren tasas o derechos (a verificar con la municipalidad bajo PEND-34), operando con total desacoplamiento del flujo clínico (ADR-005).
* **Submódulos:**
  1. *Catálogo de Tarifas y Servicios:* Configuración de precios, tasas o conceptos de cobro por atenciones, vacunas, cirugías, desparasitaciones o certificados.
  2. *Órdenes de cobro y recibos internos:* Generación de recibos internos o notas de cobro vinculadas a hechos consumidos de MOD-03 (`ActoClinicoCerrado`) o MOD-04 (`CampanaLiquidada` en campañas privadas con cobro). No constituyen comprobantes con validez tributaria formal (**PEND-03**).
  3. *Venta de Mostrador Desacoplada (POS Directo):* Emisión de órdenes de cobro para venta directa de productos o servicios sueltos sin necesidad de que exista una atención clínica ni un animal registrado (con asociación opcional al animal/responsable si se encuentra en ventanilla). Si vende ítems de stock, se integra con MOD-05; si MOD-05 no está activo, vende conceptos o servicios tarifados.
  4. *Arqueo y Cierre Diario de Caja:* Registro de turno, fondo inicial de caja chica, registro de pagos por medio (efectivo, POS/tarjeta, transferencias bancarias), solicitud de arqueo preliminar y aprobación obligatoria de cierre por supervisor (ROL-02).
  5. *Protocolo de Anulación Controlada:* Solicitud obligatoria justificada de anulación de recibos internos (`S`), requiriendo aprobación formal de supervisor (`A`) bajo **PEND-09**.
* **Actores participantes:** Cajero / Cobranzas (ROL-08), Jefe / Supervisor de Área (ROL-02, aprobación obligatoria de arqueo, cierre y anulaciones).
* **Datos que posee (Fuente de verdad):** Catálogo de tarifas vigentes, órdenes de cobro, recibos internos emitidos, cierres de turno de caja, cuadre de medios de pago y solicitudes/aprobaciones de anulación.
* **Datos que consume:** Consume hechos de actos clínicos cerrados de MOD-03 y liquidaciones de campaña de MOD-04 (consumo no bloqueante de solo lectura para liquidación); catálogo de existencias de MOD-05 (si está activo para ventas con baja de stock de mostrador).
* **Dependencias:** SRV-01. Integración opcional con MOD-01 (si se asocia a un animal), MOD-03 (si cobra una consulta) y MOD-05 (si descuenta existencias de farmacia al vender).
* **Comportamiento cuando está desactivado:** Todos los servicios institucionales se brindan sin registro de cobro ni gestión de caja. Ninguna pantalla del sistema solicita medios de pago ni muestra botones de liquidación comercial.

---

### MÓDULO 08: Reportes Consolidados e Indicadores de Gestión (MOD-08)
* **Propósito:** Consolidar, sintetizar y presentar reportes estadísticos, tableros de control e indicadores de gestión epidemiológica, cobertura de salud pública y rendimiento operativo, operando de forma 100% desacoplada en modo de solo lectura.
* **Mapeo Explícito de Indicadores y Módulos de Origen:**
  1. *Vigilancia Epidemiológica y Zoonosis:*
     - Vacunación antirrábica distrital: Originada en los registros del Componente Canónico de Actos Sanitarios (CAS), ejecutados vía MOD-04 (campañas) o MOD-03 (consultorio).
     - Accidentes por mordedura: [Pendiente de decisión sobre submódulo de vigilancia de mordeduras / accidentes por animales (PEND-31)]. No existe capacidad disponible asumida sin fuente.
     - Casos de esterilización masiva: Originada en actos quirúrgicos registrados en CAS vía MOD-04 o MOD-03.
  2. *Métricas de Campañas Masivas:*
     - Cumplimiento de metas sanitarias por barrio y rendimiento de brigadas: Originada en las liquidaciones y cierres de jornada de MOD-04.
  3. *Indicadores de Bienestar y Albergue:*
     - Tiempos promedio de permanencia en custodia: Originada en registros de ingreso y egreso de MOD-06 (submódulo 6.1).
     - Tasa de adopción de animales rescatados: Originada en actas de adopción formalizadas en MOD-06 (submódulo 6.2).
     - Tasa de retorno de extraviados y animales hallados: [Pendiente de decisión sobre submódulo de reporte de extravíos y hallazgos (PEND-31)].
     - Porcentaje de efectividad en seguimiento post-adopción: Originada en fichas de visitas de fiscalización post-adopción de MOD-06 (submódulo 6.2).
  4. *Métricas Operativas y Financieras:*
     - Resumen consolidado de ingresos por tipo de servicio y recaudación de caja: Originada en cierres de turno de caja aprobados de MOD-07.
     - Rotación de fármacos, valorización de inventario y pérdidas por mermas/vencimientos: Originada en el kardex y ajustes aprobados de MOD-05.
* **Regla para Métricas sin Datos Suficientes:** Sin datos suficientes, MOD-08 muestra "Sin datos suficientes" (o equivalente), nunca un valor cero ("0") que aparente ausencia comprobada del fenómeno.
* **Naturaleza de la Memoria de Cálculo y Protección de Datos:** La memoria de cálculo consiste en consultas agregadas, vistas materializadas o tablas analíticas de resumen que conservan datos anonimizados o métricas cuantitativas consolidadas, sin almacenar datos personales identificables (DNI, nombres, teléfonos o domicilios de ciudadanos quedan expresamente excluidos de los reportes consolidados para salvaguardar la Ley 29733).
* **Actores participantes:** Jefe de Área (ROL-02), Auditor (Perfil 09), Administrador TI (ROL-01, solo métricas técnicas), Alta Dirección (Gobernanza institucional).
* **Datos que posee (Fuente de verdad):** No genera datos transaccionales propios. Almacena definiciones de plantillas de reporte e históricos consolidados en memoria de cálculo.
* **Datos que consume:** Lee de forma no bloqueante datos de MOD-01, CAS, MOD-03, MOD-04, MOD-05, MOD-06 y MOD-07 (exclusivamente de aquellos módulos que se encuentren activados en la instalación).
* **Dependencias:** SRV-01.
* **Comportamiento cuando está desactivado:** Cada módulo individual conserva y sigue generando sus reportes operativos inmediatos (listados, fichas individuales y resúmenes diarios). Lo que se desactiva es la vista analítica consolidada y los tableros de control gerenciales y epidemiológicos agregados.

---

## 8. Remisión al Registro Unificado de Pendientes

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
* **PEND-25:** Alcance y gobernanza de los "parámetros sensibles" autorizados por la Alta Dirección (condición de cierre restaurada: a resolver antes de cerrar el Catálogo de Módulos).
* **PEND-27:** Asignación de médico titular de reemplazo para adendas clínicas en consultorios unipersonales para MOD-03.
* **PEND-28:** Prevención de autoaprobación del Coordinador de Campaña en el retiro de insumos para MOD-04 y MOD-05.
* **PEND-29:** Registro canónico único de actos preventivos y procedimientos sanitarios, componente propietario y delimitación frente a MOD-01/MOD-03/MOD-04.
* **PEND-30:** Mecanismo de gestión de identidad mínima de producto sanitario y lote ante la ausencia del módulo de Farmacia (MOD-05).
* **PEND-31:** Definición funcional y asignación de módulo de origen para accidentes por mordedura, denuncias de extravío y hallazgo de animales.
* **PEND-32:** Alcance del registro de estado de salud y triaje visual inicial de ingreso a albergue en instalaciones con MOD-03 desactivado (perfil ONG).
* **PEND-33:** Definición funcional y operativa de la modalidad de custodia en Hogares Temporales Externos dentro de MOD-06.
* **PEND-34:** Verificación del cobro de tasas o derechos por servicios en la municipalidad de referencia, base legal y procedimiento administrativo.
* **PEND-35:** Mecanismo de registro de alimentación en albergue e integración condicional con salidas de stock de MOD-05.
* **PEND-36:** Reglas detalladas de transición de ciclo de vida de módulos (desactivación, reactivación, integridad de datos históricos y procesos en curso).
