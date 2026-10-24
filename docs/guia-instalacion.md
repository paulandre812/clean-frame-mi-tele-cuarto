# Guía de instalación: Samsung UN65AU7000FXZX

Esta guía instala "Mi Tele" como una app más del televisor. **No toca el firmware** y se puede desinstalar como cualquier app. Lo que **no** hacemos aquí es reemplazar el botón Home del control: el método del proyecto original sólo se probó en otro modelo (2024) y en esta tele podría dejar el inicio o el encendido en mal estado.

Datos de tu TV (de "Acerca de este TV"):

| Dato | Valor |
| --- | --- |
| Modelo | UN65AU7000FXZX (Crystal UHD 2021) |
| Software | T-KSU2EAKUC-2301.1 |
| Tizen | 6.0 (se confirma en el paso 3) |

No compartas el número de serie ni el "Unique Device ID" en sitios públicos.

## 0. Quita los anuncios de Samsung (5 minutos, sin riesgo)

Haz esto aunque no sigas con lo demás:

1. **Configuración → Soporte → Términos y privacidad**: desactiva "Servicios de información de visualización" e "Anuncios basados en intereses".
2. **Configuración → General → Funciones inteligentes**: desactiva "Ejecutar automáticamente Smart Hub". Así no aparece el inicio de Samsung con anuncios al prender la tele.

Los nombres exactos pueden variar un poco según el idioma.

## 1. Lo que necesitas

- Una computadora (Windows, Mac o Linux) conectada a **la misma red Wi-Fi** que el TV.
- Una **cuenta Samsung** (gratis).
- **Tizen Studio** con la extensión de TV y "Samsung Certificate Extension": [instrucciones oficiales](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/installing-tv-sdk.html).
- Este repositorio descargado (botón verde **Code → Download ZIP** en GitHub, o `git clone`).

## 2. Activa el modo desarrollador en el TV

1. Averigua la IP de tu computadora (Windows: `ipconfig`; Mac: Ajustes del sistema → Red). Algo como `192.168.1.25`.
2. En el TV abre **Inicio → Apps**.
3. En la pantalla de Apps, con el teclado numérico del control (botón **123**), marca **1 2 3 4 5**. Aparece "Developer mode".
4. Pon **Developer mode: On** y escribe la IP de **tu computadora** en "Host PC IP". Acepta.
5. **Desconecta el TV de la pared**, espera 10 segundos y vuelve a conectarlo. (En el TV de referencia, el reinicio normal no bastó.)
6. Anota la IP del **TV**: Configuración → General → Red → Estado de la red → Configuración IP.

## 3. Conecta la computadora al TV

En Tizen Studio abre **Tools → Device Manager → Remote Device Manager → +**, pon la IP del TV y el puerto **26101**, y activa la conexión.

O por terminal (la herramienta `sdb` está en `tizen-studio/tools/`):

```sh
sdb connect IP_DEL_TV:26101
sdb capability          # confirma la versión de Tizen (platform_version) y la arquitectura
sdb shell 0 applist     # lista las apps instaladas y sus ID
```

Guarda la salida de `applist`. La necesitamos para confirmar los ID de Netflix, YouTube, Prime Video y Disney+, y para encontrar Twitch y Gravity Player.

Si la conexión falla: revisa que ambas IPs sean correctas, que no estés en una VPN y repite el desconectar-de-la-pared.

## 4. Crea tus certificados (una sola vez)

En Tizen Studio: **Tools → Certificate Manager → + → Samsung → TV**.

1. Nombre del perfil: **`TeleCuarto`** (el script de empaquetado usa ese nombre).
2. Crea un **certificado de autor** nuevo. Guarda la contraseña y haz copia de seguridad del archivo: lo necesitas para futuras actualizaciones.
3. Inicia sesión con tu cuenta Samsung cuando lo pida (tú escribes la contraseña).
4. Crea el **certificado de distribuidor** y agrega el **DUID** de tu TV (si el TV está conectado, se elige de la lista).

Guía oficial: [creating certificates](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/creating-certificates.html).

## 5. Empaqueta, firma e instala

Desde la carpeta del repositorio:

```sh
npm ci
npm run export:public          # crea dist/CleanFrame-public-unsigned.wgt
```

Firma con tu perfil (la herramienta `tizen` está en `tizen-studio/tools/ide/bin/`):

```sh
tizen package -t wgt -s TeleCuarto -o dist/firmado -- dist/CleanFrame-public-unsigned.wgt
```

Instala:

```sh
tizen install -n dist/firmado/CleanFrame-public-unsigned.wgt -t NOMBRE_DEL_TV_EN_DEVICE_MANAGER
```

Si la instalación se queda en "transfer" sin terminar (le pasó al TV de referencia), hazlo en dos pasos:

```sh
sdb push dist/firmado/CleanFrame-public-unsigned.wgt /home/owner/share/tmp/sdk_tools/mitele.wgt
sdb shell 0 vd_appinstall TeleCuarto.CleanHome /home/owner/share/tmp/sdk_tools/mitele.wgt
```

Debe decir `install completed`. Para abrirla:

```sh
sdb shell 0 execute TeleCuarto.CleanHome
```

En el TV también aparece en **Apps** como "Mi Tele".

## 6. Agrega Twitch y Gravity Player

Instálalas primero desde la tienda de Samsung si aún no están. Luego, en Mi Tele:

1. Abre **All apps**: lista las apps instaladas en tu TV.
2. Para fijarlas en el inicio ve a **Settings → Apps on Home** y agrega Twitch y Gravity Player. Caben hasta 9 apps.

Usan el icono que da el propio TV.

## 7. Comprueba que todo funcione

- Abre cada app desde Mi Tele: Netflix, YouTube, Prime Video, Disney+, Twitch, Gravity Player.
- Abre **Settings → Samsung → Samsung Home** y **TV settings**. Esos dos accesos usan ID del TV de referencia (2024) y **puede que no funcionen en la tuya**. Si fallan, el botón Home del control sigue llevándote al inicio de Samsung.

Si alguna app no abre, comparte la salida de `sdb shell 0 applist` (sin el número de serie) para corregir su ID en `app/device.js` y `app/main.js`.

## Cómo usarla en el día a día

Como no cambiamos el botón Home, abre "Mi Tele" desde Apps. Con "Ejecutar automáticamente Smart Hub" desactivado y "Ejecutar automáticamente la última app" activado, la tele vuelve a la última app usada; si esa fue Mi Tele, arrancará en Mi Tele.

## Desinstalar

En el TV: **Apps → Configuración de apps → Mi Tele → Eliminar**. O `sdb shell 0 vd_appuninstall TeleCuarto.CleanHome`. Para salir del modo desarrollador, repite el **12345** en Apps y ponlo en Off.
