### Descripción del Laboratorio

En este laboratorio, crearemos una aplicación simple de contador en Angular utilizando NGRX para la gestión del estado y Local Storage para la persistencia de datos. Aprenderemos a configurar y utilizar NGRX, y a implementar una estrategia para guardar y recuperar el estado del contador desde Local Storage. Al finalizar, tendrás una aplicación funcional que mantendrá el estado del contador incluso después de reiniciar la aplicación.

### Paso 1: Crear un nuevo proyecto Angular

Primero, crea un nuevo proyecto Angular si no lo has hecho ya. Abre tu terminal y ejecuta:

```sh
ng new counter-app
cd counter-app
```

### Paso 2: Instalar NGRX

Instala las dependencias de NGRX:

```sh
ng add @ngrx/store@latest
ng add @ngrx/store-devtools@latest
ng add @ngrx/schematics@latest
```

### Paso 3: Configurar NGRX

Crea los archivos necesarios para NGRX:

```sh
ng generate store AppState --root --module app.module.ts
ng generate action counter --group
ng generate reducer counter --reducersPath store/reducers
ng generate effect counter --root
```

### Paso 4: Definir el estado y las acciones

En `src/app/store/actions/counter.actions.ts`, define las acciones:

```typescript
import { createAction, props } from '@ngrx/store';

export const increment = createAction('[Counter] Increment');
export const decrement = createAction('[Counter] Decrement');
export const reset = createAction('[Counter] Reset');
export const loadState = createAction('[Counter] Load State', props<{ count: number }>());
```

### Paso 5: Crear el reductor

En `src/app/store/reducers/counter.reducer.ts`, define el reductor:

```typescript
import { createReducer, on } from '@ngrx/store';
import * as CounterActions from '../actions/counter.actions';

export interface State {
  count: number;
}

export const initialState: State = {
  count: 0,
};

const counterReducer = createReducer(
  initialState,
  on(CounterActions.increment, state => ({ ...state, count: state.count + 1 })),
  on(CounterActions.decrement, state => ({ ...state, count: state.count - 1 })),
  on(CounterActions.reset, state => ({ ...state, count: 0 })),
  on(CounterActions.loadState, (state, { count }) => ({ ...state, count }))
);

export function reducer(state: State | undefined, action: Action) {
  return counterReducer(state, action);
}
```

### Paso 6: Configurar efectos

En `src/app/store/effects/counter.effects.ts`, define los efectos:

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { tap } from 'rxjs/operators';
import * as CounterActions from '../actions/counter.actions';

@Injectable()
export class CounterEffects {
  constructor(private actions$: Actions) {}

  persistCount$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(CounterActions.increment, CounterActions.decrement, CounterActions.reset),
        tap(action => {
          const state = JSON.parse(localStorage.getItem('counterState') || '{"count": 0}');
          if (action.type === CounterActions.increment.type) {
            state.count += 1;
          } else if (action.type === CounterActions.decrement.type) {
            state.count -= 1;
          } else if (action.type === CounterActions.reset.type) {
            state.count = 0;
          }
          localStorage.setItem('counterState', JSON.stringify(state));
        })
      ),
    { dispatch: false }
  );
}
```

### Paso 7: Crear el componente del contador

Crea un componente de contador:

```sh
ng generate component counter
```

En `src/app/counter/counter.component.ts`, define el componente:

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as CounterActions from '../store/actions/counter.actions';
import { AppState } from '../store/reducers';

@Component({
  selector: 'app-counter',
  templateUrl: './counter.component.html',
  styleUrls: ['./counter.component.css']
})
export class CounterComponent implements OnInit {
  count$: Observable<number>;

  constructor(private store: Store<AppState>) {
    this.count$ = this.store.select(state => state.counter.count);
  }

  ngOnInit(): void {
    const savedState = JSON.parse(localStorage.getItem('counterState') || '{"count": 0}');
    this.store.dispatch(CounterActions.loadState({ count: savedState.count }));
  }

  increment() {
    this.store.dispatch(CounterActions.increment());
  }

  decrement() {
    this.store.dispatch(CounterActions.decrement());
  }

  reset() {
    this.store.dispatch(CounterActions.reset());
  }
}
```

### Paso 8: Crear la plantilla del componente de contador

En `src/app/counter/counter.component.html`, define el HTML del componente:

```html
<div class="counter">
  <h1>Counter: {{ count$ | async }}</h1>
  <button mat-raised-button color="primary" (click)="increment()">Increment</button>
  <button mat-raised-button color="warn" (click)="decrement()">Decrement</button>
  <button mat-raised-button (click)="reset()">Reset</button>
</div>
```

### Paso 9: Integrar el componente en la aplicación

En `src/app/app.component.html`, incluye el componente del contador:

```html
<app-counter></app-counter>
```

### Paso 10: Ejecutar la aplicación

Finalmente, ejecuta la aplicación:

```sh
ng serve
```

Abre tu navegador en `http://localhost:4200` para ver la aplicación del contador en acción.

¡Y eso es todo! Has creado una aplicación simple de contador en Angular utilizando NGRX para la gestión del estado y Local Storage para la persistencia de datos. Ahora, el estado del contador se mantendrá incluso después de reiniciar la aplicación.
