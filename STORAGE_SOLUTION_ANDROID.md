# Solución: Error de Almacenamiento Insuficiente en Android

## Problema

Al intentar ejecutar `pnpm run:host:android:dev`, se presenta el siguiente error:

```
INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location
```

Este error indica que el dispositivo Android (emulador o físico) no tiene suficiente espacio de almacenamiento para instalar la aplicación.

## Diagnóstico

### 1. Verificar espacio disponible en el dispositivo

```bash
adb shell df -h
```

O específicamente para la partición `/data`:

```bash
adb shell df -h /data
```

**Resultado esperado:** Si el uso es superior al 85-90%, es probable que falle la instalación.

## Soluciones

### Solución 1: Limpiar caché del sistema

```bash
adb shell pm trim-caches 500M
```

Este comando libera hasta 500MB de caché del sistema Android.

### Solución 2: Desinstalar aplicaciones no necesarias

#### Listar aplicaciones instaladas

```bash
# Listar todas las aplicaciones
adb shell pm list packages

# Listar solo aplicaciones de terceros (no del sistema)
adb shell pm list packages -3

# Buscar una aplicación específica (ejemplo: Rimac)
adb shell pm list packages | grep -i rimac
```

#### Desinstalar aplicaciones

```bash
# Desinstalar una aplicación específica
adb uninstall <package-name>

# Ejemplo: Desinstalar la aplicación Rimac anterior
adb uninstall com.rimac.rimac_surrogas

# Desinstalar múltiples aplicaciones de prueba
adb uninstall com.proyectoprueba
adb uninstall com.ridesystem.app
adb uninstall com.example.myapplication
```

### Solución 3: Instalación directa con reemplazo forzado

Si el comando `pnpm run:host:android:dev` falla, puedes instalar directamente el APK:

```bash
cd packages/host/android
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

La opción `-r` (replace) permite reemplazar una instalación existente.

### Solución 4: Iniciar la aplicación manualmente

Después de instalar, puedes iniciar la aplicación manualmente:

```bash
adb shell am start -n com.rimac.rimac_surrogas/.MainActivity
```

## Pasos Completos de Resolución

### Paso 1: Verificar espacio disponible

```bash
adb shell df -h /data | grep -E "Filesystem|/data"
```

### Paso 2: Limpiar caché

```bash
adb shell pm trim-caches 500M
```

### Paso 3: Listar y desinstalar aplicaciones innecesarias

```bash
# Ver aplicaciones de terceros
adb shell pm list packages -3

# Desinstalar aplicaciones que no necesites
adb uninstall <package-name>
```

### Paso 4: Verificar espacio nuevamente

```bash
adb shell df -h /data | grep -E "Filesystem|/data"
```

### Paso 5: Instalar la aplicación

**Opción A:** Usando el comando original (recomendado si hay suficiente espacio):

```bash
pnpm run:host:android:dev
```

**Opción B:** Instalación directa del APK:

```bash
cd packages/host/android
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

### Paso 6: Iniciar la aplicación

```bash
adb shell am start -n com.rimac.rimac_surrogas/.MainActivity
```

## Información del Proyecto

- **Package Name:** `com.rimac.rimac_surrogas`
- **Main Activity:** `.MainActivity`
- **Ubicación del APK:** `packages/host/android/app/build/outputs/apk/debug/app-debug.apk`

## Prevención

Para evitar este problema en el futuro:

1. **Mantén el emulador limpio:** Desinstala aplicaciones de prueba regularmente
2. **Aumenta el almacenamiento del emulador:** Si usas un emulador, configura más espacio en el AVD Manager
3. **Limpia la caché periódicamente:** Ejecuta `adb shell pm trim-caches` regularmente
4. **Monitorea el espacio:** Verifica el espacio disponible antes de instalar aplicaciones grandes

## Comandos Útiles Adicionales

### Ver información detallada del dispositivo

```bash
adb shell getprop | grep -i "ro.product"
```

### Ver aplicaciones instaladas con su tamaño

```bash
adb shell pm list packages -f
```

### Limpiar datos de una aplicación específica (sin desinstalarla)

```bash
adb shell pm clear <package-name>
```

### Ver logs en tiempo real

```bash
adb logcat
```

### Reiniciar el dispositivo/emulador

```bash
adb reboot
```

## Notas

- El espacio mínimo recomendado para instalar aplicaciones React Native es de al menos **1GB libre**
- Si el problema persiste, considera usar un emulador con más almacenamiento o un dispositivo físico con más espacio
- Algunos emuladores tienen límites de almacenamiento que no se pueden cambiar después de crearlos; en ese caso, crea un nuevo emulador con más espacio

