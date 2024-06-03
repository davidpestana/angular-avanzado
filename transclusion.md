### Aplicación de Notas con Transclusión en Angular

**Objetivo del Ejercicio:**
Crear una aplicación de "Notas" que permita a los usuarios crear, editar y visualizar notas. Las notas deben poder ser categorizadas y mostradas en diferentes secciones, utilizando la transclusión para personalizar la presentación de las notas en diferentes contextos.

**Requisitos:**

1. **Componentes:**
   - `app-root`: El componente raíz que contiene la estructura básica de la aplicación.
   - `note-list`: Un componente que muestra una lista de notas.
   - `note-item`: Un componente que muestra una nota individual, utilizando transclusión para personalizar el contenido de la nota.
   - `category`: Un componente que agrupa las notas por categorías, utilizando transclusión para personalizar la cabecera y el pie de la categoría.
   - `note-editor`: Un componente para crear y editar notas.

2. **Funcionalidades:**
   - Crear y editar notas con título, contenido y categoría.
   - Mostrar las notas agrupadas por categoría.
   - Personalizar la presentación de las notas y las categorías utilizando transclusión.

3. **Uso de la Transclusión:**
   - En el componente `note-item`, usar transclusión para permitir que el contenido de la nota sea personalizado.
   - En el componente `category`, usar transclusión para permitir que la cabecera y el pie de la categoría sean personalizados.
   
### Pasos para el Desarrollo:

1. **Configuración Inicial:**
   - Configurar un nuevo proyecto Angular.
   - Crear los componentes básicos: `note-list`, `note-item`, `category`, y `note-editor`.

2. **Desarrollo de Componentes:**
   - **note-item**: Implementar el componente para mostrar una nota individual. Utilizar `<ng-content>` para permitir la personalización del contenido de la nota.
     ```html
     <!-- note-item.component.html -->
     <div class="note-item">
       <ng-content></ng-content>
     </div>
     ```
   - **category**: Implementar el componente para agrupar notas por categoría. Utilizar `<ng-content select>` para personalizar la cabecera y el pie de la categoría.
     ```html
     <!-- category.component.html -->
     <div class="category">
       <div class="category-header">
         <ng-content select="[category-header]"></ng-content>
       </div>
       <div class="category-body">
         <ng-content></ng-content>
       </div>
       <div class="category-footer">
         <ng-content select="[category-footer]"></ng-content>
       </div>
     </div>
     ```

3. **Integración de Componentes:**
   - En el componente `note-list`, utilizar `note-item` y `category` para organizar y mostrar las notas.
   - Implementar el componente `note-editor` para permitir la creación y edición de notas.

4. **Estilización y Personalización:**
   - Añadir CSS para estilizar los componentes y hacer que la aplicación sea visualmente atractiva.
   - Proveer ejemplos de personalización utilizando transclusión.

### Ejemplo de Uso:

```html
<!-- app.component.html -->
<category>
  <div category-header>
    <h2>Categoría 1</h2>
  </div>
  
  <note-item>
    <h3>Título de la Nota 1</h3>
    <p>Contenido de la Nota 1</p>
  </note-item>
  
  <note-item>
    <h3>Título de la Nota 2</h3>
    <p>Contenido de la Nota 2</p>
  </note-item>
  
  <div category-footer>
    <p>Pie de categoría</p>
  </div>
</category>
```

### Evaluación:

- Una aplicación funcional con la estructura y componentes descritos.
- Uso correcto de la transclusión para personalizar el contenido de las notas y las categorías.
- Código limpio y bien comentado.
- Una demostración de la aplicación en funcionamiento.

