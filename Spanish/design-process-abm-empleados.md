# Documentación del Proceso de Diseño
### US-001 · ABM de Empleados — Cómo y por qué se construyó esta User Story

---

## Tabla de contenidos

1. [¿Qué es un ABM?](#1-qué-es-un-abm)
2. [¿Qué es una User Story?](#2-qué-es-una-user-story)
3. [Proceso de pensamiento](#3-proceso-de-pensamiento)
4. [Anatomía de la User Story](#4-anatomía-de-la-user-story)
5. [Frameworks y herramientas utilizados](#5-frameworks-y-herramientas-utilizados)
6. [Lógica de negocio embebida](#6-lógica-de-negocio-embebida)
7. [Marco Normativo ISO — Requisitos y Cobertura](#7-marco-normativo-iso--requisitos-y-cobertura)
8. [Decisiones de diseño](#8-decisiones-de-diseño)
9. [Glosario](#9-glosario)

---

## 1. ¿Qué es un ABM?

**ABM** es el acrónimo de **Alta, Baja y Modificación**. Es un término ampliamente utilizado en el desarrollo de software hispanohablante (especialmente en Argentina) para describir el conjunto de operaciones básicas que permiten gestionar un conjunto de registros en un sistema.

En el mundo anglosajón, el equivalente conceptual es **CRUD** (Create, Read, Update, Delete).

| Operación ABM | Equivalente CRUD | Descripción |
|---|---|---|
| **Alta** | Create | Crear un nuevo registro |
| — | Read | Consultar / listar registros existentes |
| **Modificación** | Update | Editar un registro existente |
| **Baja** | Delete | Eliminar o desactivar un registro |

>  En el contexto empresarial, la **Baja** raramente es una eliminación física (borrar el registro de la base de datos). En la mayoría de los sistemas de gestión de personas, se implementa como una **baja lógica**: el registro permanece en el sistema pero su estado cambia a "Inactivo". Esto es crucial para auditoría, historial y cumplimiento legal.

### ¿Por qué un ABM de Empleados?

Un ABM de Empleados es el módulo central de cualquier sistema de Recursos Humanos (HRMS/HCM). Sin él, no es posible:

- Procesar liquidaciones de sueldos (el sistema de payroll necesita saber quiénes son los empleados)
- Gestionar accesos al edificio o sistemas internos
- Generar reportes de dotación para gerencia
- Cumplir con obligaciones legales ante organismos como AFIP/ARCA

Es decir, es un módulo **fundacional**: otros módulos dependen de que este exista y funcione correctamente.

---

## 2. ¿Qué es una User Story?

Una **User Story** (Historia de Usuario) es una descripción breve y centrada en el usuario de una funcionalidad que un sistema debe ofrecer. No describe *cómo* construirlo técnicamente, sino *qué necesita el usuario* y *por qué*.

### Formato base

```
Como [tipo de usuario],
quiero [realizar una acción],
para que [obtener un beneficio / resolver un problema].
```

Este formato de tres partes, conocido como **"Connextra template"** (por la empresa donde se originó), obliga a quien escribe la historia a:

1. Identificar **quién** tiene la necesidad (no el sistema, sino la persona)
2. Definir **qué** quiere hacer concretamente
3. Justificar **el valor** que esa acción le genera

Las User Stories son la unidad básica de trabajo en metodologías ágiles como **Scrum** y **Kanban**. Son intencionalmente simples en su núcleo para facilitar la conversación entre el Product Manager, el equipo de desarrollo y los stakeholders.

---

## 3. Proceso de Pensamiento

A continuación se describe el razonamiento que llevó a construir esta User Story, paso a paso.

### Paso 1 — Entender el dominio del problema

Lo primero fue identificar el contexto: un **ABM de Empleados** dentro de un sistema de gestión empresarial.

Preguntas que guiaron este análisis:

- ¿Quién usa este módulo? → No solo RRHH; también gerentes y los propios empleados tienen necesidades distintas.
- ¿Qué problema resuelve? → Reemplazar procesos manuales (Excel) propensos a errores e inconsistencias.
- ¿Qué pasa si no existe? → Otros sistemas (payroll, accesos) no tienen datos confiables.

### Paso 2 — Identificar todas las personas involucradas

Un error frecuente en el diseño de historias de usuario es escribirlas desde el punto de vista de un solo actor. En este caso, el módulo tiene al menos cuatro perfiles con necesidades distintas:

- **Admin RRHH**: necesita operar (alta, baja, modificación)
- **Gerente de área**: necesita consultar su equipo
- **Empleado**: necesita ver y validar sus propios datos
- **Auditor**: necesita trazabilidad de cambios

Esto impactó directamente en las **reglas de negocio** (quién puede hacer qué) y en los **criterios de aceptación**.

### Paso 3 — Definir los datos del dominio

Antes de pensar en pantallas o flujos, hay que saber qué información maneja la entidad. Se separaron los datos en tres grupos con lógica propia:

- **Datos personales**: pertenecen a la persona, existen antes que la relación laboral
- **Datos laborales**: definen la relación entre la persona y la empresa
- **Datos de sistema**: generados por la aplicación para su propio funcionamiento (legajo, email, auditoría)

Esta separación también tiene implicancias de **privacidad y permisos**: no todos los roles deberían ver todos los campos.

### Paso 4 — Escribir criterios de aceptación cubriendo el happy path y los edge cases

Los criterios de aceptación son la traducción de la historia en condiciones verificables. Se aplicó el principio:

> "Si QA no puede escribir un test case a partir de este criterio, el criterio está mal escrito."

Se cubrieron:
- El **camino feliz** (alta exitosa, edición exitosa)
- Los **casos borde** (DNI duplicado, campos incompletos)
- Las **operaciones de consulta** (búsqueda, filtrado, exportación)
- La **baja** como operación con lógica propia (no es simplemente borrar)

### Paso 5 — Derivar las reglas de negocio

Las reglas de negocio no son criterios de aceptación; son restricciones y comportamientos que aplican transversalmente a múltiples criterios. Se extrajeron de preguntas como:

- ¿Qué pasa si dos empleados tienen el mismo nombre? → El legajo los diferencia y es autogenerado.
- ¿Puede un gerente dar de baja a alguien de su equipo? → No, solo Admin RRHH.
- ¿Qué pasa con los datos de un empleado que se fue? → Deben conservarse (baja lógica + auditoría).

### Paso 6 — Pensar en riesgos y dependencias

Una User Story que no anticipa sus riesgos y dependencias genera sorpresas en el sprint. Se identificaron:

- **Dependencias técnicas**: otros módulos que deben existir antes (áreas, log de auditoría, API de correo)
- **Riesgos de adopción**: resistencia del equipo de RRHH al cambio
- **Riesgos legales**: cumplimiento con la Ley 25.326 de Protección de Datos Personales (Argentina)

### Paso 7 — Validar contra los estándares de calidad de la empresa

Una vez redactada la historia, se realizó un análisis de gaps contra las certificaciones vigentes de la empresa: **ISO/IEC 27001:2022** (Seguridad de la Información) e **ISO 9001:2015** (Gestión de la Calidad). Este paso se detalla en la sección 7.

---

## 4. Anatomía de la User Story

Cada sección de la historia cumple un rol específico. Esta tabla explica el propósito de cada una:

| Sección | Propósito | ¿Quién la usa principalmente? |
|---|---|---|
| **Metadata** (ID, épica, sprint, SP) | Trazabilidad y planificación en el backlog | Scrum Master, PM |
| **Historia de usuario** | Define el valor desde la perspectiva del usuario | Todo el equipo |
| **Contexto** | Justifica *por qué* es necesaria esta funcionalidad | Stakeholders, nuevos integrantes |
| **Personas / Roles** | Evita diseñar para un solo tipo de usuario | UX, Dev, PM |
| **Datos a gestionar** | Define el modelo de datos a alto nivel | Backend, DBA |
| **Criterios de aceptación** | Condiciones verificables para considerar la historia "done" | QA, Dev |
| **Reglas de negocio** | Restricciones y comportamientos transversales | Dev, QA, Legal |
| **Flujo de usuario** | Visualiza la secuencia de pasos del happy path | UX, Dev Frontend |
| **Definition of Done** | Checklist de calidad acordado por el equipo | Todo el equipo |
| **Supuestos y dependencias** | Evita bloqueos y falsas suposiciones en el sprint | PM, Tech Lead |
| **Riesgos** | Anticipa problemas antes de que ocurran | PM, Stakeholders |
| **Compliance ISO** | Mapea controles normativos aplicables para trazabilidad auditora | Responsable de calidad, Auditor ISO |
| **Métricas de calidad** | Define indicadores medibles para validar el éxito en producción | PM, Responsable de calidad |
| **Out of Scope** | Define límites explícitos para evitar scope creep | PM, Stakeholders |

---

## 5. Frameworks y Herramientas Utilizados

### 5.1 Connextra Template
El formato estándar de User Story ("Como / Quiero / Para que") propuesto por Rachel Davies en Connextra (2001). Es el punto de partida universal en metodologías ágiles.

### 5.2 MoSCoW Prioritization
Framework para priorizar funcionalidades según su impacto:

| Categoría | Significado | Aplicación en esta US |
|---|---|---|
| **Must Have** | Crítico, sin esto el sistema no funciona | Alta y baja de empleados |
| **Should Have** | Importante, pero el sistema puede funcionar sin ello | Exportación a Excel |
| **Could Have** | Deseable si hay tiempo | Filtros avanzados |
| **Won't Have** | Fuera de alcance en esta iteración | Organigrama, módulo de vacaciones |

Esta US fue clasificada como **Must Have** porque otros módulos del sistema dependen de ella.

### 5.3 Formato Dado / Cuando / Entonces (Given / When / Then)
También conocido como **BDD (Behavior-Driven Development)**, este formato para los criterios de aceptación fue popularizado por Dan North. Su ventaja es que puede ser leído tanto por personas de negocio como por desarrolladores y testers, y puede traducirse directamente en tests automatizados.

```
Dado   [un contexto inicial / precondición]
Cuando [se realiza una acción]
Entonces [se produce un resultado observable y verificable]
```

### 5.4 Story Points (Estimación relativa)
Los **Story Points** no son horas. Son una medida relativa de esfuerzo, complejidad e incertidumbre. Se utiliza comúnmente la **secuencia de Fibonacci** (1, 2, 3, 5, 8, 13, 21) porque las diferencias entre números grandes son más difíciles de precisar.

Esta historia fue estimada inicialmente en **8 SP** y luego reestimada a **13 SP** al incorporar los criterios de aceptación y reglas de negocio derivados del análisis ISO.

### 5.5 Definition of Done (DoD)
La DoD es un acuerdo del equipo que define qué condiciones debe cumplir cualquier historia para considerarse completada. No es específica de esta historia; es un estándar del equipo. Incluir la DoD en la historia sirve para que todos los involucrados tengan claridad sobre lo que "terminar" significa.

En esta historia, la DoD fue ampliada para incluir verificaciones de seguridad (cifrado) y de proceso (trazabilidad ISO) a raíz del análisis de compliance.

### 5.6 INVEST Criteria
Cada buena User Story debería cumplir con los criterios **INVEST**:

| Letra | Criterio | ¿Lo cumple esta US? |
|---|---|---|
| **I** · Independent | Es independiente de otras historias en la medida posible |  Tiene dependencias declaradas pero no bloquea otras US |
| **N** · Negotiable | Los detalles son conversables, no son un contrato |  Los campos y reglas son un punto de partida |
| **V** · Valuable | Genera valor claro para el usuario |  Reemplaza un proceso manual propenso a errores |
| **E** · Estimable | El equipo puede estimar el esfuerzo |  13 SP acordados |
| **S** · Small | Es pequeña enough para completarse en un sprint |  En el límite; podría dividirse si el equipo lo considera necesario |
| **T** · Testable | Puede verificarse objetivamente |  Criterios de aceptación en formato BDD |

---

## 6. Lógica de Negocio Embebida

Esta sección explica las decisiones de negocio no triviales que están implícitas en la historia y que un lector sin contexto podría no entender.

### 6.1 Baja lógica vs. eliminación física

**¿Qué es?**
Cuando un empleado deja la empresa, su registro **no se borra** de la base de datos. En cambio, se marca como "Inactivo" y se le agrega una fecha de egreso y un motivo. El registro sigue existiendo y puede consultarse.

**¿Por qué?**
- **Auditoría**: las empresas están obligadas a mantener historial laboral.
- **Legal**: la Ley de Contrato de Trabajo en Argentina requiere conservar ciertos datos por años.
- **Integridad referencial**: si el empleado estuvo relacionado con otros registros (aprobaciones, reportes, tareas), eliminar su registro rompería esas relaciones.
- **Reincorporación**: si el empleado vuelve a la empresa, se puede reactivar su legajo con historia completa.

### 6.2 Legajo autogenerado y no editable

**¿Qué es?**
El legajo es el número interno único que identifica a un empleado dentro de la empresa.

**¿Por qué es autogenerado?**
- Evita duplicados y errores humanos.
- Garantiza que la numeración sea consistente y correlativa.
- Simplifica la experiencia del usuario (Admin RRHH no necesita decidir el número).

**¿Por qué no editable?**
Una vez asignado, el legajo puede estar referenciado en contratos, recibos de sueldo, sistemas de control de acceso y otros documentos. Permitir editarlo generaría inconsistencias graves.

### 6.3 Email corporativo autogenerado

**¿Qué es?**
Al dar de alta un empleado, el sistema genera automáticamente un email con el patrón `nombre.apellido@empresa.com`.

**¿Por qué automatizarlo?**
- Garantiza consistencia en el formato de los emails internos.
- Elimina un paso manual del proceso de onboarding.
- Puede integrarse con el proveedor de email (Google Workspace, Microsoft 365) para crear la cuenta automáticamente.

**Casos borde a considerar** (no cubiertos en esta versión):
- Dos empleados con el mismo nombre y apellido → el sistema debería agregar un número (`juan.garcia2@empresa.com`)
- Empleados con nombres compuestos o apellidos con espacios → requiere una política de normalización

### 6.4 Permisos por rol

La historia define cuatro niveles de acceso diferenciados:

| Rol | Alta | Edición | Baja | Consulta |
|---|---|---|---|---|
| Superadmin |  |  |  |  |
| Admin RRHH |  |  |  |  (todos) |
| Gerente de área |  |  |  |  (solo su equipo) |
| Empleado |  |  |  |  (solo sus datos) |

Esto refleja el principio de **mínimo privilegio**: cada usuario accede solo a lo que necesita para cumplir su función.

### 6.5 Log de auditoría

**¿Qué es?**
Un registro inmutable de cada cambio realizado sobre cualquier empleado. Cada entrada del log contiene:
- Usuario que realizó el cambio
- Fecha y hora exacta
- Campo modificado
- Valor anterior
- Valor nuevo

**¿Por qué es importante?**
- Permite responder ante una inspección laboral o legal
- Detecta cambios no autorizados
- Es requerido explícitamente por ISO/IEC 27001:2022 (control 8.15) y por la Ley 25.326
- Los logs deben retenerse por un mínimo de 5 años y no pueden ser modificados ni eliminados por ningún rol (RN-08)

### 6.6 Ley 25.326 — Protección de Datos Personales

Esta ley argentina (equivalente al GDPR europeo en su espíritu) regula cómo las empresas deben manejar datos personales. En el contexto de este módulo implica:

- Los empleados tienen derecho a acceder a sus propios datos
- Ciertos datos (como el DNI/CUIL) no pueden compartirse libremente entre roles
- Los datos deben protegerse con medidas de seguridad adecuadas (cifrado en reposo)
- El acceso a datos sensibles debe estar justificado y registrado

### 6.7 Revocación de accesos al dar de baja

Al desactivar un empleado, el sistema no solo cambia su estado: también revoca automáticamente su acceso al email corporativo y notifica al área de IT. Esto responde a un riesgo de seguridad real: un ex-empleado con acceso activo a sistemas internos representa una vulnerabilidad. Este comportamiento está cubierto por CA-08 y es exigido por ISO/IEC 27001:2022 control 5.18.

### 6.8 Cifrado y enmascaramiento de datos sensibles

Los campos DNI y CUIL se almacenan cifrados en reposo (AES-256 o equivalente) y se muestran enmascarados en la interfaz para roles sin permiso explícito. Esto responde a dos necesidades simultáneas:

- **Seguridad**: proteger datos sensibles ante una eventual brecha de la base de datos (ISO/IEC 27001:2022 control 8.24)
- **Privacidad**: un gerente que consulta a su equipo no necesita ver el número de documento completo de cada persona

---

## 7. Marco Normativo ISO — Requisitos y Cobertura

Esta sección describe qué exigen las normas ISO aplicables a un módulo de gestión de datos de personas, y cómo cada requisito queda cubierto dentro de esta User Story mediante criterios de aceptación, reglas de negocio, ítems de la Definition of Done o historias técnicas satélite.

### 7.1 Contexto normativo de la empresa

La empresa opera bajo dos certificaciones emitidas por IRAM con reconocimiento internacional IQNet:

| Estándar | Versión vigente | Alcance |
|---|---|---|
| **ISO/IEC 27001** | 2022 *(con enmienda 2024)* | Seguridad de la Información — gestión de riesgos, controles de acceso, criptografía, auditoría |
| **ISO 9001** | 2015 | Gestión de la Calidad — procesos, trazabilidad, satisfacción del cliente, mejora continua |

> **Nota sobre versiones:** No existe una versión ISO 27001:2024 ni ISO 9001:2024 como ediciones independientes del estándar. El "2024" que figura en el certificado IRAM corresponde a la fecha de emisión o auditoría del certificado, no a una nueva versión de la norma. La próxima revisión de ISO 9001 se espera para finales de 2026 (aún en borrador); para ISO 27001 no hay fecha anunciada de nueva revisión.

### 7.2 ISO/IEC 27001:2022 — Seguridad de la Información

ISO/IEC 27001 es el estándar internacional para la gestión de la seguridad de la información. Establece los requisitos para implementar, mantener y mejorar un Sistema de Gestión de Seguridad de la Información (SGSI). Su Anexo A define 93 controles organizados en cuatro dominios: organizacional, personas, físico y tecnológico.

Para un módulo que gestiona datos personales y laborales de empleados, los controles más relevantes son los siguientes:

**Control 5.12 — Clasificación de la información**

Este control exige que la información sea clasificada según su nivel de sensibilidad y que su acceso y tratamiento se gestionen en consecuencia. En esta historia, los campos DNI y CUIL están identificados como datos sensibles: no pueden ser exportados ni visualizados en texto plano por roles sin permiso explícito. Esto se implementa mediante CA-07 (exportación restringida por rol) y CA-09 (enmascaramiento en la interfaz).

**Control 5.15 — Control de acceso**

Exige que el acceso a la información esté restringido según las necesidades de cada rol. Esta historia implementa cuatro niveles de acceso diferenciados: Superadmin, Admin RRHH, Gerente de área y Empleado. Cada rol tiene permisos específicos de creación, edición, baja y consulta, definidos en RN-01 y RN-07.

**Control 5.18 — Derechos de acceso**

Establece que los derechos de acceso deben ser revocados cuando ya no sean necesarios, en particular ante el cese de la relación laboral. CA-08 garantiza que al dar de baja un empleado el sistema revoque automáticamente su acceso al email corporativo, notifique al área de IT y registre la acción en el log de auditoría.

**Control 5.31 — Requisitos legales y contractuales**

Exige identificar y cumplir con las obligaciones legales aplicables al tratamiento de datos. La DoD incluye la verificación formal con el área Legal del cumplimiento de la Ley 25.326 de Protección de Datos Personales (Argentina), que regula el tratamiento de datos personales de forma análoga al GDPR europeo.

**Control 8.5 — Autenticación segura**

Requiere que el acceso a los sistemas esté protegido mediante mecanismos de autenticación seguros. Este módulo depende del sistema de autenticación centralizado (SSO) ya implementado, documentado como supuesto en la historia. Su implementación y cumplimiento es responsabilidad del módulo de autenticación.

**Control 8.10 — Eliminación de información**

Establece que la información debe gestionarse de forma que no pueda ser recuperada de manera no autorizada tras su eliminación. En este módulo se aplica el principio de baja lógica (RN-02): ningún registro de empleado puede ser eliminado físicamente del sistema, garantizando integridad y trazabilidad permanente.

**Control 8.13 — Backup de información**

Exige que la información sea respaldada de forma regular y que los backups sean verificables. Este requisito es cubierto por la política de backup de infraestructura del área de IT/DevOps, documentado como fuera del alcance de esta User Story.

**Control 8.15 — Logging**

Requiere que los eventos relevantes del sistema sean registrados y que esos registros estén protegidos contra modificación o eliminación. RN-06 establece que toda modificación queda registrada en el log de auditoría. RN-08 agrega que esos logs deben retenerse por un mínimo de 5 años y no pueden ser alterados por ningún rol.

**Control 8.24 — Uso de criptografía**

Exige que la información sensible almacenada esté protegida mediante cifrado. RN-09 establece que los campos DNI y CUIL se almacenan cifrados en reposo con AES-256 o equivalente. La implementación técnica de este cifrado es responsabilidad de la historia satélite US-001-SEC.

### 7.3 ISO 9001:2015 — Gestión de la Calidad

ISO 9001 es el estándar internacional para sistemas de gestión de la calidad (SGC). Su enfoque central es la mejora continua, la orientación al cliente y la gestión basada en procesos y riesgos. A diferencia de ISO 27001, no prescribe controles técnicos específicos sino principios y cláusulas que deben materializarse en los procesos de la organización.

Para una historia de usuario en un entorno ágil certificado bajo ISO 9001, las cláusulas más relevantes son:

**Cláusula 6.1 — Acciones para abordar riesgos y oportunidades**

Exige que la organización identifique los riesgos que pueden afectar el logro de los resultados previstos y defina acciones para mitigarlos. Esta historia incluye una sección de Riesgos con cinco riesgos identificados, su nivel de impacto y las acciones de mitigación correspondientes.

**Cláusula 6.3 — Planificación de cambios**

Establece que los cambios al sistema de gestión deben realizarse de manera planificada. En el contexto de esta historia, cualquier modificación al alcance, criterios o reglas queda documentada en el proceso de diseño, con los Story Points actualizados y las razones registradas.

**Cláusula 8.1 — Planificación y control operacional**

Exige que los procesos necesarios para cumplir los requisitos estén planificados, controlados y documentados. Esta historia cubre este requisito mediante la documentación de supuestos, dependencias, flujo de usuario y criterios de aceptación verificables.

**Cláusula 8.5.1 — Control de la producción y de la provisión del servicio**

Establece que las condiciones controladas para la entrega del servicio deben incluir criterios de aceptación y evidencia de conformidad. La Definition of Done de esta historia incluye cobertura de tests igual o mayor al 80%, validación en staging y aprobación del Product Manager antes del cierre.

**Cláusula 8.5.2 — Trazabilidad**

Exige que el producto o servicio pueda ser identificado y trazado a lo largo de su ciclo de vida. Esta historia está identificada con un ID único (US-001), asociada a una épica y un sprint, y sus dependencias están referenciadas por nombre. Los criterios de aceptación son numerados y rastreables hasta los controles ISO que los originan.

**Cláusula 8.6 — Liberación de los productos y servicios**

Establece que no debe liberarse un producto hasta que se hayan completado satisfactoriamente las disposiciones planificadas. La DoD de esta historia incluye la aprobación formal del responsable de calidad como condición de cierre, además de la validación del Product Manager.

**Cláusula 9.1.1 — Seguimiento, medición, análisis y evaluación**

Exige que la organización determine qué necesita ser monitoreado y medido, y con qué métodos. Esta historia incluye una sección de Métricas de Calidad con cinco indicadores medibles — tasa de error en alta, tiempo promedio de alta, tiempo de respuesta del listado, tasa de revocación exitosa y satisfacción del usuario — cada uno con un umbral aceptable definido y un período de medición de 30 días post go-live.

**Cláusula 9.1.2 — Satisfacción del cliente**

Exige monitorear la percepción del cliente respecto al cumplimiento de sus requisitos. En este módulo, el "cliente" interno es el equipo de RRHH. La métrica de satisfacción del usuario (encuesta post go-live, umbral mínimo de 4.0 sobre 5) cubre este requisito de forma directa y medible.

### 7.4 Cobertura consolidada

La siguiente tabla resume cómo cada requisito normativo queda cubierto dentro de los artefactos de esta historia:

| Norma | Requisito | Artefacto que lo cubre |
|---|---|---|
| ISO 27001 — 5.12 | Clasificación de la información | CA-07, CA-09 |
| ISO 27001 — 5.15 | Control de acceso | RN-01, RN-07 |
| ISO 27001 — 5.18 | Derechos de acceso | CA-08 |
| ISO 27001 — 5.31 | Requisitos legales | DoD — verificación con Legal |
| ISO 27001 — 8.5 | Autenticación segura | Supuesto — módulo SSO |
| ISO 27001 — 8.10 | Eliminación de información | RN-02 |
| ISO 27001 — 8.13 | Backup | Out of scope — política IT/DevOps |
| ISO 27001 — 8.15 | Logging | RN-06, RN-08 |
| ISO 27001 — 8.24 | Criptografía | RN-09, CA-09, US-001-SEC |
| ISO 9001 — 6.1 | Gestión de riesgos | Sección de Riesgos |
| ISO 9001 — 6.3 | Planificación de cambios | Documentación del proceso |
| ISO 9001 — 8.1 | Control operacional | Supuestos, dependencias, flujo de usuario |
| ISO 9001 — 8.5.1 | Control de producción | DoD — cobertura de tests, validación en staging |
| ISO 9001 — 8.5.2 | Trazabilidad | ID de US, épica, sprint, referencias entre historias |
| ISO 9001 — 8.6 | Liberación del producto | DoD — aprobación del responsable de calidad |
| ISO 9001 — 9.1.1 | Seguimiento y medición | Sección de Métricas de Calidad |
| ISO 9001 — 9.1.2 | Satisfacción del cliente | Métrica de satisfacción post go-live |

---

## 8. Decisiones de Diseño

Estas son las decisiones conscientes que se tomaron al escribir la historia y las razones detrás de ellas.

### ¿Por qué incluir una sección "Out of Scope"?

El **scope creep** (expansión no controlada del alcance) es una de las principales causas de retrasos en proyectos de software. Definir explícitamente lo que *no* incluye una historia tiene el mismo valor que definir lo que sí incluye. Un stakeholder que lee "gestión de vacaciones: fuera de scope" sabe que debe buscar otra historia para eso, en lugar de asumirlo como parte de esta.

### ¿Por qué separar Reglas de Negocio de Criterios de Aceptación?

Los **Criterios de Aceptación** describen el comportamiento desde la perspectiva del usuario ("cuando hago X, veo Y"). Las **Reglas de Negocio** son restricciones y políticas que aplican transversalmente y que el sistema debe respetar siempre, independientemente del flujo. Mezclarlos genera confusión y criterios redundantes.

### ¿Por qué incluir riesgos en una User Story?

Una User Story que llega al sprint sin anticipar sus riesgos puede bloquearse durante el desarrollo. Identificar riesgos en el momento de refinamiento permite al equipo tomar acciones preventivas (como coordinar con Legal antes de empezar a codear) y no reactivas (descubrir el problema cuando la historia ya está en progreso).

### ¿Por qué organizar los datos del empleado en tres grupos?

Separar los datos en **Personales / Laborales / Sistema** tiene implicancias técnicas y de UX:

- En el **frontend**, permite organizar el formulario en pestañas lógicas que reducen la carga cognitiva del usuario.
- En el **backend**, sugiere una separación de responsabilidades (la tabla de datos personales podría ser reutilizable si la empresa maneja clientes u otros actores).
- En términos de **permisos**, algunos campos (ej: CUIL) son más sensibles que otros y requieren cifrado en reposo y enmascaramiento en la interfaz.

### ¿Por qué no numerar las dependencias con IDs de otras US?

En un backlog real, los IDs de otras historias solo existen una vez que esas historias fueron creadas y priorizadas. Inventar IDs arbitrarios (como US-002, US-010) genera confusión: implica que existe un backlog estructurado con 10+ historias cuando en realidad esas historias aún no fueron definidas. La decisión fue referenciar las dependencias por nombre descriptivo y marcarlas como "pendientes de creación", que es el estado honesto.

### ¿Por qué crear una historia técnica satélite (US-001-SEC) en lugar de incluir todo aquí?

El cifrado en reposo es trabajo de arquitectura y backend que puede ser estimado, planificado y asignado de forma independiente al flujo funcional del ABM. Incluirlo en esta historia inflaría innecesariamente su alcance y dificultaría la planificación del sprint. La historia satélite permite que ambas pistas avancen en paralelo si el equipo lo decide.

---

## 9. Glosario

| Término | Definición |
|---|---|
| **ABM** | Alta, Baja y Modificación. Equivalente hispano de CRUD (Create, Read, Update, Delete). |
| **AES-256** | Advanced Encryption Standard con clave de 256 bits. Estándar de cifrado simétrico ampliamente utilizado para proteger datos en reposo. |
| **BDD** | Behavior-Driven Development. Metodología de desarrollo que define el comportamiento del sistema en lenguaje natural mediante el formato Dado/Cuando/Entonces. |
| **Baja lógica** | Desactivación de un registro sin eliminarlo físicamente de la base de datos. El registro permanece con estado "Inactivo". |
| **Backlog** | Lista priorizada de todas las funcionalidades, mejoras y correcciones pendientes de un producto. |
| **Cifrado en reposo** | Técnica de seguridad que protege los datos almacenados en una base de datos o sistema de archivos mediante algoritmos criptográficos. Los datos solo son legibles por quien tiene la clave. |
| **Criterio de aceptación** | Condición específica y verificable que debe cumplirse para que una User Story se considere completada. |
| **DoD** | Definition of Done. Checklist de calidad acordado por el equipo que toda historia debe cumplir para cerrarse. |
| **Edge case** | Caso borde. Situación límite o poco frecuente que puede revelar comportamientos inesperados del sistema. |
| **Enmascaramiento** | Técnica de UI que oculta parcialmente un dato sensible en pantalla (ej: `••••••••123`) para proteger la privacidad sin impedir la identificación. |
| **Épica** | Unidad de trabajo grande que agrupa varias User Stories relacionadas bajo un objetivo común. |
| **Gap analysis** | Análisis de brechas. Proceso de comparar el estado actual de un artefacto contra un estándar o requisito para identificar lo que falta cubrir. |
| **HRMS/HCM** | Human Resource Management System / Human Capital Management. Sistema de gestión de recursos humanos. |
| **INVEST** | Acrónimo de criterios para evaluar la calidad de una User Story: Independent, Negotiable, Valuable, Estimable, Small, Testable. |
| **ISO/IEC 27001:2022** | Estándar internacional para la gestión de seguridad de la información (ISMS). La versión 2022 es la actualmente vigente; recibió una enmienda menor en 2024. |
| **ISO 9001:2015** | Estándar internacional para sistemas de gestión de la calidad (QMS). Es la versión actualmente vigente; una nueva revisión (ISO 9001:2026) está en borrador y se espera para finales de 2026. |
| **Legajo** | Número interno único que identifica a un empleado dentro de una empresa. |
| **Log de auditoría** | Registro inmutable de todos los cambios realizados sobre un dato, con información de quién, cuándo y qué cambió. |
| **Mínimo privilegio** | Principio de seguridad que establece que cada usuario debe tener acceso únicamente a los recursos necesarios para su función. |
| **MoSCoW** | Framework de priorización: Must Have, Should Have, Could Have, Won't Have. |
| **Payroll** | Sistema de liquidación de sueldos. |
| **Scope creep** | Expansión no controlada del alcance de un proyecto, generalmente por agregar funcionalidades no planificadas. |
| **Sprint** | Período de tiempo fijo (generalmente 1-4 semanas) en el que un equipo Scrum se compromete a completar un conjunto de historias. |
| **Story Points** | Unidad de medida relativa del esfuerzo, complejidad e incertidumbre de una User Story. No equivalen a horas. |
| **User Story** | Descripción breve de una funcionalidad desde la perspectiva del usuario final, que expresa qué necesita y por qué. |
| **US satélite** | Historia de usuario técnica derivada de una historia principal, que cubre un aspecto específico (generalmente de arquitectura o seguridad) que merece estimación y planificación independiente. |


---

*PM Challenge · Julio 2026 · Francisco Roldán — [roldanfrancisco.personal@gmail.com](mailto:roldanfrancisco.personal@gmail.com) · [LinkedIn](https://www.linkedin.com/in/roldanfrancisco) · [GitHub](https://github.com/roldn)*
