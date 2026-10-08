# Incógnito

Autoservicio web para que una persona vea dónde están publicados **sus propios** datos personales y arme el pedido de baja con la ley del país donde va a hacer el trámite.

Cobertura: **Argentina, Brasil, Perú y Chile**, tratados como jurisdicciones independientes. No hay flujo, identificadores ni base legal de Estados Unidos: el SSN no existe en estos cuatro países, así que no es un campo del producto, y si alguien lo pega igual la app se lo explica y le dice cuál es el equivalente local.

> No es Incogni (incogni.com). Ese es un servicio estadounidense de bajas ante *data brokers* que además deja a Google y Bing fuera de su alcance. Incógnito trabaja caso por caso, URL por URL, contra un responsable identificado, con el plazo legal del país y escalamiento a la autoridad, y sí gestiona la desindexación en buscadores.

## El ciclo

1. **Escaneo** — país, nombre, apellido y los identificadores que ese país exige. Gratis: se cobra la gestión de la baja, no ver tus propios datos.
2. **Triaje** — cada hallazgo se marca como *soy yo*, *omitir por ahora* o *no soy yo*. Los descartados se pueden recuperar. También se puede pegar a mano la URL exacta de una ficha que el escaneo no encontró.
3. **Pedidos por capas** — no hay un botón único de "enviar baja". Cada hallazgo recorre las capas que le correspondan, y cada capa tiene su propio paquete. La tarjeta muestra en qué capa está (*Capa 0 · fuente*, *Capa 1 · Google*, …, *Resuelto*) y qué toca hacer ahora.
4. **Re-escaneo** — los sitios que recopilan datos vuelven a listar. El re-escaneo marca qué apareció de nuevo y qué ya no está.

## La arquitectura por capas

Una **capa** es un destinatario o una instancia del reclamo. Una **pieza** es un envío concreto dentro de esa capa: un formulario, sus campos y su texto. Una capa puede tener varias piezas, porque Google, por ejemplo, tiene un formulario distinto por tipo de pedido y presentar uno no vale como el otro.

| Capa | Qué es | Piezas |
| --- | --- | --- |
| **0 · La fuente** | El sitio que publica. Es el único que puede hacer desaparecer el dato de internet, así que siempre se arranca acá. Si la fuente lo baja, la desindexación después se sostiene sola. | Pedido de supresión al responsable, por un canal que deje constancia fechada. |
| **1 · Google** | Google no borra del sitio: deja de mostrar la URL al buscar tu nombre. | Hasta tres, según el caso: el formulario de información personal (solo si el hallazgo expone un dato identificatorio), el canal jurídico con la ley local, y el formulario de solicitudes de retirada. |
| **2 · Bing y Microsoft** | Microsoft no tiene formulario de derecho al olvido para estos países: el europeo solo lista países de Europa. | El canal general de privacidad, y la herramienta de remoción de resultados, que solo corresponde si la fuente ya bajó o cambió la página. |
| **3 · Otros buscadores** | Vacía a propósito, ver más abajo. | Ninguna. |
| **4 · Segundo intento** | Se reitera el pedido en la misma capa donde no hubo respuesta, citando fecha y número del envío anterior. Muchos destinatarios contestan recién acá. | Reiteración al mismo destinatario. |
| **5 · Intimación legal** | Carta formal por medio fehaciente que deja asentado el incumplimiento y anuncia el cauce que la ley prevé. No es un emplazamiento judicial —eso solo lo dicta un juez— y no anuncia multas, denuncias penales ni consecuencias inventadas. | Intimación al responsable. |
| **6 · Autoridad del país** | Reclamo ante la autoridad de datos, o la vía judicial en Chile mientras no haya agencia con competencia sobre privados. | Presentación ante la autoridad o ante el tribunal. |

Las capas 0, 1 y 2 arrancan juntas: son trámites distintos ante destinatarios distintos. Las capas 4, 5 y 6 están encadenadas y se habilitan cuando la anterior se agotó —plazo vencido o respuesta negativa— y el contenido sigue publicado. La capa 6 además exige una confirmación explícita del titular: no se habilita sola.

Cada pieza dice solo lo que está verificado. Los formularios de Google y la herramienta de remoción de Microsoft exigen iniciar sesión, así que la app no adivina sus campos ni les inventa un plazo de respuesta: avisa que no hay plazo declarado y te dice qué llevar escrito antes de empezar.

### Por qué la capa 3 está vacía

Porque no hay un método verificado. No encontramos, para Argentina, Brasil, Perú ni Chile, un canal de remoción propio de otro buscador, y preferimos mostrar una capa vacía con el motivo antes que un formulario inventado.

- **DuckDuckGo, Yahoo, Ecosia, Startpage y similares** no tienen índice web propio: revenden o combinan los de Bing y Google. Se resuelven en las capas 1 y 2, sin pedido separado.
- **Brave** sí tiene índice propio, pero no verificamos un canal de remoción para estos países y su uso en la región es bajo.
- **Yandex** tiene formulario, pero no verificamos si acepta solicitantes desde fuera de Rusia.

### Por qué no se simula un formulario de Safari

**Safari no es un buscador, es un navegador.** Muestra los resultados del motor que tengas configurado, que por defecto es Google: lo que aparece "en Safari" se saca en la capa 1. Apple no tiene un formulario para desindexar resultados de búsqueda, y ofrecer uno sería mentir. Su canal de privacidad sirve para otra cosa —oponerse al uso de URLs para entrenar modelos— y el control de la indexación de Applebot está en manos del dueño del sitio, no del titular de los datos.

Antes de poder copiar o descargar cualquier pieza hay que **acreditar identidad**: declarar la titularidad de los datos y confirmar que se tiene el documento del país para adjuntar, más el poder si actúa un representante. Sin eso los envíos quedan bloqueados en la UI, porque sin legitimación el destinatario rechaza el pedido de entrada.

Cada pieza lleva visible **"Pedido amparado en \[ley\]"** con el derecho concreto que se ejerce en ese país: supresión en Argentina, eliminación del artículo 18 de la LGPD en Brasil, cancelación de los derechos ARCO en Perú, y eliminación o supresión en Chile según el régimen vigente a la fecha. Nunca se mezclan leyes entre países. Desde el panel se descargan de una vez todas las piezas de un hallazgo o de todos.

El comprobante es parte del flujo, no un extra: cada pieza registra estado, fecha de envío y número de caso, y con esa fecha se calcula cuándo vence el plazo legal y se habilita la capa siguiente. Además se califica al destinatario (*coopera*, *responde a medias*, *se resiste*), no solo al pedido.

## Qué no promete

Notas periodísticas, expedientes judiciales y publicaciones oficiales con base legal propia aparecen listados aparte, con el motivo por el que no se puede prometer bajarlos. Ningún resultado se garantiza: depende del sitio, del buscador y de la autoridad.

## Correrlo local

```bash
npm install
npm run dev
```

Queda en http://localhost:43717 (puerto elegido para no chocar con otros dev servers).

Otros comandos: `npm run build`, `npm start`, `npm run lint`.

## Publicarlo como web propia

Incógnito se publica como sitio de producto: la home y las páginas de respaldo legal son públicas e indexables, y el flujo con sesión queda afuera de los buscadores.

```bash
export INCOGNITO_URL_PUBLICA="https://tu-dominio"
export INCOGNITO_CLAVE_SESION="$(openssl rand -base64 48)"
npm ci
npm run build
npm start
```

### Variables de entorno

| Variable | Obligatoria | Para qué |
| --- | --- | --- |
| `INCOGNITO_URL_PUBLICA` | Sí para publicar | URL absoluta del sitio. De ahí salen el `metadataBase`, el canónico, las URLs de Open Graph, el `sitemap.xml` y el `robots.txt`. Sin ella todo apunta a `http://localhost:43717`, que es lo correcto en desarrollo y está roto en producción |
| `INCOGNITO_CLAVE_SESION` | Sí en producción | Clave de firma HMAC de la cookie de sesión, 32 caracteres o más. En producción la app no arranca sin ella, a propósito: con la clave por defecto del repo cualquiera podría falsificar una sesión |
| `INCOGNITO_CARPETA_DATOS` | No | Carpeta donde se guardan las cuentas y el libro de registro. Por defecto `.datos-demo` en la raíz del proyecto, que sirve para probar pero no para producción: apuntala a un volumen persistente y fuera del árbol de build |
| `GOOGLE_CSE_API_KEY` + `GOOGLE_CSE_CX` | No | Activan la búsqueda real con Google Programmable Search. Sin ellas el escaneo usa el conector de demostración y lo avisa en pantalla |
| `OPENAI_API_KEY` | No | Deja que un modelo reescriba la redacción de las piezas. Nunca decide el trámite |
| `OPENAI_MODEL` | No | Modelo a usar; por defecto `gpt-4o-mini` |

### SEO e indexación

- Los metadatos base están en `src/app/layout.tsx` y los datos del sitio (nombre, título, descripción, idioma, rutas) en `src/lib/sitio.ts`.
- `src/app/sitemap.ts` publica la home, `/leyes` y una entrada por país, tomando los slugs del relevamiento legal: si se agrega un país, aparece solo.
- `src/app/robots.ts` permite el resto y bloquea `/ingresar`, `/crear-cuenta`, `/historial` y `/api`. Una ruta privada nueva se suma a `RUTAS_PRIVADAS` en `src/lib/sitio.ts` y queda bloqueada y fuera del sitemap sin tocar nada más. Conviene además que esas páginas declaren `robots: { index: false, follow: false }` en su propio `metadata`.
- La imagen que se ve al compartir el enlace se genera en el build con la API de imágenes de Next (`src/app/opengraph-image.tsx` y `src/app/twitter-image.tsx`, dibujadas en `src/lib/og/tarjeta-social.tsx`). No hay binarios en el repo.
- Cada página pública nueva tiene que declarar su `alternates.canonical`: el canónico del layout es el de la home.
- El layout agrega la marca al título con la plantilla `%s — Incógnito`, así que el `title` de cada página va **sin** `— Incógnito`: si lo repite, queda duplicado. Las páginas que hoy lo repiten son `/leyes`, `/leyes/[pais]`, `/ingresar`, `/crear-cuenta` e `/historial`; sacarles el sufijo es un cambio de una línea en cada `metadata`.
- Las páginas de `/leyes` definen su propio `openGraph` sin `images`, y eso reemplaza la imagen heredada del layout: para que también se vean al compartirlas, les falta `images: ["/opengraph-image"]` o un `opengraph-image` propio del segmento.

### Antes de publicar

- `npm run build` y `npm run lint` en verde.
- `INCOGNITO_URL_PUBLICA` apuntando al dominio final, y `curl https://tu-dominio/robots.txt` y `/sitemap.xml` devolviendo ese dominio y no localhost.
- Compartir el enlace en algún chat y ver que el título, la descripción y la imagen aparezcan.
- `INCOGNITO_CLAVE_SESION` propia y distinta de la de cualquier otro entorno; el sitio servido por HTTPS, porque la cookie de sesión se marca `secure` en producción.
- `INCOGNITO_CARPETA_DATOS` en un volumen que sobreviva a los deploys, y respaldado: ahí vive el libro de registro de pedidos.
- Revisar que ningún copy nuevo viole las dos reglas de la sección siguiente.

## Reglas de copy del proyecto

Dos prohibiciones que valen para toda la interfaz, los metadatos, los textos generados y cualquier material de difusión:

1. **No prometer borrado de internet.** Nada de "te borramos de internet", "desaparecés de la web" ni equivalentes. El buscador desindexa y el sitio de origen despublica; ninguna de las dos cosas hace desaparecer el contenido de internet. La formulación válida es la del hero: limpiar la huella en los buscadores hasta donde la ley del país lo permite, con el aviso de "Qué te garantizamos y qué no".
2. **No invocar el derecho al olvido europeo ni el RGPD.** La cobertura son Argentina, Brasil, Perú y Chile, y ninguno reconoce un derecho al olvido frente a buscadores. Cada pedido se funda en la ley local —Ley 25.326, LGPD, Ley 29733, Ley 19.628 o 21.719—; citar el RGPD sería invocar una norma que no aplica y que además delata copiar un producto europeo.

## Sin credenciales funciona igual

| Pieza | Sin claves | Con claves |
| --- | --- | --- |
| Escaneo | Conector de demostración: resultados plausibles y determinísticos sobre dominios ficticios (`*.ejemplo.ar`, `*.exemplo.br`, …), con los tipos de sitio reales de cada país | `GOOGLE_CSE_API_KEY` + `GOOGLE_CSE_CX` activan Google Programmable Search |
| Motor de pedidos | Reglas por país | `OPENAI_API_KEY` (y opcionalmente `OPENAI_MODEL`) hacen que un modelo reescriba solo la redacción |

Los dominios de la demo son ficticios a propósito: alcanza para mostrar el flujo sin acusar a un sitio real de exponer datos que nadie verificó. Si el conector real falla, la app cae al de demostración y lo avisa en pantalla.

El modelo de lenguaje nunca decide el trámite: parte del paquete que armó el motor de reglas y solo reescribe el texto, con instrucción explícita de no inventar artículos, plazos ni normas.

## Base legal

El contenido por país sale del relevamiento legal del proyecto (autoridad de aplicación, canal oficial, plazo del responsable, criterio jurisprudencial sobre desindexación y documentación habitual), citando normas por nombre y número sin transcribir articulado, con el enlace a la fuente oficial en cada solicitud. Puntos que el producto refleja explícitamente:

- **CUIL/CUIT en Argentina y RUC de persona natural en Perú contienen el documento completo.** Encontrar uno de esos números indexado es encontrar el DNI indexado, y el pedido se plantea así.
- **El RG brasileño no tiene formato nacional único**, así que no se valida con una regla única; la llave presente y futura es el CPF.
- **El dígito verificador del DNI peruano no se calcula**: se lee del documento.
- **Chile cambia de régimen**: Ley 19.628 hasta el 30/11/2026 y Ley 21.719 desde el 01/12/2026, con el aviso de que hay un proyecto para postergarla.

Nada de esto es asesoramiento legal y la UI lo dice en la pantalla principal y en cada solicitud.

## Minimización

Al pedir identificadores locales el producto pasa a ser responsable del tratamiento bajo las cuatro leyes que documenta. Por eso cada campo del formulario explica para qué canal concreto se usa, los opcionales están marcados, no se persiste nada en el servidor y el seguimiento de las solicitudes vive en el `localStorage` del navegador.

## Estructura

```
src/
  app/
    api/escanear/route.ts      valida la consulta y llama al conector
    api/solicitudes/route.ts   recibe los hallazgos marcados y arma las capas
    page.tsx                   ciclo del servicio, las siete capas, avisos y planes
    layout.tsx                 metadatos del sitio, Open Graph, idioma y tema
    leyes/                     respaldo legal público: índice y una página por país
    robots.ts, sitemap.ts      qué se indexa y qué queda fuera
    opengraph-image.tsx        imagen para compartir el enlace, generada en el build
  components/
    asistente.tsx              orquesta escaneo, triaje y seguimiento
    formulario-escaneo.tsx     escaneo
    lista-hallazgos.tsx        triaje, con URL manual
    panel-solicitudes.tsx      panel por capas, con comprobante y plazos
    planes.tsx                 qué incluye cada plan, sin precios
  lib/
    sitio.ts                   nombre, título, descripción, idioma y rutas públicas y privadas
    og/                        dibujo de la tarjeta social
    dominio/                   países, identificadores, sensibilidad, plazos y la máquina de capas
    buscadores/                interfaz de conector, demo y Google CSE
    motor/                     interfaz, armado de las capas, reglas por país y capa de IA
```

Stack: Next.js 16 (App Router), TypeScript, Tailwind CSS 4 y shadcn/ui.
