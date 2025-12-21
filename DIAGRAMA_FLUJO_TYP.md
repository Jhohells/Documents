# Diagrama de Flujo - AssistancesTyP

## Flujo Principal de la Pantalla

```mermaid
flowchart TD
    Start([Pantalla AssistancesTyP]) --> Init[Inicialización]
    Init --> CheckData{¿Hay datos?}
    CheckData -->|No| Loading[Mostrar Shimmer]
    CheckData -->|Sí| CheckError{¿Hay error?}
    CheckError -->|Sí| EndError[Retornar null]
    CheckError -->|No| CheckPermission{¿Tiene permisos<br/>de notificación?}

    CheckPermission -->|null| Wait[Esperar verificación]
    Wait --> CheckPermission
    CheckPermission -->|true| ShowContent[Mostrar Contenido]
    CheckPermission -->|false| ShowNotifModal[Mostrar Modal<br/>Permisos Notificación]

    ShowNotifModal --> UserDecision{Usuario decide}
    UserDecision -->|Acepta| SetupPush[Setup Push Notifications<br/>Solicita permisos internamente]
    UserDecision -->|Rechaza/Cierra| CloseNotifModal[Cerrar Modal]

    SetupPush --> VerifyPerm[Verificar estado<br/>del permiso]
    VerifyPerm --> CheckGranted{¿Permiso otorgado?<br/>GRANTED o LIMITED}
    CheckGranted -->|Sí| MarkRef[Marcar hasRequestedPermissionRef<br/>= true]
    CheckGranted -->|No| CloseNotifModal
    MarkRef --> CheckPermChange{¿Cambió permiso<br/>false → true?}
    CheckPermChange -->|Sí| ShowSuccessModal[Mostrar Modal<br/>Éxito]
    CheckPermChange -->|No| CloseNotifModal

    ShowSuccessModal --> CloseSuccessModal[Cerrar Modal Éxito]
    CloseSuccessModal --> GoHome[Navegar a Home]

    CloseNotifModal --> ShowContent
    ShowContent --> UserActions{Acciones del Usuario}

    UserActions -->|Volver| GoBack[Navegar Atrás]
    UserActions -->|Cancelar Servicio| OpenCancelModal[Abrir Modal<br/>Cancelación]

    OpenCancelModal --> CancelRequest[Request Cancel<br/>OPERATION_CONSULT]
    CancelRequest --> ShowCancelModal[Mostrar Modal<br/>Cancelación]

    ShowCancelModal --> CancelDecision{Usuario decide}
    CancelDecision -->|Confirma| ConfirmCancel[Request Cancel<br/>OPERATION_CONSULT.CANCEL_YES]
    CancelDecision -->|Cancela/Cierra| CloseCancelModal[Cerrar Modal]

    ConfirmCancel --> CheckResponse{¿Respuesta es<br/>UPDATE_CASE?}
    CheckResponse -->|Sí| NavigateTracking[Navegar a<br/>ASSISTANCE_TRACKING_SCREEN<br/>Estado: CANCELLED]
    CheckResponse -->|No| CloseCancelModal

    CloseCancelModal --> ShowContent
    GoBack --> CheckEndFlow{¿Es fin<br/>de flujo?}
    CheckEndFlow -->|Sí| ClearStores[Limpiar Stores CDR]
    CheckEndFlow -->|No| NavigateBack[Navegar Atrás]
    ClearStores --> NavigateBack

    GoHome --> End([Fin])
    NavigateBack --> End
    NavigateTracking --> End
    Loading --> CheckData
    EndError --> End

    style Start fill:#e1f5ff
    style End fill:#ffe1f5
    style ShowNotifModal fill:#fff4e1
    style ShowSuccessModal fill:#e1ffe1
    style ShowCancelModal fill:#ffe1e1
    style RequestPerm fill:#e1e1ff
    style ConfirmCancel fill:#ffe1e1
```

## Flujo Detallado de Modales

```mermaid
stateDiagram-v2
    [*] --> Cargando: Inicialización

    Cargando --> VerificandoPermisos: Datos cargados

    VerificandoPermisos --> SinPermisos: hasPermission === false
    VerificandoPermisos --> ConPermisos: hasPermission === true
    VerificandoPermisos --> Esperando: hasPermission === null

    SinPermisos --> ModalPermisos: Mostrar modal
    ModalPermisos --> UsuarioDecide: Usuario interactúa

    UsuarioDecide --> EjecutarSetupPush: Acepta
    UsuarioDecide --> CerrarModalPermisos: Rechaza/Cierra

    EjecutarSetupPush --> SetupPushNotifications: Ejecutar setupPushNotifications
    SetupPushNotifications --> VerificarPermiso: Verificar estado del permiso

    VerificarPermiso --> PermisoOtorgado: GRANTED o LIMITED
    VerificarPermiso --> PermisoDenegado: DENIED o BLOCKED

    PermisoOtorgado --> MarcarRef: Marcar hasRequestedPermissionRef = true
    PermisoDenegado --> CerrarModalPermisos

    MarcarRef --> VerificarCambio: hasPermission cambió?

    VerificarCambio --> ModalExito: false → true
    VerificarCambio --> ContenidoPrincipal: No cambió

    ModalExito --> ContenidoPrincipal: Usuario cierra

    CerrarModalPermisos --> ContenidoPrincipal

    ConPermisos --> ContenidoPrincipal

    ContenidoPrincipal --> ModalCancelacion: Usuario presiona cancelar
    ContenidoPrincipal --> NavegarAtras: Usuario presiona volver

    ModalCancelacion --> ConsultarCancelacion: Request OPERATION_CONSULT
    ConsultarCancelacion --> MostrarModalCancelacion: Mostrar respuesta

    MostrarModalCancelacion --> ConfirmarCancelacion: Usuario confirma
    MostrarModalCancelacion --> CerrarModalCancelacion: Usuario cancela

    ConfirmarCancelacion --> EnviarCancelacion: Request CANCEL_YES
    EnviarCancelacion --> VerificarRespuesta: Verificar respuesta

    VerificarRespuesta --> NavegarTracking: UPDATE_CASE
    VerificarRespuesta --> ContenidoPrincipal: Otra respuesta

    CerrarModalCancelacion --> ContenidoPrincipal

    NavegarTracking --> [*]
    NavegarAtras --> [*]

    note right of ModalPermisos
        Condiciones para mostrar:
        - hasPermission === false
        - !shouldShowLoading
        - hasData === true
    end note

    note right of ModalExito
        Condiciones para mostrar:
        - hasRequestedPermissionRef === true
        - initialPermissionRef === false
        - hasPermission === true
        - !shouldShowLoading && hasData
    end note
```

## Diagrama de Estados de los Modales

```mermaid
graph LR
    subgraph "Estados de Modales"
        A[showNotificationPermissionModal: false] -->|Condiciones cumplidas| B[showNotificationPermissionModal: true]
        B -->|Usuario acepta| C[Solicitar permiso]
        B -->|Usuario rechaza| A
        C -->|Permiso otorgado| D[Verificar cambio]
        C -->|Permiso denegado| A
        D -->|false → true| E[showSuccessPermissionModal: true]
        D -->|Sin cambio| A
        E -->|Usuario cierra| F[showSuccessPermissionModal: false]
        F --> G[Navegar a Home]
    end

    subgraph "Modal de Cancelación"
        H[showCancelModal: false] -->|Usuario presiona cancelar| I[showCancelModal: true]
        I -->|Usuario confirma| J[Enviar cancelación]
        I -->|Usuario cancela| H
        J -->|UPDATE_CASE| K[Navegar a Tracking]
        J -->|Otra respuesta| H
    end

    style B fill:#fff4e1
    style E fill:#e1ffe1
    style I fill:#ffe1e1
```

## Secuencia de Interacciones

```mermaid
sequenceDiagram
    participant U as Usuario
    participant P as Pantalla AssistancesTyP
    participant NP as NotificationPermissionModal
    participant SP as SuccessPermissionModal
    participant CM as CancelAssistanceModal
    participant API as API/Servicios

    Note over P: Inicialización
    P->>P: Cargar datos asistencia
    P->>P: Verificar permisos notificación

    alt No tiene permisos
        P->>NP: Mostrar modal permisos
        NP->>U: Solicitar permiso
        U->>NP: Acepta
        NP->>NP: Ejecutar setupPushNotifications
        NP->>API: Setup push notifications<br/>(solicita permisos internamente)
        API-->>NP: Resultado
        NP->>NP: Verificar estado del permiso<br/>checkNotificationPermissionStatus
        alt Permiso otorgado (GRANTED/LIMITED)
            NP->>NP: Marcar hasRequestedPermissionRef = true
            NP->>P: onActivateNotifications()
            NP->>NP: Actualizar estado hook
            NP->>NP: Cerrar modal
            P->>P: hasPermission = true
            P->>SP: Mostrar modal éxito
            SP->>U: Mostrar mensaje éxito
            U->>SP: Cerrar
            SP->>P: Navegar a home
        else Permiso denegado
            NP->>NP: Cerrar modal<br/>(sin marcar ref)
            NP->>P: Continuar flujo normal
        end
    else Tiene permisos o rechaza
        NP->>NP: Cerrar modal
        NP->>P: Continuar flujo normal
    end

    Note over P: Usuario interactúa con pantalla

    alt Usuario presiona cancelar
        U->>P: Presiona botón cancelar
        P->>P: Tracking Adobe/Instana
        P->>API: Request cancel (CONSULT)
        API-->>P: Respuesta cancelación
        P->>CM: Mostrar modal cancelación
        CM->>U: Mostrar opciones

        alt Usuario confirma cancelación
            U->>CM: Confirma cancelación
            CM->>P: handleCancelService
            P->>API: Request cancel (CANCEL_YES)
            API-->>P: Respuesta

            alt Respuesta es UPDATE_CASE
                P->>P: Navegar a ASSISTANCE_TRACKING_SCREEN
            else Otra respuesta
                P->>CM: Cerrar modal
            end
        else Usuario cancela
            U->>CM: Cancela/Cierra
            CM->>P: Cerrar modal
        end
    else Usuario presiona volver
        U->>P: Presiona volver
        P->>P: Tracking Adobe/Instana

        alt Es fin de flujo
            P->>P: Limpiar stores CDR
        end

        P->>P: Navegar atrás
    end
```

## Matriz de Estados y Condiciones

| Estado                | Condición para Mostrar                                                                                                                     | Acción del Usuario | Resultado                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | ---------------------------------------------------------------------------------------- |
| **Modal Permisos**    | `hasPermission === false`<br/>`!shouldShowLoading`<br/>`hasData === true`                                                                  | Acepta             | Ejecuta setupPushNotifications → Verifica permiso → Si otorgado marca ref → Cierra modal |
| **Modal Permisos**    | Mismo                                                                                                                                      | Rechaza/Cierra     | Solo cierra modal                                                                        |
| **Modal Éxito**       | `hasRequestedPermissionRef === true`<br/>`initialPermissionRef === false`<br/>`hasPermission === true`<br/>`!shouldShowLoading && hasData` | Cierra             | Navega a home                                                                            |
| **Modal Cancelación** | Usuario presiona botón cancelar                                                                                                            | Confirma           | Envía cancelación → Si UPDATE_CASE navega a tracking                                     |
| **Modal Cancelación** | Mismo                                                                                                                                      | Cancela/Cierra     | Solo cierra modal                                                                        |
