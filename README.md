# antoniofunes

Portafolio personal de Antonio Funes. Sitio de una sola pagina (`index.html`) con secciones de Inicio, Proyectos, Camino, Contacto y tema claro/oscuro.

---

## Paleta de colores

| Uso                             | Color          | HEX       |
| ------------------------------- | -------------- | --------- |
| **Background / principal**      | Charcoal       | `#262322` |
| **Texto principal / contraste** | Mint Ice       | `#DDFFF7` |
| **Accent principal**            | Muted Lavender | `#6969B3` |
| **Accent secundario**           | Sage Teal      | `#55917F` |
| **Detalles / neutro**           | Olive Gray     | `#82816D` |

---

## Estructura de archivos

```
index.html                  # Todo el HTML + el JS del carrusel de proyectos
css/styles.css              # Todos los estilos (tema claro/oscuro, carrusel)
resources/
  ├── Perfil3.jpeg          # Foto del hero
  ├── Perfil4.jpeg          # Foto de la seccion about
  ├── glyph-container.svg   # Logo de la navbar
  ├── wordmark-text.svg     # Texto del logo
  ├── vineta.png            # Bullet del tema oscuro
  ├── mateblanco.png        # Bullet del tema claro
  └── projects/
      ├── deportes/         # Capturas del proyecto Deportes
      ├── juegos/           # Capturas del proyecto Juegos
      └── mi-pagina/        # Capturas de Mi pagina Web (aun vacia)
```

---

## Como funcionan las imagenes de proyectos (paso a paso)

### 1. Las capturas viven en carpetas por proyecto

Cada proyecto tiene su carpeta en `resources/projects/<slug>/`:

- `resources/projects/deportes/`
- `resources/projects/juegos/`
- `resources/projects/mi-pagina/`

Dentro se guardan las capturas como `imagen1.png`, `imagen2.png`, etc.
La carpeta `mi-pagina/` todavia esta vacia (solo tiene `.gitkeep`), por eso esa
tarjeta muestra el placeholder "Pronto: capturas de este proyecto".

### 2. Las rutas se registran en un objeto JS (`index.html`)

En `index.html` hay un objeto llamado `projectImages`:

```js
const projectImages = {
  deportes: [
    'resources/projects/deportes/imagen1.png',
    'resources/projects/deportes/imagen2.png',
    'resources/projects/deportes/imagen3.png'
  ],
  juegos: [
    'resources/projects/juegos/imagen1.png',
    'resources/projects/juegos/imagen2.png',
    'resources/projects/juegos/imagen3.png',
    'resources/projects/juegos/imagen4.png',
    'resources/projects/juegos/imagen5.png'
  ],
  'mi-pagina': []   // <- aca se cargan las capturas de mi-pagina
};
```

La clave de cada proyecto (`deportes`, `juegos`, `mi-pagina`) debe coincidir con
el atributo `data-slug` de su tarjeta:

```html
<div class="project-card" data-slug="deportes" ...>
```

### 3. Para agregar capturas a mi-pagina hay que hacer 2 cosas

1. Copiar los archivos de imagen a `resources/projects/mi-pagina/`
   (ej: `imagen1.png`, `imagen2.png`, ...).
2. Completar el array vacio en `index.html`:

```js
'pagina-mia': []  // ANTES (vacio)

'mi-pagina': [
  'resources/projects/mi-pagina/imagen1.png',
  'resources/projects/mi-pagina/imagen2.png'
]                 // DESPUES (con las rutas)
```

Con eso, al hacer clic en la tarjeta "Mi pagina Web" se construye el carrusel
con esas imagenes.

### 4. La funcion `buildPanel()` construye el carrusel

Cada vez que se abre una tarjeta, `buildPanel(banner, slug)`:

1. Lee las rutas con `projectImages[slug]`.
2. Si el array esta vacio, muestra el placeholder
   (icono de carpeta + "Pronto: capturas de este proyecto").
3. Si hay imagenes, genera el HTML del carrusel:

```html
<div class="carousel">
  <div class="carousel-viewport">
    <div class="carousel-track">
      <div class="carousel-slide"><img src="...imagen1.png" ...></div>
      <div class="carousel-slide"><img src="...imagen2.png" ...></div>
      ...
    </div>
    <button class="carousel-btn prev">&#10094;</button>
    <button class="carousel-btn next">&#10095;</button>
  </div>
  <div class="carousel-dots"></div>
</div>
```

Despues conecta las flechas (prev/next), los puntos indicadores
(`carousel-dots`) y el swipe tactil para que el carrusel funcione como el
de Bootstrap: flechas laterales, puntos abajo y navegacion touch.

### 5. La apertura y cierre de tarjetas se controla con `toggleExpanded()`

- Hacer clic o presionar Enter/Espacio en una tarjeta la abre (`class="open"`).
- Al abrir una tarjeta, las demas se cierran (solo una se muestra a la vez).
- El panel se construye SOLO la primera vez que se abre (variable `built`),
  despues se reutiliza para no regenerar el DOM.

### 6. Las imagenes se ven completas y el contenedor no "salta"

Cada captura tiene su propia proporcion (algunas mas anchas, otras mas altas),
por eso se fijo una altura estable para el carrusel:

```css
.carousel-viewport {
  height: clamp(280px, 55vh, 520px);
}

.carousel-track {
  height: 100%;
}

.carousel-slide {
  flex: 0 0 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.carousel-slide img {
  width: 85%;
  height: 100%;
  object-fit: contain;
  border-radius: var(--radius);
  box-shadow: var(--shadow-card);
}
```

Que hace cada cosa:

- **Altura fija (`clamp`)** → el viewport mide entre 280px y 520px (55% del alto
  de la pantalla). Como todas las imagenes viven en un contenedor del mismo
  alto, al cambiar de slide **el tamaño ya no se minimiza ni salta**.
- **Slide con `height: 100%` y flex centrado** → la imagen queda centrada
  (horizontal y verticalmente) aunque sea mas baja o mas alta que el contenedor.
- **Imagen con marco fijo** (`width: 85%` + `height: 100%`) + `object-fit:
  contain` → la imagen SIEMPRE ocupa el mismo cuadro, se escala para verse
  completa y sin recortes. Como el cuadro no depende del tamaño natural de la
  imagen (no usa `auto`), aunque la imagen tarde en cargar o cambie de
  proporcion, **el carrusel no se achica al pasar de slide**.
- **`box-shadow` + `border-radius`** → la captura queda "flotando" sobre el
  fondo del carrusel, con un look mas pulido que el borde plano.

Antes las imagenes usaban `width/height: auto`, por eso al cambiar de slide la
imagen cargaba con su tamaño natural y el carrusel saltaba de alto (efecto
"minimizando").

---

## Como funciona la expansion de las imagenes (carousel stack)

Originalmente el carrusel quedaba recortado dentro de la tarjeta porque:

- `.project-card` tenia `overflow: hidden`.
- `.carousel` tampoco podia salirse de su contenedor.

Los cambios en `css/styles.css` hacen que cada proyecto se expanda en una
direccion distinta al abrirse:

### 1. La tarjeta abierta deja salir su contenido

```css
.project-card.open {
  transform: none;
  overflow: visible;   /* permite que el carrusel se salga de la tarjeta */
  z-index: 3;          /* queda por encima de las tarjetas vecinas */
}

.project-card.open .project-banner,
.project-card.open .carousel {
  overflow: visible;   /* ni el banner ni el carrusel recortan la imagen */
}
```

### 2. La direccion de expansion la decide el JS segun la posicion

En `index.html`, `applyVariant(banner, slug)` agrega una clase distinta
segun el `data-slug` de la tarjeta:

- **`deportes`** (tarjeta de la izquierda) → `carousel--right` → expande **a la derecha**
- **`juegos`** (tarjeta del centro) → `carousel--both` → expande **a ambos lados**
- **`mi-pagina`** → toma `carousel--right` (igual que deportes; aun sin capturas)

```js
if (slug === 'juegos') {
  carousel.classList.add('carousel--both');
} else {
  carousel.classList.add('carousel--right');
}
```

### 3. `resizeCarousel()` calcula el tamaño real contra el viewport

Las tarjetas de los costados NO estan centradas en la pantalla, por eso no
alcanza solo con CSS. `resizeCarousel(banner, slug)` mide la posicion real
de la tarjeta con `getBoundingClientRect()` y setea el ancho del carrusel:

- `carousel--both` → ancho = todo el viewport, empujado desde la posicion
  real de la tarjeta (izquierda y derecha).
- `carousel--right` → ancho = desde el borde izquierdo de la tarjeta hasta
  el borde derecho del viewport (solo se expande a la derecha).

Tambien hay un listener de `resize` que recalcula la medida cuando se
redimensiona la ventana con una tarjeta abierta:

```js
window.addEventListener('resize', function () {
  document.querySelectorAll('.project-card.open .project-banner').forEach(function (banner) {
    const card = banner.closest('.project-card');
    resizeCarousel(banner, card.dataset.slug);
  });
});
```

Gracias a esto las imagenes se ven a lo ancho (efecto tipo "stack" del carrusel
de Bootstrap con `w-100`), en vez de quedar cortadas por el borde de la tarjeta.

---

## Resumen de lo modificado

| Archivo       | Que se cambio                                                        |
| ------------- | -------------------------------------------------------------------- |
| `index.html`  | Se cargaron las 3 rutas de `deportes` y las 5 de `juegos` en `projectImages` |
| `index.html`  | Se agrego `applyVariant()` y `resizeCarousel()` para animar la expansion segun la posicion de cada tarjeta |
| `css/styles.css` | `.project-card.open` permite overflow y z-index; el banner y el carrusel abiertos ya no recortan la imagen |
| `css/styles.css` | `.carousel-viewport` con altura estable (`clamp`) y slides centrados; las imagenes se ven completas (`object-fit: contain`, `max-width: 80%`) sin saltos al cambiar de slide |