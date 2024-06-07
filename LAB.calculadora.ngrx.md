### Paso 1: Crear un nuevo proyecto Angular

Primero, crea un nuevo proyecto Angular si no lo has hecho ya. Abre tu terminal y ejecuta:

```sh
ng new angular-calculator
cd angular-calculator
```

### Paso 2: Instalar Angular Material y NGRX

Instala Angular Material y las dependencias de NGRX:

```sh
ng add @angular/material
npm install @ngrx/store @ngrx/effects @ngrx/store-devtools
```

### Paso 3: Configurar Angular Material

En `src/app/app.module.ts`, importa los módulos de Angular Material necesarios. Por ejemplo:

```typescript
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { MatButtonModule } from '@angular/material/button';
import { MatInputModule } from '@angular/material/input';
import { MatGridListModule } from '@angular/material/grid-list';
import { MatCardModule } from '@angular/material/card';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MatButtonModule,
    MatInputModule,
    MatGridListModule,
    MatCardModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Paso 4: Configurar NGRX

Crea los archivos necesarios para NGRX:

```sh
ng generate store AppState --root --module app.module.ts
ng generate action calculator --group
ng generate reducer calculator --reducersPath store/reducers
ng generate effect calculator --root
```

### Paso 5: Definir el estado y las acciones

En `src/app/store/actions/calculator.actions.ts`, define las acciones:

```typescript
import { createAction, props } from '@ngrx/store';

export const addDigit = createAction('[Calculator] Add Digit', props<{ digit: string }>());
export const clear = createAction('[Calculator] Clear');
export const calculate = createAction('[Calculator] Calculate');
```

### Paso 6: Crear el reductor

En `src/app/store/reducers/calculator.reducer.ts`, define el reductor:

```typescript
import { createReducer, on } from '@ngrx/store';
import * as CalculatorActions from '../actions/calculator.actions';

export interface State {
  display: string;
}

export const initialState: State = {
  display: '',
};

const calculatorReducer = createReducer(
  initialState,
  on(CalculatorActions.addDigit, (state, { digit }) => ({
    ...state,
    display: state.display + digit
  })),
  on(CalculatorActions.clear, state => ({
    ...state,
    display: ''
  })),
  on(CalculatorActions.calculate, state => ({
    ...state,
    display: eval(state.display) // Nota: eval tiene riesgos de seguridad, considera usar una biblioteca de cálculo seguro.
  }))
);

export function reducer(state: State | undefined, action: Action) {
  return calculatorReducer(state, action);
}
```

### Paso 7: Configurar efectos (opcional)

Si necesitas efectos, configúralos en `src/app/store/effects/calculator.effects.ts`:

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import * as CalculatorActions from '../actions/calculator.actions';
import { map } from 'rxjs/operators';

@Injectable()
export class CalculatorEffects {

  constructor(private actions$: Actions) {}

  // Ejemplo de efecto, si fuera necesario
  calculate$ = createEffect(() => this.actions$.pipe(
    ofType(CalculatorActions.calculate),
    map(() => {
      // Aquí podrías realizar operaciones asincrónicas si fuera necesario
      return { type: '[Calculator] Calculation Complete' };
    })
  ));
}
```

### Paso 8: Crear el componente de la calculadora

Crea un componente de calculadora:

```sh
ng generate component calculator
```

En `src/app/calculator/calculator.component.ts`, define el componente:

```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { addDigit, clear, calculate } from '../store/actions/calculator.actions';
import { AppState } from '../store/reducers';

@Component({
  selector: 'app-calculator',
  templateUrl: './calculator.component.html',
  styleUrls: ['./calculator.component.css']
})
export class CalculatorComponent {
  display$: Observable<string>;

  constructor(private store: Store<AppState>) {
    this.display$ = this.store.select(state => state.calculator.display);
  }

  addDigit(digit: string) {
    this.store.dispatch(addDigit({ digit }));
  }

  clear() {
    this.store.dispatch(clear());
  }

  calculate() {
    this.store.dispatch(calculate());
  }
}
```

### Paso 9: Crear la plantilla de la calculadora

En `src/app/calculator/calculator.component.html`, define el HTML de la calculadora:

```html
<mat-card>
  <mat-card-content>
    <div>{{ display$ | async }}</div>
    <div>
      <button mat-button (click)="addDigit('1')">1</button>
      <button mat-button (click)="addDigit('2')">2</button>
      <button mat-button (click)="addDigit('3')">3</button>
      <button mat-button (click)="addDigit('+')">+</button>
    </div>
    <div>
      <button mat-button (click)="addDigit('4')">4</button>
      <button mat-button (click)="addDigit('5')">5</button>
      <button mat-button (click)="addDigit('6')">6</button>
      <button mat-button (click)="addDigit('-')">-</button>
    </div>
    <div>
      <button mat-button (click)="addDigit('7')">7</button>
      <button mat-button (click)="addDigit('8')">8</button>
      <button mat-button (click)="addDigit('9')">9</button>
      <button mat-button (click)="addDigit('*')">*</button>
    </div>
    <div>
      <button mat-button (click)="addDigit('0')">0</button>
      <button mat-button (click)="clear()">C</button>
      <button mat-button (click)="calculate()">=</button>
      <button mat-button (click)="addDigit('/')">/</button>
    </div>
  </mat-card-content>
</mat-card>
```

### Paso 10: Integrar el componente en la aplicación

En `src/app/app.component.html`, incluye el componente de la calculadora:

```html
<app-calculator></app-calculator>
```

### Paso 11: Ejecutar la aplicación

Finalmente, ejecuta la aplicación:

```sh
ng serve
```

Abre tu navegador en `http://localhost:4200` para ver la calculadora en acción.