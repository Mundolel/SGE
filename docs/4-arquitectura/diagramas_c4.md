# Diagramas C4
### Sistema de Gestión Económica — Finca Ganadera
*Versión 1 · 9 de julio de 2026*

---

## Nivel 1 — Contexto del Sistema

Muestra el sistema como caja negra, su actor principal y los sistemas externos con los que interactúa.

```mermaid
C4Context
    title Diagrama de Contexto — SGE Finca

    Person(admin, "Administrador", "Dueño/encargado de la finca. Único usuario del sistema.")

    System(sge, "SGE Finca", "App Android para registrar ingresos/egresos y consultar el balance financiero de la finca.")

    System_Ext(gdrive, "Google Drive", "Almacenamiento cloud para backup automático de la BD.")
    System_Ext(aiapi, "API IA Multimodal", "Servicio externo para extraer datos de fotos de anotaciones manuscritas. (Could)")

    Rel(admin, sge, "Registra transacciones, consulta balance, exporta reportes", "Android")
    Rel(sge, gdrive, "Backup periódico de la BD", "Google Drive API")
    Rel(sge, aiapi, "Envía foto, recibe datos extraídos", "HTTPS / REST")
```

### Notas
- La API IA Multimodal es opcional (prioridad Could). Solo se invoca si se implementa el épico de captura por foto (RF-09/10/11). El proveedor concreto se decidirá en la fase de implementación.
- Google Drive se usa exclusivamente para backup, no para sincronización multi-dispositivo (decisión ADR-003).

---

## Nivel 2 — Contenedores

Muestra las unidades desplegables del sistema y cómo se comunican.

```mermaid
C4Container
    title Diagrama de Contenedores — SGE Finca

    Person(admin, "Administrador")

    System_Boundary(sge, "SGE Finca") {
        Container(app, "App Android", "Kotlin, Jetpack Compose", "Interfaz de usuario: registro, historial, balance, reportes, configuración.")
        ContainerDb(room, "Base de Datos Local", "Room / SQLite", "Fuente de verdad. Almacena transacciones, categorías, usuario y configuración.")
        Container(backup, "Servicio de Backup", "WorkManager", "Tarea en segundo plano que exporta la BD a Google Drive periódicamente.")
    }

    System_Ext(gdrive, "Google Drive", "Almacenamiento cloud del administrador.")
    System_Ext(aiapi, "API IA Multimodal", "Extracción de datos desde foto. (Could)")

    Rel(admin, app, "Usa", "Pantalla táctil")
    Rel(app, room, "Lee/escribe", "Room DAOs")
    Rel(backup, room, "Exporta BD", "SQLite file")
    Rel(backup, gdrive, "Sube backup", "Google Drive API")
    Rel(app, aiapi, "Envía foto, recibe JSON", "HTTPS")
    Rel(app, backup, "Dispara backup manual", "WorkManager")
```

### Notas
- Solo hay un contenedor desplegable: la **app Android** (APK). Room y WorkManager son componentes internos de la app, no servicios separados.
- No hay backend propio ni servidor remoto (ADR-003).

---

## Nivel 3 — Componentes

Muestra los módulos internos de la app Android y sus responsabilidades.

```mermaid
C4Component
    title Diagrama de Componentes — App Android SGE Finca

    Container_Boundary(app, "App Android") {

        Component(nav, "Navigation", "Jetpack Navigation Compose", "Gestiona la navegación entre pantallas.")

        Component(ui_reg, "Pantallas de Registro", "Compose", "Formularios de ingreso/egreso. UC-01, UC-02.")
        Component(ui_hist, "Pantalla de Historial", "Compose", "Lista filtrable de transacciones. UC-05, UC-06.")
        Component(ui_bal, "Pantalla de Balance", "Compose", "Balance general, comparación, desglose. UC-03.")
        Component(ui_rep, "Pantalla de Reportes", "Compose", "Gráficos y exportación. UC-08, UC-12.")
        Component(ui_cfg, "Pantalla de Config", "Compose", "Categorías, backup, cerrar sesión. UC-04.")
        Component(ui_auth, "Pantalla de Auth", "Compose", "Login / registro inicial. UC-07.")
        Component(ui_cam, "Captura por Foto", "Compose + CameraX", "Cámara + preview. UC-09. (Could)")

        Component(vm, "ViewModels", "Hilt + StateFlow", "Estado de UI, lógica de presentación.")
        Component(uc, "Use Cases", "Kotlin", "Lógica de negocio: cálculo de balance, validaciones.")
        Component(repo, "Repositories", "Kotlin", "Abstracción de acceso a datos.")
        Component(dao, "Room DAOs", "Room", "Queries SQL para CRUD y agregaciones.")
        Component(db, "Room Database", "SQLite", "Esquema, migraciones, instancia singleton.")
        Component(bk, "Backup Service", "WorkManager", "Exporta/importa BD a Google Drive.")
        Component(ai, "AI Client", "Retrofit/Ktor", "Envía foto a API multimodal, parsea respuesta. (Could)")
    }

    System_Ext(gdrive, "Google Drive")
    System_Ext(aiapi, "API IA Multimodal")

    Rel(nav, ui_reg, "")
    Rel(nav, ui_hist, "")
    Rel(nav, ui_bal, "")
    Rel(nav, ui_rep, "")
    Rel(nav, ui_cfg, "")
    Rel(nav, ui_auth, "")
    Rel(nav, ui_cam, "")

    Rel(ui_reg, vm, "Eventos / Estado")
    Rel(ui_hist, vm, "Eventos / Estado")
    Rel(ui_bal, vm, "Eventos / Estado")
    Rel(ui_rep, vm, "Eventos / Estado")
    Rel(ui_cfg, vm, "Eventos / Estado")
    Rel(ui_auth, vm, "Eventos / Estado")
    Rel(ui_cam, vm, "Eventos / Estado")

    Rel(vm, uc, "Invoca")
    Rel(vm, repo, "CRUD directo")
    Rel(uc, repo, "Consulta / persiste")
    Rel(repo, dao, "Queries")
    Rel(dao, db, "SQLite")
    Rel(bk, db, "Exporta archivo")
    Rel(bk, gdrive, "Sube/descarga backup")
    Rel(ai, aiapi, "HTTPS")
    Rel(vm, ai, "Envía imagen")
```

### Mapeo componentes → requisitos

| Componente | Casos de Uso | Requisitos |
|---|---|---|
| Pantallas de Registro + VM | UC-01, UC-02 | RF-01, RF-02, RNF-01 |
| Pantalla de Balance + VM | UC-03 | RF-03, RNF-02 |
| Pantalla de Historial + VM | UC-05, UC-06 | RF-05, RF-06 |
| Pantalla de Config | UC-04 | RF-04 |
| Pantalla de Auth | UC-07 | RF-07, RNF-04 |
| Pantalla de Reportes + VM | UC-08, UC-12 | RF-08, RF-12 |
| Captura por Foto + AI Client | UC-09, UC-10, UC-11 | RF-09, RF-10, RF-11, RNF-03 |
| Room Database + DAOs | — (transversal) | RNF-06, RNF-07 |
| Backup Service | — (transversal) | RNF-04 |
| Repositories + Use Cases | — (transversal) | Testabilidad, mantenibilidad |
