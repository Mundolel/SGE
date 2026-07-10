# Diagramas de Secuencia
### Sistema de Gestión Económica — Finca Ganadera
*Versión 1 · 9 de julio de 2026*

---

> **Nota:** Estos diagramas representan los flujos principales del sistema a nivel de análisis. Los nombres de componentes (UI, Controller, Repository, etc.) son genéricos — la tecnología concreta se define en el Paso 5. El objetivo es mostrar la interacción entre el actor y el sistema, no la arquitectura interna.

## SD-01 — Registrar transacción (ingreso o egreso)

Cubre UC-01 y UC-02 (HU-01, HU-02).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UI as Pantalla de Registro
    participant Ctrl as Controlador
    participant Repo as Almacén Local
    participant Sync as Cola de Sincronización

    Admin->>UI: Selecciona "Registrar ingreso/egreso"
    UI->>Ctrl: solicitarFormulario(tipo)
    Ctrl->>Repo: obtenerCategoríasActivas(tipo)
    Repo-->>Ctrl: listaCategorías
    Ctrl-->>UI: formulario(fecha=hoy, categorías)
    Admin->>UI: Completa campos y confirma

    UI->>Ctrl: registrarTransacción(datos)
    Ctrl->>Ctrl: validarCamposObligatorios()

    alt Validación falla
        Ctrl-->>UI: error("Monto es obligatorio")
        UI-->>Admin: Muestra error
    else Validación exitosa
        Ctrl->>Repo: guardarTransacción(transacción)
        Repo-->>Ctrl: transacciónGuardada

        alt Con conexión
            Ctrl->>Sync: sincronizar(transacción)
        else Sin conexión
            Ctrl->>Sync: encolar(transacción)
            Note over Sync: Se sincronizará<br/>cuando haya conexión
        end

        Ctrl-->>UI: confirmación
        UI-->>Admin: "Transacción registrada"
    end
```

---

## SD-02 — Consultar balance general con comparación de periodos

Cubre UC-03 (HU-03).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UI as Pantalla de Balance
    participant Ctrl as Controlador
    participant Repo as Almacén Local

    Admin->>UI: Abre pantalla de balance
    UI->>Ctrl: solicitarBalance(mesActual)
    Ctrl->>Repo: obtenerTransacciones(mesActual)
    Repo-->>Ctrl: transacciones
    Ctrl->>Ctrl: calcular(ingresos, egresos, neto)
    Ctrl-->>UI: balance(ingresos, egresos, neto)
    UI-->>Admin: Muestra balance del mes

    Admin->>UI: Selecciona "Comparar con mes anterior"
    UI->>Ctrl: solicitarComparación(mesActual, mesAnterior)
    Ctrl->>Repo: obtenerTransacciones(mesAnterior)
    Repo-->>Ctrl: transaccionesMesAnterior
    Ctrl->>Ctrl: calcular(ambos periodos + variación)
    Ctrl-->>UI: comparación(actual, anterior, variación)
    UI-->>Admin: Muestra ambos periodos lado a lado

    Admin->>UI: Selecciona "Desglose por actividad"
    UI->>Ctrl: solicitarDesglose(periodo)
    Ctrl->>Ctrl: agruparPorActividad(transacciones)
    Ctrl-->>UI: desglose(Lechería, Ganado, General)
    UI-->>Admin: Muestra subtotales por actividad
```

---

## SD-03 — Editar una transacción

Cubre UC-06 vía UC-05 (HU-06, HU-05).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UIHist as Pantalla de Historial
    participant UIEdit as Formulario de Edición
    participant Ctrl as Controlador
    participant Repo as Almacén Local

    Admin->>UIHist: Selecciona una transacción
    Admin->>UIHist: Elige "Editar"
    UIHist->>Ctrl: obtenerTransacción(id)
    Ctrl->>Repo: buscarPorId(id)
    Repo-->>Ctrl: transacción
    Ctrl-->>UIEdit: formulario(datosPrecargados)
    UIEdit-->>Admin: Muestra formulario con datos

    Admin->>UIEdit: Modifica campos y confirma
    UIEdit->>Ctrl: actualizarTransacción(id, datosNuevos)
    Ctrl->>Ctrl: validarCamposObligatorios()

    alt Validación falla
        Ctrl-->>UIEdit: error
        UIEdit-->>Admin: Muestra error
    else Validación exitosa
        Ctrl->>Repo: actualizar(id, datosNuevos)
        Repo-->>Ctrl: transacciónActualizada
        Ctrl-->>UIEdit: confirmación
        UIEdit-->>Admin: "Transacción actualizada"
    end
```

---

## SD-04 — Eliminar una transacción

Cubre UC-06 vía UC-05 (HU-06).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UIHist as Pantalla de Historial
    participant Dialog as Diálogo de Confirmación
    participant Ctrl as Controlador
    participant Repo as Almacén Local

    Admin->>UIHist: Selecciona una transacción
    Admin->>UIHist: Elige "Eliminar"
    UIHist->>Dialog: mostrarConfirmación("¿Eliminar esta transacción?")
    Dialog-->>Admin: Muestra diálogo

    alt Administrador confirma
        Admin->>Dialog: Confirma
        Dialog->>Ctrl: eliminarTransacción(id)
        Ctrl->>Repo: eliminar(id)
        Repo-->>Ctrl: eliminada
        Ctrl-->>UIHist: confirmación
        UIHist-->>Admin: "Transacción eliminada"
    else Administrador cancela
        Admin->>Dialog: Cancela
        Dialog-->>UIHist: sin cambios
    end
```

---

## SD-05 — Captura por foto → extracción IA → corrección → registro

Cubre UC-09 → UC-10 → UC-11 (HU-09, HU-10, HU-11). Solo aplica si se implementa el épico de IA (Could).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UI as Pantalla de Registro
    participant Cam as Cámara
    participant Ctrl as Controlador
    participant API as API IA Multimodal
    participant Repo as Almacén Local

    Admin->>UI: Selecciona "Registrar desde foto"
    UI->>Cam: abrirCámara()
    Admin->>Cam: Toma foto de anotación
    Cam-->>UI: vistaPrevia(imagen)
    UI-->>Admin: Muestra foto con "Usar" / "Volver a tomar"

    alt Volver a tomar
        Admin->>UI: "Volver a tomar"
        UI->>Cam: abrirCámara()
    else Usar foto
        Admin->>UI: "Usar foto"

        alt Sin conexión
            UI-->>Admin: "Esta función requiere internet"
            Note over UI: Ofrece guardar foto<br/>o registrar manualmente
        else Con conexión
            UI->>Ctrl: extraerDatos(imagen)
            Ctrl->>API: enviarImagen(imagen)
            Note over UI: Indicador de carga
            API-->>Ctrl: datosExtraídos(fecha, concepto, monto)

            alt Extracción fallida
                Ctrl-->>UI: error("No se pudo leer la anotación")
                UI-->>Admin: Ofrece retomar foto o registro manual
            else Extracción exitosa
                Ctrl-->>UI: formularioPrellenado(datos, imagenRef)
                UI-->>Admin: Muestra formulario + foto de referencia

                Admin->>UI: Revisa, corrige si necesario, confirma
                UI->>Ctrl: registrarTransacción(datosFinales)
                Ctrl->>Ctrl: validar()
                Ctrl->>Repo: guardar(transacción)
                Repo-->>Ctrl: guardada
                Ctrl-->>UI: confirmación
                UI-->>Admin: "Transacción registrada"
            end
        end
    end
```

---

## SD-06 — Exportar reporte

Cubre UC-08 (HU-08).

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UI as Pantalla de Balance/Historial
    participant Ctrl as Controlador
    participant Gen as Generador de Reportes
    participant OS as Sistema Operativo

    Admin->>UI: Selecciona "Exportar"
    UI-->>Admin: Muestra opciones de formato (PDF / Excel)
    Admin->>UI: Elige formato

    UI->>Ctrl: exportar(datos, filtrosActivos, formato)
    Ctrl->>Gen: generar(datos, formato)

    alt Sin datos
        Gen-->>Ctrl: error("No hay datos para exportar")
        Ctrl-->>UI: error
        UI-->>Admin: "No hay datos para exportar"
    else Con datos
        Gen-->>Ctrl: archivo(bytes, nombreArchivo)
        Ctrl->>OS: compartir/guardar(archivo)
        OS-->>Admin: Muestra opciones del sistema (compartir, guardar, etc.)
    end
```

---

## Resumen de cobertura

| Diagrama | Casos de uso | Historias de usuario | Prioridad |
|---|---|---|---|
| SD-01 | UC-01, UC-02 | HU-01, HU-02 | Must |
| SD-02 | UC-03 | HU-03 | Must |
| SD-03 | UC-05, UC-06 | HU-05, HU-06 | Must |
| SD-04 | UC-05, UC-06 | HU-05, HU-06 | Must |
| SD-05 | UC-09, UC-10, UC-11 | HU-09, HU-10, HU-11 | Could |
| SD-06 | UC-08 | HU-08 | Should |

**Nota:** UC-04 (Gestionar categorías) y UC-07 (Autenticarse) no tienen diagrama de secuencia dedicado porque sus flujos son lineales y quedan suficientemente cubiertos en las especificaciones de casos de uso. UC-12 (Reportes visuales) es análogo a SD-02 (consulta de datos + renderizado).
