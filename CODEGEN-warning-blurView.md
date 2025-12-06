# Documentación: ¿Qué es Codegen en React Native?

## 🔧 ¿Qué es Codegen?

**Codegen** es el sistema de React Native que genera automáticamente
código nativo (Java/Kotlin para Android y Obj-C/Swift para iOS) basado
en definiciones de módulos y componentes escritos en JavaScript o
TypeScript.

Este sistema forma parte de la **Nueva Arquitectura de React Native**
(TurboModules + Fabric).

------------------------------------------------------------------------

## 🛠️ ¿Qué hace Codegen?

-   Genera bridges nativos automáticamente.
-   Evita escribir código nativo manualmente.
-   Optimiza la comunicación JS ↔ Nativo.
-   Es requerido por librerías modernas que usan la nueva arquitectura.

------------------------------------------------------------------------

## ⚠️ ¿Por qué aparece el mensaje?

    Codegen didn't run for AndroidBlurView. This will be an error in the future. Make sure you are using @react-native/babel-preset when building your JavaScript code.

Este warning indica que la librería **AndroidBlurView** (o sus
variantes) intenta usar Codegen, pero tu proyecto **no está configurado
correctamente** para ejecutarlo.

React Native planea hacer obligatorio Codegen, así que este warning será
un error en versiones futuras.

------------------------------------------------------------------------

## ✔️ ¿Cómo solucionarlo?

### 1. Verificar `babel.config.js`

Debe incluir:

``` js
module.exports = {
  presets: ['module:@react-native/babel-preset'],
};
```

Si no está → agrégalo.

------------------------------------------------------------------------

### 2. Limpiar y recompilar (Android)

``` bash
cd android
./gradlew clean
cd ..
npm start --reset-cache
npx react-native run-android
```

------------------------------------------------------------------------

## 🧩 ¿Por qué afecta a AndroidBlurView?

Porque esta librería usa: - Componentes nativos modernos. - Requiere
bindings generados automáticamente con Codegen. - Si no se ejecuta, usa
la arquitectura vieja (Old Architecture), la cual será removida
eventualmente.

------------------------------------------------------------------------

## 📌 Recomendación

Asegúrate de: - Tener configurado el nuevo preset de Babel. - Mantener
tu proyecto actualizado. - Revisar librerías que utilicen TurboModules o
Fabric.

------------------------------------------------------------------------

Si quieres, puedo ayudarte a revisar tu configuración actual del
proyecto.
