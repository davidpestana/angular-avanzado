### Aplicación de Gestión de Tareas con Transclusión en Angular

**Objetivo del Ejercicio:**
Crear una aplicación de "Gestión de Tareas" que permita a los usuarios crear, editar, eliminar y visualizar tareas. Utilizar la transclusión para personalizar la presentación de las tareas y las categorías.

### Componentes:

1. `app-root`: El componente raíz.
2. `task-list`: Un componente que muestra una lista de tareas.
3. `task-item`: Un componente que muestra una tarea individual, utilizando transclusión para personalizar el contenido de la tarea.
4. `category`: Un componente que agrupa las tareas por categorías, utilizando transclusión para personalizar la cabecera y el pie de la categoría.
5. `task-editor`: Un componente para crear y editar tareas.

### Pasos para el Desarrollo:

#### Paso 1: Configuración Inicial

1. **Configura un nuevo proyecto Angular.**
   ```bash
   ng new task-manager-app
   cd task-manager-app
   ```

2. **Genera los componentes necesarios utilizando el CLI.**
   ```bash
   ng generate component task-list
   ng generate component task-item
   ng generate component category
   ng generate component task-editor
   ```

#### Paso 2: Desarrollo de Componentes

##### 2.1. `task-item` Component

Este componente mostrará una tarea individual y permitirá la transclusión del contenido.

**Archivo: `task-item/task-item.component.html`**
```html
<div class="task-item">
  <ng-content></ng-content>
</div>
```

**Archivo: `task-item/task-item.component.css`**
```css
.task-item {
  border: 1px solid #ddd;
  padding: 10px;
  margin: 5px 0;
  border-radius: 5px;
  background-color: #f9f9f9;
}
```

##### 2.2. `category` Component

Este componente agrupará tareas por categorías y permitirá la transclusión de la cabecera y el pie de la categoría.

**Archivo: `category/category.component.html`**
```html
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

**Archivo: `category/category.component.css`**
```css
.category {
  border: 1px solid #ccc;
  border-radius: 10px;
  margin: 10px 0;
  padding: 10px;
}

.category-header, .category-footer {
  background-color: #f5f5f5;
  padding: 5px;
  margin: -10px -10px 10px;
}

.category-footer {
  margin-top: 10px;
}
```

##### 2.3. `task-editor` Component

Este componente permitirá crear y editar tareas.

**Archivo: `task-editor/task-editor.component.html`**
```html
<div class="task-editor">
  <h2>Create a New Task</h2>
  <form (ngSubmit)="addTask()">
    <div>
      <label for="title">Title:</label>
      <input type="text" id="title" [(ngModel)]="task.title" name="title" required>
    </div>
    <div>
      <label for="description">Description:</label>
      <textarea id="description" [(ngModel)]="task.description" name="description" required></textarea>
    </div>
    <div>
      <label for="category">Category:</label>
      <input type="text" id="category" [(ngModel)]="task.category" name="category" required>
    </div>
    <button type="submit">Add Task</button>
  </form>
</div>
```

**Archivo: `task-editor/task-editor.component.css`**
```css
.task-editor {
  border: 1px solid #ccc;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.task-editor form div {
  margin-bottom: 10px;
}

.task-editor form label {
  display: block;
  font-weight: bold;
}

.task-editor form input,
.task-editor form textarea {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
}
```

**Archivo: `task-editor/task-editor.component.ts`**
```typescript
import { Component, EventEmitter, Output } from '@angular/core';

@Component({
  selector: 'app-task-editor',
  templateUrl: './task-editor.component.html',
  styleUrls: ['./task-editor.component.css']
})
export class TaskEditorComponent {
  @Output() taskAdded = new EventEmitter<any>();

  task = {
    title: '',
    description: '',
    category: ''
  };

  addTask() {
    this.taskAdded.emit(this.task);
    this.task = { title: '', description: '', category: '' }; // Reset form
  }
}
```

##### 2.4. `task-list` Component

Este componente utilizará `task-item` y `category` para organizar y mostrar las tareas.

**Archivo: `task-list/task-list.component.html`**
```html
<app-category>
  <div category-header>
    <h2>Work</h2>
  </div>

  <app-task-item>
    <h3>Finish report</h3>
    <p>Complete the financial report by EOD.</p>
  </app-task-item>

  <app-task-item>
    <h3>Team meeting</h3>
    <p>Discuss project milestones with the team at 2 PM.</p>
  </app-task-item>

  <div category-footer>
    <p>End of Work Tasks</p>
  </div>
</app-category>

<app-category>
  <div category-header>
    <h2>Personal</h2>
  </div>

  <app-task-item>
    <h3>Grocery shopping</h3>
    <p>Buy fruits and vegetables.</p>
  </app-task-item>

  <app-task-item>
    <h3>Exercise</h3>
    <p>Go for a run in the park.</p>
  </app-task-item>

  <div category-footer>
    <p>End of Personal Tasks</p>
  </div>
</app-category>
```

**Archivo: `task-list/task-list.component.css`**
```css
/* Estilos personalizados para el componente task-list, si es necesario */
```

**Archivo: `task-list/task-list.component.ts`**
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-task-list',
  templateUrl: './task-list.component.html',
  styleUrls: ['./task-list.component.css']
})
export class TaskListComponent {
  tasks = [
    { title: 'Finish report', description: 'Complete the financial report by EOD.', category: 'Work' },
    { title: 'Team meeting', description: 'Discuss project milestones with the team at 2 PM.', category: 'Work' },
    { title: 'Grocery shopping', description: 'Buy fruits and vegetables.', category: 'Personal' },
    { title: 'Exercise', description: 'Go for a run in the park.', category: 'Personal' }
  ];
}
```

#### Paso 3: Integrar Todos los Componentes en el Componente Raíz

Finalmente, integra todos los componentes en el componente raíz (`app.component`).

**Archivo: `app/app.component.html`**
```html
<app-task-editor (taskAdded)="onTaskAdded($event)"></app-task-editor>
<app-task-list [tasks]="tasks"></app-task-list>
```

**Archivo: `app/app.component.css`**
```css
/* Estilos personalizados para el componente app, si es necesario */
```

**Archivo: `app/app.component.ts`**
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  tasks = [
    { title: 'Finish report', description: 'Complete the financial report by EOD.', category: 'Work' },
    { title: 'Team meeting', description: 'Discuss project milestones with the team at 2 PM.', category: 'Work' },
    { title: 'Grocery shopping', description: 'Buy fruits and vegetables.', category: 'Personal' },
    { title: 'Exercise', description: 'Go for a run in the park.', category: 'Personal' }
  ];

  onTaskAdded(task: any) {
    this.tasks.push(task);
  }
}
```

#### Paso 4: Estilización y Personalización Final

Puedes añadir estilos globales en `styles.css` para mejorar la apariencia general de la aplicación.

**Archivo: `src/styles.css`**
```css
body {
  font-family: Arial, sans-serif;
  margin: 20px;
  background-color: #f9f9f9;
}

h2 {
  color: #333;
}
```

