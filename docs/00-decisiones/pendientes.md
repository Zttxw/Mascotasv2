# SIRACOM — Registro Unificado de Decisiones Pendientes

**Versión del documento:** 1.0 (Borrador)  
**Fecha de actualización:** 26 de octubre de 2026  
**Ubicación en el repositorio:** `docs/00-decisiones/pendientes.md`  
**Referencia arquitectónica:** ADR-001 a ADR-009 ([ADR-SIRACOM.md](ADR-SIRACOM.md))  
**Estado:** Activo y vinculante para el diseño funcional y técnico

---

## 1. Propósito y Criterios de Gestión de Pendientes

Este documento constituye la **única fuente de verdad** para las decisiones abiertas, incertidumbres funcionales, vacíos normativos y aspectos técnicos no resueltos de SIRACOM.

**Reglas de gobernanza:**
1. **No suponer resoluciones:** Ningún agente ni desarrollador puede asumir por cerrada una decisión que figure en este registro.
2. **Condición de cierre obligatoria:** Cada ítem pendiente cuenta con una condición de cierre explícita (evidencia documental, definición legal o decisión formal) sin la cual no puede darse por resuelto.
3. **Trazabilidad:** Al cerrarse una decisión, se actualiza su estado a *Cerrada*, se registra la fecha, se consigna la evidencia de resolución y, si amerita, se formaliza mediante un nuevo ADR o una actualización del documento de diseño respectivo.

---

## 2. Catálogo Consolidado de Decisiones Pendientes

| Código | Asunto Pendiente | Impacto en el Diseño / Operación | Condición de Cierre Explícita | Responsable Propuesto | Estado |
|---|---|---|---|---|:---:|
| **PEND-01** | Confirmación formal del cliente prioritario para validación | Define la prioridad en la elaboración de bocetos de pantalla y flujos de usuario (municipalidad vs. clínica privada). | Decisión escrita y comunicada por Jeanpier que ratifique a la Municipalidad o designe a la Clínica Privada como cliente piloto formal. | Jeanpier / Producto | **Abierto** |
| **PEND-02** | Obligatoriedad de formatos oficiales de salud pública (MINSA / DIRIS en Perú) | Determinará si se requieren plantillas de exportación con campos fijos normados para vigilancia epidemiológica (rabia, mordeduras, zoonosis). | Entrega de directivas sanitarias oficiales vigentes o pronunciamiento formal de que basta con reportes estadísticos internos exportables. | Analista Funcional / Enlace Municipal | **Abierto** |
| **PEND-03** | Modalidad de integración de facturación electrónica y comprobantes tributarios | Define si se requiere desarrollo de conector directo a API de PSE/OSE/SUNAT o si la exportación contable satisface la operación comercial inicial. | Definición contractual del alcance de cobros con el primer cliente comercial o decisión formal de mantenerlo en exportación sin validez tributaria directa inicial. | Arquitectura Técnica / Asesoría Contable | **Abierto** |
| **PEND-04** | Especies secundarias y parámetros clínicos particulares | Determinar qué parámetros clínicos específicos requerirán especies no convencionales cuando se activen en el catálogo (inicialmente solo caninos y felinos). | Aprobación de la ficha técnica de parámetros clínicos por parte del especialista veterinario para especies distintas a caninos y felinos. | Médico Veterinario Asesor | **Abierto** |
| **PEND-05** | Política institucional de protección de datos personales y retención (Ley 29733) | Define textos de consentimiento informado, plazos de conservación de historias clínicas y datos de responsables, ejercicio de derechos ARCO y protocolos de respaldo por instalación. | Aprobación de la política institucional de tratamiento de datos personales y cláusulas de confidencialidad por la asesoría legal correspondiente. | Asesoría Legal / Seguridad | **Abierto** |
| **PEND-06** | Procedimiento de contingencia operativa en campañas ante ausencia o corte de conectividad a Internet | Mitiga el riesgo operativo aceptado (ADR-003) de requerir conexión obligatoria en campañas masivas en campo. | Aprobación del protocolo operativo de contingencia (fichas físicas estandarizadas de contingencia y procedimiento de digitación diferida). | Coordinación Operativa / Analista Funcional | **Abierto** |
| **PEND-07** | Garantías técnicas de bitácora inalterable y cumplimiento de aislamiento (ADR-001) cuando el proveedor aloja la instalación (ADR-002) | Asegura que el personal del proveedor no tenga acceso implícito a datos de producción en entornos administrados en la nube del proveedor. | Especificación técnica de cifrado, segregación de llaves, permisos de infraestructura y procedimiento de auditoría para despliegues alojados por el proveedor. | Arquitectura Técnica / Seguridad | **Abierto** |
| **PEND-08** | Validez y mecanismo de formalización de cierre de actos médicos veterinarios | Determinar si se requiere firma digital formal (certificado digital RENIEC/proveedor acreditado) o autenticación bajo credencial interna en el expediente médico cerrado. | Informe de asesoría legal sobre validez de historia clínica veterinaria digital en Perú y pronunciamiento del especialista médico veterinario. | Asesoría Legal / Médico Veterinario Asesor | **Abierto** |
| **PEND-09** | Límites operativos de caja chica, arqueo y protocolo de anulación de cobros | Define montos máximos autorizados en caja chica, periodicidad de arqueos y procedimiento de anulación de comprobantes de cobro con intervención de supervisor. | Aprobación de las directivas de control interno y procedimientos administrativos de caja por el área contable o comercial. | Especialista Contable / Comercial | **Abierto** |
| **PEND-10** | Procedimiento formal de reasignación y transferencia de dueños de animales | Define el flujo de validación formal cuando un animal cambia de responsable (cesión voluntaria, extravío o fallecimiento del tenedor). | Elaboración del procedimiento funcional de transferencia de custodia y validación de requisitos de identidad y consentimiento de las partes. | Analista Funcional / Asesoría Legal | **Abierto** |
| **PEND-11** | Decisión formal sobre roles propuestos (Auditor, Coordinador de Campaña, Encargado de Albergue) | Define si se consolidan como roles independientes fijos o si se tratan como conjuntos de permisos configurables por instalación o responsabilidades asignables. | Decisión formal y documentada por Jeanpier antes de cerrar el Catálogo de Módulos (Etapa 3 del ciclo combinado). | Jeanpier / Producto | **Abierto** |
| **PEND-12** | Confirmación y significado oficial del acrónimo SIRACOM | Establecer la denominación oficial institucional o comercial del producto. | Comunicación formal por parte de Jeanpier de la denominación oficial o ratificación del nombre como marca propia. | Jeanpier / Producto | **Abierto** |
