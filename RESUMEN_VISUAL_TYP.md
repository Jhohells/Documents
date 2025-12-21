# 📊 Resumen Visual - AssistancesTyP

## 🎯 Vista General Rápida

```
┌─────────────────────────────────────────────────────────────┐
│                  ASSISTANCESTYP SCREEN                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           ESTADOS DE LA PANTALLA                     │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │  • Loading (Shimmer)                                 │  │
│  │  • Contenido Principal                               │  │
│  │  • Error (retorna null)                              │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              TRES MODALES PRINCIPALES                 │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │                                                       │  │
│  │  1️⃣  MODAL PERMISOS NOTIFICACIÓN                    │  │
│  │      ┌──────────────────────────────┐               │  │
│  │      │ ¿Activar notificaciones?     │               │  │
│  │      │                              │               │  │
│  │      │  [Sí]  [No]                  │               │  │
│  │      └──────────────────────────────┘               │  │
│  │                                                       │  │
│  │  2️⃣  MODAL ÉXITO PERMISOS                           │  │
│  │      ┌──────────────────────────────┐               │  │
│  │      │ ✅ Permisos activados         │               │  │
│  │      │                              │               │  │
│  │      │        [Continuar]           │               │  │
│  │      └──────────────────────────────┘               │  │
│  │                                                       │  │
│  │  3️⃣  MODAL CANCELACIÓN SERVICIO                     │  │
│  │      ┌──────────────────────────────┐               │  │
│  │      │ ⚠️  Cancelar asistencia?     │               │  │
│  │      │                              │               │  │
│  │      │  [No]  [Sí, cancelar]        │               │  │
│  │      └──────────────────────────────┘               │  │
│  │                                                       │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 🔄 Flujo Simplificado de Modales

```
INICIO
  │
  ├─► ¿Cargando datos?
  │   └─► SÍ → Mostrar Shimmer
  │   └─► NO → Continuar
  │
  ├─► ¿Tiene permisos de notificación?
  │   │
  │   ├─► NO → ┌─────────────────────────────┐
  │   │        │ MODAL PERMISOS NOTIFICACIÓN │
  │   │        └─────────────────────────────┘
  │   │              │
  │   │              ├─► Usuario ACEPTA
  │   │              │   └─► Ejecutar setupPushNotifications
  │   │              │       └─► (Solicita permisos internamente)
  │   │              │           └─► Verificar estado del permiso
  │   │              │               ├─► ¿Permiso otorgado?
  │   │              │               │   └─► SÍ → Marcar ref = true
  │   │              │               │       └─► ¿Cambió false → true?
  │   │              │               │           └─► SÍ → ┌──────────────────────┐
  │   │              │               │                     │ MODAL ÉXITO PERMISOS │
  │   │              │               │                     └──────────────────────┘
  │   │              │               │                           │
  │   │              │               │                           └─► Cerrar → Ir a Home
  │   │              │               │
  │   │              │               └─► NO → Cerrar modal (sin marcar ref)
  │   │              │
  │   │              └─► Usuario RECHAZA
  │   │                  └─► Cerrar modal → Continuar
  │   │
  │   └─► SÍ → Continuar
  │
  └─► MOSTRAR CONTENIDO PRINCIPAL
      │
      ├─► Usuario presiona VOLVER
      │   └─► Tracking → ¿Fin de flujo?
      │       └─► SÍ → Limpiar stores → Navegar atrás
      │       └─► NO → Navegar atrás
      │
      └─► Usuario presiona CANCELAR SERVICIO
          └─► ┌──────────────────────────────┐
              │ MODAL CANCELACIÓN SERVICIO    │
              └──────────────────────────────┘
                    │
                    ├─► Usuario CONFIRMA
                    │   └─► Enviar cancelación
                    │       └─► ¿Respuesta UPDATE_CASE?
                    │           └─► SÍ → Navegar a Tracking Screen
                    │           └─► NO → Cerrar modal
                    │
                    └─► Usuario CANCELA
                        └─► Cerrar modal
```

## 📋 Tabla de Condiciones de Modales

| Modal                     | Condición para Mostrar                                                                                                                     | Estado Inicial                           | Estado Final                                                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Permisos Notificación** | `hasPermission === false`<br/>`!shouldShowLoading`<br/>`hasData === true`                                                                  | `showNotificationPermissionModal = true` | Ejecuta setupPushNotifications → Verifica permiso → Si otorgado marca ref → `showNotificationPermissionModal = false` |
| **Éxito Permisos**        | `hasRequestedPermissionRef === true`<br/>`initialPermissionRef === false`<br/>`hasPermission === true`<br/>`!shouldShowLoading && hasData` | `showSuccessPermissionModal = true`      | `showSuccessPermissionModal = false` → Navega a Home                                                                  |
| **Cancelación**           | Usuario presiona botón cancelar                                                                                                            | `showCancelModal = true`                 | `showCancelModal = false`<br/>O navega a Tracking si `UPDATE_CASE`                                                    |

## 🎨 Diagrama de Estados de los Modales

```
                    ┌─────────────────┐
                    │   INICIALIZACIÓN │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Cargar Datos    │
                    └────────┬────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
        ┌───────────────┐        ┌───────────────┐
        │  Sin Permisos │        │  Con Permisos │
        └───────┬───────┘        └───────┬───────┘
                │                         │
                ▼                         │
    ┌───────────────────────┐            │
    │ Modal Permisos         │            │
    │ Notificación           │            │
    └───────┬───────────────┘            │
            │                             │
    ┌───────┴────────┐                    │
    │                │                    │
    ▼                ▼                    │
┌────────┐    ┌──────────────┐           │
│ Rechaza│    │   Acepta     │           │
└───┬────┘    └──────┬───────┘           │
    │                │                    │
    │                ▼                    │
    │        ┌──────────────┐             │
    │        │ Ejecutar      │             │
    │        │ setupPush     │             │
    │        │ Notifications │             │
    │        └──────┬───────┘             │
    │               │                     │
    │               ▼                     │
    │       ┌──────────────┐             │
    │       │ Verificar    │             │
    │       │ estado permiso│            │
    │       └───┬──────┬───┘             │
    │           │      │                  │
    │           │      └─ DENEGADO ───────┼──┐
    │           │                         │  │
    │           └─ OTORGADO               │  │
    │               │                     │  │
    │               ▼                     │  │
    │       ┌──────────────┐             │  │
    │       │ Marcar ref   │             │  │
    │       │ = true       │             │  │
    │       └──────┬───────┘             │  │
    │              │                     │  │
    │              ▼                     │  │
    │       ┌──────────────┐             │  │
    │       │ ¿Cambió      │             │  │
    │       │ false→true?  │             │  │
    │       └───┬──────┬───┘             │  │
    │           │      │                 │  │
    │           │      └─ NO ────────────┼──┼──┐
    │           │                         │  │  │
    │           └─ SÍ                     │  │  │
    │               │                     │  │  │
    │               ▼                     │  │  │
    │       ┌──────────────┐             │  │  │
    │       │ Modal Éxito  │             │  │  │
    │       │ Permisos     │             │  │  │
    │       └──────┬───────┘             │  │  │
    │              │                     │  │  │
    │              └─ Cerrar ────────────┼──┼──┼──┐
    │                                  │  │  │  │
    └──────────────────────────────────┼──┘  │  │
                                       │     │  │
                                       ▼     │  │
                              ┌──────────────┐  │
                              │   Contenido  │  │
                              │   Principal  │  │
                              └──────┬───────┘  │
                                     │          │
                                     └──────────┘
                                     │
                        ┌────────────┴────────────┐
                        │                         │
                        ▼                         ▼
                ┌──────────────┐         ┌──────────────┐
                │    Volver    │         │   Cancelar   │
                └──────┬───────┘         └──────┬───────┘
                       │                        │
                       │                        ▼
                       │              ┌─────────────────┐
                       │              │ Modal           │
                       │              │ Cancelación     │
                       │              └──────┬──────────┘
                       │                     │
                       │          ┌──────────┴──────────┐
                       │          │                     │
                       │          ▼                     ▼
                       │    ┌──────────┐         ┌──────────┐
                       │    │ Confirma │         │ Cancela  │
                       │    └────┬─────┘         └────┬─────┘
                       │         │                    │
                       │         ▼                    │
                       │    ┌──────────┐             │
                       │    │ Enviar   │             │
                       │    │ Cancel.  │             │
                       │    └────┬─────┘             │
                       │         │                   │
                       │    ┌────┴────┐              │
                       │    │         │              │
                       │    ▼         ▼              │
                       │ UPDATE_CASE  Otro           │
                       │    │         │              │
                       │    │         └──────────────┘
                       │    │              │
                       │    ▼              ▼
                       │ Navegar      Cerrar
                       │ Tracking     Modal
                       │    │              │
                       └────┴──────────────┘
                                │
                                ▼
                           ┌─────────┐
                           │   FIN    │
                           └─────────┘
```

## 🔑 Puntos Clave de la Lógica

### 1. **Gestión de Permisos**

- Se guarda el estado inicial (`initialPermissionRef`)
- Se rastrea si se solicitó **Y se otorgó** el permiso (`hasRequestedPermissionRef`)
- El ref solo se marca cuando el permiso fue realmente otorgado (`RESULTS.GRANTED` o `RESULTS.LIMITED`)
- Se verifica el estado del permiso después de ejecutar `setupPushNotifications()` para asegurar que realmente se otorgó
- Solo muestra modal de éxito si cambió de `false` a `true` y el ref fue marcado

### 2. **Prevención de Tracking Duplicado**

- `hasTrackedSuccessRef`: Evita tracking duplicado de éxito
- `hasTrackedTyPScreenRef`: Evita tracking duplicado de pantalla

### 3. **Navegación Condicional**

- Si se cancela y respuesta es `UPDATE_CASE` → Navega a Tracking Screen
- Si es fin de flujo y vuelve → Limpia stores CDR

### 4. **Estados de Carga**

- `shouldShowLoading`: Muestra shimmer mientras carga
- `isCancelling`: Muestra overlay durante cancelación
- `hasData`: Determina si mostrar contenido

## 📝 Notas Importantes

⚠️ **Los modales nunca se muestran simultáneamente**

- El modal de éxito cierra el de permisos
- Solo un modal visible a la vez

⚠️ **El modal de permisos solo se muestra una vez**

- Se verifica al cargar datos
- No se vuelve a mostrar si el usuario lo rechazó

⚠️ **El modal de éxito solo aparece si el permiso fue realmente otorgado**

- `setupPushNotifications()` siempre se ejecuta cuando el usuario presiona "Activar"
- Después se verifica el estado del permiso con `checkNotificationPermissionStatus()`
- Solo si el permiso está `GRANTED` o `LIMITED` se marca el ref y se muestra el modal de éxito

⚠️ **La cancelación tiene dos pasos**

1. Primero consulta (`OPERATION_CANCEL.CONSULT`)
2. Luego confirma (`OPERATION_CONSULT.CANCEL_YES`)
