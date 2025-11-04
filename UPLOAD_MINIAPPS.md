# Subir Miniapps a Zephyr

1. Posiciónate en la carpeta de la miniapp que deseas subir:
   ```bash
   cd packages/<miniapp>
   # Ejemplo: cd packages/accounting
   ```

2. Valida el bundle QA:
   ```bash
   pnpm bundle:android:qa
   # o
   pnpm bundle:ios:qa
   ```
   Verifica que el proceso termine sin errores.

3. Sube el bundle a Zephyr:
   ```bash
   ZC=1 pnpm bundle:android:qa
   # o
   ZC=1 pnpm bundle:ios:qa
   ```
   - La primera vez, se abrirá la web para autenticación en Zephyr.
   - Inicia sesión con tu usuario.
   - La subida será automática tras autenticación.

---
