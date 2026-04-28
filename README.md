# 📡 RSS Reader

### Aplicación web para agregar y seguir feeds RSS con actualización automática en tiempo real

---

## 📖 Sobre el Proyecto

**RSS Reader** es una Single Page Application (SPA) que permite al usuario agregar múltiples feeds RSS, visualizar sus publicaciones en tiempo real y leer una vista previa de cada artículo sin salir de la página.

El proyecto representa el salto de aplicaciones Node.js de consola a una **aplicación web frontend completa**, donde el reto principal no es solo mostrar datos, sino gestionar un estado de aplicación que cambia constantemente — nuevos feeds, nuevos posts que llegan cada 5 segundos, posts marcados como leídos — y reflejar cada cambio en el DOM de forma reactiva y ordenada.

---

## 🛠️ Tecnologías Utilizadas

![JavaScript](https://img.shields.io/badge/JavaScript-ES2020-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-7952B3?style=for-the-badge&logo=bootstrap&logoColor=white)
![Webpack](https://img.shields.io/badge/Webpack-5-8DD6F9?style=for-the-badge&logo=webpack&logoColor=black)
![Playwright](https://img.shields.io/badge/Playwright-E2E-45ba4b?style=for-the-badge&logo=playwright&logoColor=white)

| Librería      | Rol en el proyecto                                                              |
| ------------- | ------------------------------------------------------------------------------- |
| `axios`       | Peticiones HTTP al proxy para obtener los feeds RSS                             |
| `on-change`   | Observa mutaciones en el estado y dispara el re-render automáticamente          |
| `yup`         | Validación del formulario (URL válida, no duplicada, no vacía)                  |
| `i18next`     | Internacionalización: todos los textos de la UI viven en archivos de traducción |
| `Bootstrap 5` | Estilos y componente Modal para la vista previa de posts                        |
| `Webpack`     | Bundling del proyecto y servidor de desarrollo                                  |
| `Playwright`  | Tests End-to-End que simulan interacciones reales del usuario en el navegador   |

---

## ✨ Características Principales

- **Agregar múltiples feeds RSS** mediante URL desde el formulario
- **Actualización automática** cada 5 segundos: los posts nuevos aparecen sin recargar la página
- **Vista previa en modal**: cada post tiene un botón "Vista previa" que abre título y descripción en un modal de Bootstrap
- **Seguimiento de posts leídos**: los posts visitados cambian visualmente de negrita a gris
- **Validación completa del formulario** con mensajes de error específicos:
  - URL inválida → _"El enlace debe ser una URL válida"_
  - Feed ya agregado → _"El RSS ya existe"_
  - Error de red → _"Network error"_
  - XML malformado → _"invalidXml"_
- **Arquitectura MVC**: separación clara entre estado (`controller.js`), lógica de render (`view.js`) y parseo de datos (`rss.js`)
- **Proxy CORS**: las peticiones pasan por `allorigins.hexlet.app` para evitar restricciones de CORS del navegador

---

## ⚙️ Instalación y Configuración

### Pre-requisitos

- `Node.js` v18 o superior
- `npm` v8 o superior

### Pasos

```bash
# 1. Clona el repositorio
git clone https://github.com/Porico94/Proyecto-Agregador-de-RSS

# 2. Entra al directorio
cd Proyecto-Agregador-de-RSS

# 3. Instala las dependencias
npm install

# 4. Levanta el servidor de desarrollo
npm start
# La app estará disponible en http://localhost:8080
```

### Build para producción

```bash
npm run build
# Los archivos compilados quedan en /dist
```

### Ejecutar los tests E2E

```bash
# Asegúrate de tener el servidor corriendo antes de ejecutar los tests
npm start &
npx playwright test
```

---

## 💡 Aprendizajes

> ### 🏆 El reto técnico clave: gestionar el estado de la aplicación con `on-change`

El problema más difícil del proyecto no fue hacer las peticiones HTTP ni parsear el XML, sino **mantener la interfaz sincronizada con el estado** de forma ordenada. Al principio estaba modificando el DOM directamente desde el controlador cada vez que algo cambiaba (después de cargar un feed, después de recibir posts nuevos, después de abrir un modal). El problema fue que el código se volvió imposible de seguir: había lógica de renderizado mezclada con lógica de negocio en todas partes.

La solución fue adoptar un patrón inspirado en cómo trabajan los frameworks modernos: **separar completamente el estado del render**. El estado de la aplicación vive en un único objeto:

```javascript
const state = {
  feeds: [],
  posts: [],
  form: { status: "filling", error: null },
  uiState: { visitedPosts: new Set() },
};
```

La librería `on-change` envuelve ese objeto y **detecta automáticamente cualquier cambio**. Cuando el estado cambia, dispara una función que decide qué parte del DOM actualizar:

```javascript
const watchedState = onChange(state, (path, value) => {
  render(path, value, watchedState, elements);
  //      ↑ qué cambió   ↑ nuevo valor
});

// Ahora cuando ocurre un cambio en el estado, el render ocurre solo
watchedState.form.status = "success"; // → render() se ejecuta automáticamente
watchedState.posts.unshift(...newPosts); // → la lista de posts se actualiza sola
```

Esto me enseñó el principio fundamental detrás de React y Vue: No manipulamos el DOM directamente, describo qué debería verse dado el estado(state) actual, y dejas que el sistema se encargue del resto. Ese cambio de mentalidad fue el aprendizaje más importante de todo el proyecto.

Un segundo reto fue la **actualización automática de feeds**. Cada feed necesita revisarse cada 5 segundos de forma independiente, comparar los posts nuevos con los existentes (por `link`) y agregar solo los que no están. La solución fue un ciclo recursivo con `setTimeout` que se reinicia solo al finalizar cada ronda de peticiones:

```javascript
// Se llama a sí misma cada 5 segundos indefinidamente
Promise.all(requests).finally(() => {
  setTimeout(() => updateFeeds(watchedState), 5000);
});
```

---

## 🧪 Cobertura de Tests E2E con Playwright

Los tests simulan interacciones reales de un usuario en el navegador, interceptando las peticiones de red con `page.route()` para controlar las respuestas:

| Test                           | Escenario                                                      |
| ------------------------------ | -------------------------------------------------------------- |
| ✅ Agregar feed válido         | Muestra feedback de éxito y lista los posts del feed           |
| ✅ Abrir modal de vista previa | El modal muestra título y descripción del post correcto        |
| ❌ URL inválida                | Muestra error sin hacer ninguna petición HTTP                  |
| ❌ URL duplicada               | Muestra _"El RSS ya existe"_ en el segundo intento             |
| ❌ Error de red                | La petición es abortada y aparece el mensaje de error de red   |
| ❌ XML malformado              | El contenido no es RSS válido y se muestra el error de parsing |

---

## 📂 Estructura del Proyecto

```
rss-reader/
├── src/
│   ├── index.js              # Entry point: inicializa i18n y arranca la app
│   ├── controller.js         # Orquestador: maneja eventos, estado y flujo principal
│   ├── view.js               # Capa de render: actualiza el DOM según el estado
│   ├── watcher.js            # Configura on-change para observar el estado
│   ├── rss.js                # Parser de XML RSS con DOMParser
│   ├── api.js                # Petición HTTP al proxy allorigins
│   ├── validationSchema.js   # Esquema de validación con yup
│   ├── modal.js              # Lógica del modal de Bootstrap
│   ├── updateFeeds.js        # Ciclo de actualización automática cada 5s
│   ├── extractPosts.js       # Extrae posts de un documento XML parseado
│   └── locales/
│       ├── index.js          # Inicialización de i18next
│       └── en.js             # Traducciones en inglés
├── __tests__/
│   └── rss.test.js           # Tests E2E con Playwright
├── index.html                # HTML base (Webpack lo procesa)
├── webpack.config.js         # Configuración de Webpack + loaders
└── package.json
```

---

## 👤 Autor

**Pool Rimari** — Desarrollador Full-stack JavaScript
[![GitHub](https://img.shields.io/badge/GitHub-Porico94-181717?style=flat&logo=github)](https://github.com/Porico94)
