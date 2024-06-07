### Descripción del Laboratorio

En este laboratorio, crearemos una aplicación de lista de tareas ("To-Do") en Angular utilizando Angular Material para la interfaz de usuario, NGRX para la gestión del estado y Firebase para la persistencia de datos. A través de este proceso, aprenderemos a configurar y utilizar NGRX, enfocándonos en los efectos (effects) para manejar operaciones asíncronas como la comunicación con Firebase. El laboratorio incluye la configuración inicial de Firebase, la integración de AngularFire en el proyecto Angular, la definición de acciones, reductores y efectos en NGRX, y la implementación de un componente de lista de tareas interactivo. Al finalizar, tendrás una aplicación funcional que te permitirá gestionar tareas almacenadas en Firestore, el sistema de base de datos en tiempo real de Firebase.

### Paso 1: Crear un nuevo proyecto Angular

Primero, crea un nuevo proyecto Angular si no lo has hecho ya. Abre tu terminal y ejecuta:

```sh
ng new todo-app
cd todo-app
```

### Paso 2: Instalar Angular Material, NGRX y AngularFire

Instala Angular Material, las dependencias de NGRX y AngularFire (para Firebase):

```sh
ng add @angular/material
npm install @ngrx/store @ngrx/effects @ngrx/store-devtools
ng add @angular/fire
```

### Paso 3: Configurar Firebase

1. **Crear un proyecto en Firebase:**
   - Ve a [Firebase Console](https://console.firebase.google.com/).
   - Haz clic en "Add project" y sigue las instrucciones para crear un nuevo proyecto de Firebase.

2. **Agregar una aplicación web a Firebase:**
   - En tu proyecto de Firebase, ve a "Project settings" y selecciona "Add app".
   - Selecciona la opción "Web" y sigue las instrucciones para registrar tu aplicación web.
   - Obtendrás un objeto de configuración de Firebase. Guárdalo, lo necesitarás para configurar AngularFire.

3. **Configurar Firestore:**
   - En el menú de Firebase, ve a "Firestore Database".
   - Haz clic en "Create database" y selecciona el modo "Production".

### Paso 4: Configurar AngularFire en tu proyecto

En `src/environments/environment.ts`, agrega tu configuración de Firebase:

```typescript
export const environment = {
  production: false,
  firebase: {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
  }
};
```

En `src/app/app.module.ts`, importa y configura AngularFire:

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { AngularFireModule } from '@angular/fire';
import { AngularFirestoreModule } from '@angular/fire/firestore';
import { environment } from '../environments/environment';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    AngularFireModule.initializeApp(environment.firebase),
    AngularFirestoreModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Paso 5: Configurar NGRX

Crea los archivos necesarios para NGRX:

```sh
ng generate store AppState --root --module app.module.ts
ng generate action todo --group
ng generate reducer todo --reducersPath store/reducers
ng generate effect todo --root
```

### Paso 6: Definir el estado y las acciones

En `src/app/store/actions/todo.actions.ts`, define las acciones:

```typescript
import { createAction, props } from '@ngrx/store';
import { Todo } from '../models/todo.model';

export const loadTodos = createAction('[Todo] Load Todos');
export const loadTodosSuccess = createAction('[Todo] Load Todos Success', props<{ todos: Todo[] }>());
export const loadTodosFailure = createAction('[Todo] Load Todos Failure', props<{ error: any }>());

export const addTodo = createAction('[Todo] Add Todo', props<{ todo: Todo }>());
export const addTodoSuccess = createAction('[Todo] Add Todo Success', props<{ todo: Todo }>());
export const addTodoFailure = createAction('[Todo] Add Todo Failure', props<{ error: any }>());

export const deleteTodo = createAction('[Todo] Delete Todo', props<{ id: string }>());
export const deleteTodoSuccess = createAction('[Todo] Delete Todo Success', props<{ id: string }>());
export const deleteTodoFailure = createAction('[Todo] Delete Todo Failure', props<{ error: any }>());
```

### Paso 7: Crear el modelo de Todo

Crea un archivo `src/app/models/todo.model.ts` y define el modelo de Todo:

```typescript
export interface Todo {
  id?: string;
  title: string;
  completed: boolean;
}
```

### Paso 8: Crear el reductor

En `src/app/store/reducers/todo.reducer.ts`, define el reductor:

```typescript
import { createReducer, on } from '@ngrx/store';
import * as TodoActions from '../actions/todo.actions';
import { Todo } from '../../models/todo.model';

export interface State {
  todos: Todo[];
  error: any;
}

export const initialState: State = {
  todos: [],
  error: null
};

const todoReducer = createReducer(
  initialState,
  on(TodoActions.loadTodosSuccess, (state, { todos }) => ({
    ...state,
    todos
  })),
  on(TodoActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error
  })),
  on(TodoActions.addTodoSuccess, (state, { todo }) => ({
    ...state,
    todos: [...state.todos, todo]
  })),
  on(TodoActions.addTodoFailure, (state, { error }) => ({
    ...state,
    error
  })),
  on(TodoActions.deleteTodoSuccess, (state, { id }) => ({
    ...state,
    todos: state.todos.filter(todo => todo.id !== id)
  })),
  on(TodoActions.deleteTodoFailure, (state, { error }) => ({
    ...state,
    error
  }))
);

export function reducer(state: State | undefined, action: Action) {
  return todoReducer(state, action);
}
```

### Paso 9: Configurar efectos

En `src/app/store/effects/todo.effects.ts`, define los efectos:

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { AngularFirestore } from '@angular/fire/firestore';
import { of } from 'rxjs';
import { catchError, map, mergeMap } from 'rxjs/operators';
import * as TodoActions from '../actions/todo.actions';
import { Todo } from '../../models/todo.model';

@Injectable()
export class TodoEffects {

  constructor(
    private actions$: Actions,
    private firestore: AngularFirestore
  ) {}

  loadTodos$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.loadTodos),
    mergeMap(() => this.firestore.collection<Todo>('todos').valueChanges({ idField: 'id' }).pipe(
      map(todos => TodoActions.loadTodosSuccess({ todos })),
      catchError(error => of(TodoActions.loadTodosFailure({ error })))
    ))
  ));

  addTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.addTodo),
    mergeMap(action => this.firestore.collection('todos').add(action.todo).then(() => ({
      ...action.todo,
      id: this.firestore.createId()
    })).then(todo => TodoActions.addTodoSuccess({ todo })).catch(error => of(TodoActions.addTodoFailure({ error }))))
  ));

  deleteTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.deleteTodo),
    mergeMap(action => this.firestore.collection('todos').doc(action.id).delete().then(() => TodoActions.deleteTodoSuccess({ id: action.id })).catch(error => of(TodoActions.deleteTodoFailure({ error }))))
  ));
}
```

### Paso 10: Crear el componente de To-Do

Crea un componente de To-Do:

```sh
ng generate component todo
```

En `src/app/todo/todo.component.ts`, define el componente:

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { Todo } from '../models/todo.model';
import * as TodoActions from '../store/actions/todo.actions';
import { AppState } from '../store/reducers';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.css']
})
export class TodoComponent implements OnInit {
  todos$: Observable<Todo[]>;
  newTodo: string = '';

  constructor(private store: Store<AppState>) {
    this.todos$ = this.store.select(state => state.todo.todos);
  }

  ngOnInit(): void {
    this.store.dispatch(TodoActions.loadTodos());
  }

  addTodo() {
    if (this.newTodo.trim()) {
      const todo: Todo = {
        title: this.newTodo.trim(),
        completed: false
      };
      this.store.dispatch(TodoActions.addTodo({ todo }));
      this.newTodo = '';
    }
  }

  deleteTodo(id: string) {
    this.store.dispatch(TodoActions.deleteTodo({ id }));
  }
}
```

### Paso 11: Crear la plantilla del componente de To-Do

En `src/app/todo/todo.component.html`, define el HTML del componente:

```html
<mat-card>
  <mat-card-content>
    <mat-form-field>
      <input matInput placeholder="New Todo" [(ngModel)]="newTodo">
    </mat-form-field>
    <button mat-raised-button color="primary" (click)="

addTodo()">Add</button>

    <mat-list>
      <mat-list-item *ngFor="let todo of todos$ | async">
        {{ todo.title }}
        <button mat-icon-button color="warn" (click)="deleteTodo(todo.id)">
          <mat-icon>delete</mat-icon>
        </button>
      </mat-list-item>
    </mat-list>
  </mat-card-content>
</mat-card>
```

### Paso 12: Integrar el componente en la aplicación

En `src/app/app.component.html`, incluye el componente de To-Do:

```html
<app-todo></app-todo>
```

### Paso 13: Ejecutar la aplicación

Finalmente, ejecuta la aplicación:

```sh
ng serve
```

Abre tu navegador en `http://localhost:4200` para ver la aplicación de lista de tareas en acción.

¡Y eso es todo! Has creado una aplicación de lista de tareas en Angular usando NGRX para la gestión del estado y Firebase para la persistencia de datos. 
Este laboratorio se centra en los efectos, especialmente en cómo gestionar las operaciones asíncronas con Firebase a través de NGRX.