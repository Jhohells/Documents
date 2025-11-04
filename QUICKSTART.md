# Super App - Guía Rápida de Inicio

Sigue estos pasos para levantar el proyecto en tu entorno local.

## 1. Clonar el repositorio

```bash
git clone <url-del-repo>
cd superapp
```

## 2. Instalación de dependencias

```bash
pnpm install
```

## 3. Instalación de pods (solo para iOS)

```bash
pnpm pods
```

## 4. Configuración de Firebase

**Android:**  
Coloca el archivo `google-services.json` en la ruta:  
`packages/host/android/app/`

**iOS:**  
Coloca el archivo `GoogleService-Info.plist` en la ruta:  
`packages/host/ios/`

## 5. Iniciar el servidor Metro (bundler)

```bash
pnpm start:dev
```

## 6. Ejecutar la app

**Android:** (en otra terminal)

```bash
pnpm run:host:android:dev
```

**iOS:** (solo si necesitas correr en iOS, usar XCode)

---

> ⚠️ **Notas:**
> - Asegúrate de tener configurado tu entorno con un emulador Android/iOS previamente instalado.
> - Si tienes problemas con pods en iOS, revisa que Xcode esté actualizado y configurado correctamente.
