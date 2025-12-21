# Análisis de la Pantalla AssistancesTyP

## 📋 Resumen General

La pantalla `AssistancesTyP` es una pantalla compleja que muestra el seguimiento de asistencia (CDR - Conductor Designado de Reemplazo) después de que el usuario ha solicitado una asistencia. Esta pantalla maneja múltiples estados, tres modales diferentes, y tiene lógica compleja para el manejo de permisos de notificaciones.

## 🎯 Propósito Principal

1. **Mostrar el estado de la asistencia**: Muestra los detalles de la asistencia solicitada o en seguimiento
2. **Gestionar permisos de notificaciones**: Solicita y gestiona permisos de notificaciones push para mantener al usuario informado
3. **Permitir cancelación de servicio**: Permite al usuario cancelar la asistencia solicitada
4. **Tracking y analytics**: Registra eventos para Adobe Analytics e Instana

## 🔄 Estados de la Pantalla

### Estados de Carga y Datos

- **`shouldShowLoading`**: Muestra un shimmer mientras carga los datos
- **`hasData`**: Indica si hay datos disponibles para mostrar
- **`isError`**: Indica si hubo un error al cargar los datos
- **`isEndFlow`**: Indica si es el final del flujo (usa `assistanceRequest` en lugar de `rawData`)

### Estados de Modales

- **`showCancelModal`**: Controla la visibilidad del modal de cancelación
- **`showNotificationPermissionModal`**: Controla la visibilidad del modal de permisos de notificación
- **`showSuccessPermissionModal`**: Controla la visibilidad del modal de éxito al otorgar permisos

## 🎭 Modales y sus Interacciones

### 1. Modal de Permisos de Notificación (`NotificationPermissionModal`)

**Cuándo se muestra:**

- Cuando `hasPermission === false` (no tiene permisos)
- Cuando `!shouldShowLoading` (ya terminó de cargar)
- Cuando `hasData === true` (hay datos para mostrar)

**Comportamiento:**

- El usuario puede aceptar o rechazar la solicitud de permisos
- Si acepta (`handleActivateNotifications` en el modal):
  - Se ejecuta `setupPushNotifications()` que solicita permisos internamente
  - Se verifica el estado del permiso después de ejecutar `setupPushNotifications()`
  - **Solo si el permiso fue otorgado** (`RESULTS.GRANTED` o `RESULTS.LIMITED`):
    - Se llama a `onActivateNotifications()` que marca `hasRequestedPermissionRef.current = true`
  - Se actualiza el estado del hook llamando a `requestPermission()`
  - Se cierra el modal
- Si rechaza o cierra:
  - Solo se cierra el modal

**Lógica de seguimiento:**

- Se guarda el estado inicial del permiso en `initialPermissionRef`
- `hasRequestedPermissionRef` se marca **solo cuando el permiso fue otorgado exitosamente**, no solo cuando se solicita
- El modal verifica el permiso después de `setupPushNotifications()` para asegurar que realmente se otorgó

### 2. Modal de Éxito de Permisos (`SuccessPermissionModal`)

**Cuándo se muestra:**

- Cuando se cumplen TODAS estas condiciones:
  - `hasRequestedPermissionRef.current === true` (se solicitó el permiso)
  - `initialPermissionRef.current === false` (no tenía permiso inicialmente)
  - `hasPermission === true` (ahora tiene permiso)
  - `!shouldShowLoading && hasData` (está listo)

**Comportamiento:**

- Muestra un mensaje de éxito
- Al cerrar o presionar el botón, ejecuta `handleGoBack` (navega a home)
- Se resetea `hasRequestedPermissionRef.current = false`

### 3. Modal de Cancelación (`CancelAssistanceModal`)

**Cuándo se muestra:**

- Cuando el usuario presiona el botón secundario (cancelar servicio)
- Controlado por `showCancelModal`

**Comportamiento:**

- Primera llamada (`handleOpenCancelModal`):
  - Hace tracking de Adobe e Instana
  - Llama a `handleCancelRequest` con `OPERATION_CANCEL.CONSULT`
  - Muestra el modal con la respuesta del servidor
- Si el usuario confirma cancelación (`handleCancelService`):
  - Si `data?.operation === OPERATION_CONSULT.CANCEL_YES`:
    - Llama nuevamente a `handleCancelRequest` con `OPERATION_CONSULT.CANCEL_YES`
    - Hace tracking de Instana
  - Cierra el modal
- Si el usuario cancela o cierra:
  - Solo cierra el modal (`handleCloseModal`)
  - Hace tracking de Instana

**Navegación especial:**

- Si `data?.operation === OPERATION_CONSULT.UPDATE_CASE`:
  - Navega automáticamente a `ASSISTANCE_TRACKING_SCREEN` con estado `CANCELLED`
  - Resetea la navegación

## 🔍 Hooks y Lógica Clave

### `useAssistanceData`

- Obtiene los datos de la asistencia desde la API
- Determina qué datos mostrar según `isEndFlow`
- Calcula estados de carga y visibilidad

### `useNotificationPermission`

- Verifica el estado actual de permisos de notificaciones
- Proporciona función para solicitar permisos
- Maneja diferencias entre iOS y Android

### `useTracking`

- Maneja todos los eventos de tracking (Adobe e Instana)
- Se ejecuta en momentos clave del flujo

### `useCancelRequestCDR`

- Maneja las peticiones de cancelación al servidor
- Proporciona estados de carga (`isCancelling`)

## 📊 Flujo de Datos y Estados

### Inicialización

1. Se cargan los datos de asistencia (`useAssistanceData`)
2. Se verifica el estado de permisos (`useNotificationPermission`)
3. Se guarda el estado inicial de permisos (`initialPermissionRef`)

### Flujo de Permisos de Notificación

1. Si no tiene permisos y hay datos → muestra `NotificationPermissionModal`
2. Usuario acepta → se ejecuta `setupPushNotifications()` (solicita permisos internamente)
3. Se verifica el estado del permiso después de `setupPushNotifications()`
4. **Solo si el permiso fue otorgado** → se marca `hasRequestedPermissionRef.current = true`
5. Si cambió de `false` a `true` → muestra `SuccessPermissionModal`
6. Usuario cierra éxito → navega a home

### Flujo de Cancelación

1. Usuario presiona botón cancelar → tracking → consulta servidor → muestra modal
2. Usuario confirma → envía cancelación → cierra modal
3. Si respuesta es `UPDATE_CASE` → navega a pantalla de seguimiento

## 🎨 Componentes Visuales

- **`AssistancesTyPShimmer`**: Muestra mientras carga
- **`AssistancesTyPContentScreen`**: Contenido principal con detalles de asistencia
- **`EmptyScrollWrapper`**: Wrapper con botones de acción
- **`LoadingMiniappTab`**: Overlay de carga durante cancelación

## 🔐 Referencias y Refs

- **`hasTrackedSuccessRef`**: Evita tracking duplicado de éxito
- **`hasTrackedTyPScreenRef`**: Evita tracking duplicado de pantalla TyP
- **`hasRequestedPermissionRef`**: Rastrea si se solicitó **Y se otorgó** el permiso (solo se marca cuando realmente se otorgó)
- **`initialPermissionRef`**: Guarda el estado inicial del permiso

## 🚨 Casos Especiales

1. **Error en carga**: Si `isError === true`, retorna `null` (no renderiza nada)
2. **Sin datos**: Si no hay `rawData` ni `assistanceRequest`, retorna `null`
3. **Fin de flujo**: Usa `assistanceRequest` en lugar de `rawData` cuando `isEndFlow === true`
4. **Navegación condicional**: Si se cancela exitosamente, navega automáticamente a seguimiento

## 📝 Notas Importantes

- Los modales de permisos tienen lógica compleja para evitar mostrar múltiples modales simultáneamente
- El tracking se ejecuta solo una vez usando refs para evitar duplicados
- La navegación puede resetearse completamente cuando se cancela el servicio
- Los permisos se verifican tanto al inicio como cuando cambian dinámicamente
- **IMPORTANTE**: El modal de éxito solo aparece si el permiso fue realmente otorgado. El ref `hasRequestedPermissionRef` se marca solo después de verificar que el permiso fue otorgado (`RESULTS.GRANTED` o `RESULTS.LIMITED`), no solo cuando se solicita
- `setupPushNotifications()` siempre se ejecuta cuando el usuario presiona "Activar", ya que esta función también solicita permisos internamente y puede mostrar el diálogo del sistema incluso si el permiso fue denegado previamente
