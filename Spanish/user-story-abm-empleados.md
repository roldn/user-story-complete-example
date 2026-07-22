# US-001 · ABM de Empleados
### Sistema de Gestión de Recursos Humanos

---

| Campo | Detalle | Campo | Detalle |
|---|---|---|---|
| **ID** | US-001 | **Módulo** | Gestión de Empleados |
| **Épica** | Gestión de Recursos Humanos | **Sprint** | Sprint 3 |
| **Prioridad** | Alta (Must Have) | **Story Points** | 13 SP |
| **Estado** | `Ready for Development` | **Versión** | 1.0 |
| **Autor** | Product Manager | **Fecha** | Julio 2026 |

---

## Tabla de Contenidos

1. [Historia de Usuario](#historia-de-usuario)
2. [Contexto y Descripción](#contexto-y-descripción)
3. [Personas / Roles Involucrados](#personas--roles-involucrados)
4. [Datos a Gestionar](#datos-a-gestionar-entidad-empleado)
5. [Criterios de Aceptación](#criterios-de-aceptación)
6. [Reglas de Negocio](#reglas-de-negocio)
7. [Flujo de Usuario](#flujo-de-usuario-happy-path)
8. [Definition of Done](#definition-of-done)
9. [Supuestos y Dependencias](#supuestos-y-dependencias)
10. [Riesgos](#riesgos)
11. [Compliance ISO](#compliance-iso--trazabilidad-de-controles)
12. [Métricas de Calidad](#métricas-de-calidad-iso-9001-cláusula-9111)
13. [Out of Scope](#out-of-scope)

---

## Historia de Usuario

> **Como** Administrador de RRHH,  
> **quiero** gestionar el alta, modificación y baja de empleados en el sistema,  
> **para que** la información del personal esté siempre actualizada y disponible para los procesos internos de la empresa.

---

## Contexto y Descripción

La empresa cuenta actualmente con un proceso manual de registro de empleados mediante planillas Excel, lo que genera errores de consistencia, duplicación de datos y dificultades en auditorías.

Este módulo de ABM centraliza la gestión del ciclo de vida del empleado desde su ingreso hasta su egreso, integrándose con los sistemas de payroll, control de accesos y reportes gerenciales.

La empresa opera bajo certificaciones **ISO/IEC 27001:2022** *(enm. 2024)* (Seguridad de la Información) e **ISO 9001:2015** (Gestión de la Calidad), por lo que este módulo debe cumplir con los controles y cláusulas aplicables de ambas normas.

---

## Personas / Roles Involucrados

| Rol | Perfil | Necesidad principal |
|---|---|---|
| **Admin RRHH** | Usuario primario del sistema | Crear, editar y dar de baja empleados de forma rápida y sin errores |
| **Gerente de área** | Consulta del listado de su equipo | Visualizar el estado y datos de sus reportes directos |
| **Empleado** | Consulta de sus propios datos | Ver y solicitar corrección de sus datos personales y laborales |
| **Auditor interno** | Revisión del historial de cambios | Consultar trazabilidad de modificaciones con fecha y usuario responsable |

---

## Datos a Gestionar (Entidad Empleado)

<details>
<summary><strong>Datos Personales</strong></summary>

- Nombre completo (nombre, apellido)
- DNI / CUIL *(almacenado cifrado — ver RN-09)*
- Fecha de nacimiento
- Género (M / F / No binario / Prefiero no decirlo)
- Nacionalidad
- Domicilio (calle, número, localidad, provincia, código postal)
- Teléfono de contacto y email personal

</details>

<details>
<summary><strong>Datos Laborales</strong></summary>

- Legajo / N° interno
- Fecha de ingreso
- Área / Departamento
- Puesto / Cargo
- Tipo de contrato (efectivo, a término, pasante, freelance)
- Modalidad de trabajo (presencial, remoto, híbrido)
- Sede / Sucursal asignada
- Supervisor directo (relación con otro empleado en el sistema)
- Estado (Activo / Inactivo / Con licencia)

</details>

<details>
<summary><strong>Datos de Acceso y Sistema</strong></summary>

- Email corporativo (autogenerado al dar de alta)
- Rol en el sistema (permisos)
- Fecha y usuario que creó / modificó el registro (auditoría)

</details>

---

## Criterios de Aceptación

### CA-01 — Alta de empleado exitosa
- **Dado** que estoy autenticado como Admin RRHH
- **Cuando** completo todos los campos obligatorios del formulario de alta y presiono "Guardar"
- **Entonces** el sistema crea el registro, genera el legajo automáticamente, asigna un email corporativo y muestra el mensaje "Empleado registrado correctamente"

### CA-02 — Alta con campos incompletos
- **Dado** que intento guardar un empleado sin completar un campo obligatorio
- **Cuando** presiono "Guardar"
- **Entonces** el sistema resalta los campos vacíos en rojo y muestra un mensaje descriptivo por cada campo faltante, sin guardar el registro

### CA-03 — Alta con DNI duplicado
- **Dado** que ingreso un DNI que ya existe en el sistema
- **Cuando** presiono "Guardar"
- **Entonces** el sistema bloquea la acción y muestra: "Ya existe un empleado registrado con este DNI"

### CA-04 — Modificación de empleado
- **Dado** que selecciono un empleado activo y edito uno o más campos
- **Cuando** guardo los cambios
- **Entonces** el sistema actualiza el registro y registra en el historial de auditoría el campo modificado, el valor anterior, el nuevo valor, fecha, hora y usuario que realizó el cambio

### CA-05 — Baja lógica de empleado
- **Dado** que selecciono la opción "Dar de baja" en un empleado activo
- **Cuando** confirmo la acción e ingreso el motivo (renuncia / despido / vencimiento de contrato / otro) y la fecha de egreso
- **Entonces** el estado cambia a "Inactivo", se registra el motivo y fecha, y el registro queda accesible en modo solo lectura para auditoría

### CA-06 — Búsqueda y filtrado
- **Dado** que accedo al listado de empleados
- **Cuando** utilizo los filtros por nombre, legajo, área, estado o fecha de ingreso
- **Entonces** el sistema muestra únicamente los registros que coincidan con los criterios en menos de 2 segundos

### CA-07 — Exportación de listado
- **Dado** que visualizo el listado de empleados (con o sin filtros aplicados)
- **Cuando** presiono "Exportar"
- **Entonces** el sistema descarga un archivo `.xlsx` con los datos visibles, excluyendo campos sensibles como CUIL si el rol no tiene permiso

### CA-08 — Revocación automática de accesos al dar de baja *(ISO 27001 control 5.18)*
- **Dado** que un Admin RRHH confirma la baja de un empleado
- **Cuando** el estado cambia a "Inactivo"
- **Entonces** el sistema revoca automáticamente el acceso al email corporativo, notifica al área de IT con los detalles del empleado desvinculado, y registra la revocación en el log de auditoría con fecha y hora exacta

### CA-09 — Enmascaramiento de datos sensibles por rol *(ISO 27001 control 8.24)*
- **Dado** que un usuario con rol Gerente de área o Empleado accede a un perfil
- **Cuando** visualiza los datos del empleado
- **Entonces** los campos DNI y CUIL se muestran enmascarados (ej: `••••••••123`) y no pueden ser copiados ni exportados por ese rol

---

## Reglas de Negocio

| ID | Regla |
|---|---|
| RN-01 | Solo los usuarios con rol **Admin RRHH** o **Superadmin** pueden crear, editar o dar de baja empleados |
| RN-02 | Un empleado **no puede ser eliminado físicamente**; solo puede pasarse a estado Inactivo (baja lógica) |
| RN-03 | El legajo se genera de forma **automática, correlativa y no editable** |
| RN-04 | El email corporativo se genera automáticamente bajo el patrón `nombre.apellido@empresa.com` al confirmar el alta |
| RN-05 | No se puede reactivar un empleado dado de baja sin aprobación de un usuario **Superadmin** |
| RN-06 | Toda modificación queda registrada en el log de auditoría con usuario, fecha/hora, campo modificado y valores anterior/nuevo |
| RN-07 | Los Gerentes de área solo pueden **visualizar** empleados de su departamento, no editarlos |
| RN-08 | Los logs de auditoría se retienen por un **mínimo de 5 años** y no pueden ser modificados ni eliminados por ningún rol *(ISO 27001 control 8.15)* |
| RN-09 | Los campos DNI y CUIL se almacenan **cifrados en reposo** (AES-256 o equivalente) y solo se muestran en texto plano a roles con permiso explícito *(ISO 27001 control 8.24)* |

---

## Flujo de Usuario (Happy Path)

```
Admin RRHH
    │
    ├─► Accede a RRHH > Empleados
    │         │
    │         ├─► [Listado] Filtra / Exporta
    │         │
    │         └─► [+ Nuevo Empleado]
    │                   │
    │                   ├─► Pestaña: Datos Personales
    │                   ├─► Pestaña: Datos Laborales
    │                   └─► Pestaña: Acceso al Sistema
    │                               │
    │                               └─► [Guardar]
    │                                       │
    │                         Sistema genera legajo + email
    │                                       │
    │                              Confirmación
    │
    ├─► [Editar empleado] → Modifica campos → Guarda → Log de auditoría
    │
    └─► [Dar de baja] → Modal: motivo + fecha → Confirmar → Estado: Inactivo
                                                                    │
                                                        Revocación de accesos (CA-08)
                                                        Notificación a IT
                                                        Log de auditoría
```

| Paso | Acción | Descripción |
|---|---|---|
| 1 | Acceso al módulo | Admin RRHH ingresa a **RRHH > Empleados** desde el menú principal |
| 2 | Listado general | Tabla paginada: Legajo · Nombre · Área · Estado · Fecha Ingreso. Permite filtrar y exportar |
| 3 | Iniciar alta | Presiona **"+ Nuevo Empleado"** → formulario organizado en pestañas |
| 4 | Completar formulario | Pestañas: Datos Personales / Datos Laborales / Acceso al Sistema |
| 5 | Guardar | Validación en tiempo real → genera legajo y email → confirmación |
| 6 | Editar empleado | Click en empleado → Editar → modificar → Guardar → log de auditoría |
| 7 | Baja de empleado | Click en "Dar de baja" → modal con motivo y fecha → Confirmar → revocación automática de accesos + notificación IT |

---

## Definition of Done

- [x] Todos los criterios de aceptación (CA-01 a CA-09) validados y aprobados por QA
- [x] Código revisado mediante pull request y aprobado por al menos un peer
- [x] Cobertura de pruebas unitarias e integración ≥ 80%
- [x] Funcionalidad probada en entorno de staging sin regresiones
- [x] Documentación técnica y de usuario actualizada
- [x] Product Manager validó la demo en staging
- [x] Cumplimiento con Ley 25.326 de Protección de Datos Personales verificado con área Legal
- [x] Cifrado de campos sensibles verificado por el área de Seguridad
- [x] Proceso de revocación de accesos probado end-to-end con IT
- [x] Revisión de trazabilidad ISO documentada y aprobada por el responsable de calidad

---

## Supuestos y Dependencias

**Supuestos**
- El sistema de autenticación (SSO / login) ya se encuentra implementado
- La tabla de áreas y cargos está precargada en el sistema
- El módulo de payroll puede consumir datos de este módulo vía API interna
- El área de IT dispone de una API o proceso para revocar accesos al proveedor de email corporativo

**Dependencias**

| Dependencia | Descripción | Estado |
|---|---|---|
| US — Gestión de Áreas y Departamentos | Debe existir para que el formulario de alta pueda asignar área y departamento | Pendiente de creación y estimación |
| US — Log de auditoría centralizado | Requerido para CA-04 y RN-08; debe definirse como historia propia o como componente de infraestructura | Pendiente de creación y estimación |
| US-001-SEC | Cifrado y enmascaramiento de datos sensibles | Historia técnica satélite — ver sección de Compliance |
| — | API de correo corporativo | Necesaria para generación, notificación y revocación del email asignado |
| — | API / proceso de IT para revocación de accesos | Requerida para CA-08 |

---

## Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Resistencia del equipo de RRHH a abandonar Excel | Alto | Capacitación temprana y periodo de transición paralelo |
| Migración de datos históricos con campos inconsistentes | Medio | Sprint de limpieza de datos previo al go-live |
| Compliance con Ley 25.326 de Protección de Datos | Alto | Revisión con área Legal antes del inicio del desarrollo |
| Proveedor de email sin API de revocación disponible | Medio | Evaluar alternativa manual con notificación automática a IT como contingencia |
| Impacto en performance por cifrado en campos de búsqueda | Medio | Diseñar índices sobre campos no sensibles; cifrar solo en reposo |

---

## Compliance ISO — Trazabilidad de controles

Esta sección mapea los requisitos normativos aplicables y el artefacto donde se resuelve cada uno. Está destinada al **responsable de calidad y al auditor ISO**.

### ISO/IEC 27001:2022 *(enm. 2024)* — Seguridad de la Información

| Control | Descripción | Cómo se cubre |
|---|---|---|
| **5.12** | Clasificación de la información | CA-07, CA-09: exportación y visualización restringida por rol |
| **5.15** | Control de acceso | RN-01, RN-07: permisos diferenciados por rol |
| **5.18** | Derechos de acceso | CA-08, RN-08: revocación automática en baja + log |
| **5.31** | Requisitos legales y contractuales | DoD: verificación con Legal; Ley 25.326 |
| **8.10** | Eliminación de información | RN-02: baja lógica, sin eliminación física |
| **8.15** | Logging | RN-06, RN-08: log inmutable con retención mínima de 5 años |
| **8.24** | Uso de criptografía | RN-09, CA-09: cifrado AES-256 en reposo + enmascaramiento en UI |
| **8.13** | Backup de información | Delegado a la política de backup de infraestructura — fuera del scope de esta US |
| **8.5** | Autenticación segura | Delegado al módulo de autenticación (SSO) — supuesto como implementado |

### ISO 9001:2015 — Gestión de la Calidad

| Cláusula | Descripción | Cómo se cubre |
|---|---|---|
| **6.1** | Acciones para abordar riesgos | Sección de Riesgos con mitigaciones definidas |
| **6.3** | Planificación de cambios | Versionado de la US (v1.0 → v1.1); changelog documentado |
| **8.1** | Planificación y control operacional | Supuestos, dependencias y flujo de usuario documentados |
| **8.5.1** | Control de producción y provisión | DoD con cobertura de tests ≥ 80% y validación en staging |
| **8.5.2** | Trazabilidad | ID de US, épica, sprint, y referencias entre historias |
| **8.6** | Liberación de productos | DoD incluye aprobación del PM y del responsable de calidad |
| **9.1.1** | Seguimiento y medición | Ver sección de Métricas de Calidad ↓ |
| **9.1.2** | Satisfacción del cliente | Pendiente: encuesta de satisfacción post go-live al equipo de RRHH |

### Historia técnica satélite

| ID | Descripción | Controles que cubre |
|---|---|---|
| **US-001-SEC** | Cifrado en reposo de datos sensibles (DNI/CUIL) y enmascaramiento en UI por rol | ISO 27001: 8.24, 5.12 |

---

## Métricas de Calidad *(ISO 9001 cláusula 9.1.1)*

Estas métricas deben relevarse durante los primeros 30 días post go-live y reportarse al responsable de calidad.

| Métrica | Descripción | Umbral aceptable |
|---|---|---|
| **Tasa de error en alta** | % de altas que resultan en error del sistema (excluye errores de usuario) | ≤ 1% |
| **Tiempo promedio de alta** | Tiempo desde que el Admin inicia el formulario hasta la confirmación | ≤ 3 minutos |
| **Tiempo de respuesta de listado** | Tiempo de carga del listado con filtros aplicados | ≤ 2 segundos (ya en CA-06) |
| **Tasa de revocación exitosa** | % de bajas donde la revocación de acceso se completó sin intervención manual | ≥ 98% |
| **Satisfacción del usuario** | Encuesta post go-live al equipo de RRHH (escala 1-5) | ≥ 4.0 |

---

## Out of Scope

Las siguientes funcionalidades quedan **fuera del alcance** de esta User Story:

- Gestión de vacaciones y licencias → `US-005`
- Liquidación de sueldos → módulo de Payroll
- Gestión de legajos digitales / documentación adjunta → `US-008`
- Organigrama interactivo
- Política de backup e infraestructura → responsabilidad del área de IT / DevOps
- Módulo de autenticación y SSO → supuesto como implementado

---

*PM Challenge · Julio 2026 · Francisco Roldán — [roldanfrancisco.personal@gmail.com](mailto:roldanfrancisco.personal@gmail.com) · [LinkedIn](https://www.linkedin.com/in/roldanfrancisco) · [GitHub](https://github.com/roldn)*
