# CallPoint — ficha técnica de seguridad

**Versión 1.9 · 16 de agosto de 2026**

Resumen de qué hace la herramienta con la red, qué guarda en el equipo y qué no hace.
Todo lo que aquí se afirma es comprobable leyendo el propio `callpoint.html`, que es
el único fichero del producto.

---

## 1. Qué es

Una página HTML de un solo fichero que convierte unas coordenadas GPS en el punto
kilométrico de carretera más cercano, junto con el municipio, la dirección y el
servicio de emergencias responsable. Se usa como referencia durante llamadas de
emergencia en carretera.

- **No se instala.** Se abre con doble clic desde el disco o desde una carpeta de red.
- **No necesita servidor**, ni base de datos, ni cuenta de usuario.
- **No lleva claves de API**, credenciales ni secretos de ningún tipo.
- Todo el código es legible en texto plano dentro del fichero.

---

## 2. Tráfico de red

**La herramienta solo lee. No existe ni una sola petición `POST` o `PUT` en todo el
fichero**, por lo que no tiene ningún camino de código capaz de enviar información
fuera del equipo. Únicamente consulta servicios cartográficos públicos.

| Servicio | Para qué | Organismo |
|---|---|---|
| `services1.arcgis.com` | Hitos kilométricos de España; placas kilométricas y distritos policiales de Irlanda | IGN/CNIG, TII, An Garda Síochána |
| `www.cartociudad.es` | Municipio, provincia y portal en España | IGN (España) |
| `data.geopf.fr` | *Points de repère* de Francia | IGN (Francia) |
| `api.postcodes.io` | Código postal y condado del Reino Unido | ONS (datos abiertos) |
| `data.police.uk` | Cuerpo policial responsable en el Reino Unido | Home Office |
| `nominatim.openstreetmap.org` | Dirección más cercana | OpenStreetMap Foundation |
| `api.open-meteo.com` | Meteorología del punto | Open-Meteo |
| `server.arcgisonline.com` · `tile.opentopomap.org` | Imágenes del mapa | Esri, OpenTopoMap |

Todas las peticiones son HTTPS. Lo único que viaja en ellas son las coordenadas
consultadas; no se envía ningún identificador de usuario, equipo ni sesión.

---

## 3. Qué guarda en el equipo

`localStorage` del propio navegador, con claves bajo el prefijo `locpk_`:

- Frases configurables y sus colores
- Tema, modo claro/oscuro, idioma y capa de mapa preferida
- Ubicaciones guardadas por el usuario e historial de búsquedas

Todo permanece en la máquina. **No se usan cookies.** Los datos se pueden exportar o
borrar desde la propia interfaz.

---

## 4. Qué NO hace

| Comprobación | Resultado |
|---|---|
| `eval()` / `new Function()` | **0 apariciones** |
| Cookies (`document.cookie`) | **0 apariciones** |
| Analítica, telemetría, `sendBeacon`, Google Tag Manager | **0 apariciones** |
| `XMLHttpRequest` | **0** — solo `fetch`, en 8 puntos del código |
| Peticiones `POST` / `PUT` | **0** |
| Carga de código de terceros en tiempo de ejecución | **0** desde la v1.9 |

---

## 5. Cambios de la v1.9

Dos elementos que sí ejecutaban código externo se han eliminado:

1. **Librería del mapa (Leaflet 1.9.4).** Hasta la v1.8 se descargaba de la CDN pública
   `unpkg.com` en cada carga de la página. Ahora va **incrustada dentro del fichero**,
   junto con sus iconos. Deja de haber dependencia de un tercero y el mapa sigue
   funcionando si esa CDN está bloqueada o caída. Licencia BSD-2-Clause, aviso de
   copyright conservado.

2. **Respaldo JSONP de los hitos españoles.** Si la petición normal fallaba, se
   insertaba una etiqueta `<script>` apuntando al servidor y se ejecutaba su respuesta
   como código. **Eliminado.** No era necesario: el servicio del IGN admite peticiones
   normales entre orígenes, y ninguna de las otras fuentes ha tenido nunca ese respaldo.

**Resultado: la herramienta no descarga ni ejecuta ningún código que no venga dentro
del propio fichero.**

---

## 6. Origen del código

Repositorio público: <https://github.com/LeonidasGK/CallPoint>

El historial completo de cambios es consultable ahí, y la herramienta incluye un
registro de versiones visible desde su propia interfaz.

---

## 7. Consideraciones para su uso

- Las coordenadas consultadas viajan a los servicios cartográficos listados en el
  apartado 2. No van acompañadas de ningún dato de la llamada, del cliente ni del
  operador, pero conviene tenerlo presente al valorar su uso.
- Los datos son de fuentes oficiales, pero la herramienta **es una ayuda de
  consulta, no una fuente autorizada**: el resultado debe confirmarse con quien llama
  antes de comunicarlo a un servicio de emergencias.
- La cobertura se limita a España, Francia, Irlanda y el Reino Unido. Fuera de esas
  zonas la herramienta lo indica expresamente en lugar de dar un resultado dudoso.
