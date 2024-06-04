## Laboratorio: Refactorización del Servicio de Personajes

### Descripción

En este laboratorio, vamos a refactorizar el servicio de personajes (`CharacterService`) en nuestra aplicación Angular. Utilizaremos `BehaviorSubject` para almacenar la lista de personajes y `async` pipes para gestionar los datos asincrónicos en los componentes. También implementaremos otras técnicas para mejorar la eficiencia y la legibilidad del código.

### Objetivos

1. Refactorizar el servicio de personajes para utilizar `BehaviorSubject`.
2. Utilizar `async` pipes en los componentes para gestionar los datos asincrónicos.
3. Implementar técnicas para mejorar la eficiencia y legibilidad del código.

### Paso 1: Crear el Servicio de Personajes

1. Crea un nuevo servicio llamado `CharacterService` utilizando el CLI de Angular:
    ```sh
    ng generate service character
    ```

2. Abre el archivo `character.service.ts` y reemplaza el contenido con el siguiente código:
    ```typescript
    import { HttpClient } from '@angular/common/http';
    import { Injectable } from '@angular/core';
    import { BehaviorSubject, map } from 'rxjs';

    @Injectable({
      providedIn: 'root'
    })
    export class CharacterService {
      private characters$ = new BehaviorSubject<any[]>([]);
      private page = 1;

      constructor(private http: HttpClient) { }

      get characters() {
        return this.characters$.asObservable();
      }

      load() {
        this.http.get(`https://rickandmortyapi.com/api/character/?page=${this.page}`)
          .pipe(map((data: any) => data.results))
          .subscribe(results => {
            this.characters$.next([...this.characters$.getValue(), ...results]);
            this.page++;
          });
      }

      clear() {
        this.page = 1;
        this.characters$.next([]);
        this.load();
      }
    }
    ```

### Paso 2: Utilizar `async` Pipe en los Componentes

1. En los componentes donde necesites mostrar la lista de personajes, utiliza el `async` pipe en el template para suscribirte al observable del servicio `characters`:
    ```html
    <ng-container *ngIf="(characterService.characters | async) as characters">
      <!-- Aquí va tu lógica para mostrar los personajes -->
    </ng-container>
    ```

### Paso 3: Implementar Otras Técnicas

1. Para obtener la longitud de la lista de personajes, puedes usar una función getter en el servicio:
    ```typescript
    get characterCount() {
      return this.characters$.pipe(map(data => data.length));
    }
    ```

2. Para cargar más personajes cuando se haga scroll, puedes escuchar el evento de scroll en el componente y llamar al método `load()` del servicio cuando sea necesario.

### Paso 4: Ejecutar el Servicio

1. En el componente donde quieras cargar los personajes, inyecta el servicio `CharacterService` y llama al método `load()` en el `ngOnInit()` para cargar los personajes inicialmente.

### Conclusiones

Con esta refactorización, hemos mejorado la eficiencia y la legibilidad del servicio de personajes en nuestra aplicación Angular. Ahora utilizamos `BehaviorSubject` para almacenar y emitir la lista de personajes, y `async` pipes para gestionar los datos asincrónicos en los componentes. Además, hemos implementado otras técnicas para mejorar la funcionalidad y el rendimiento general de la aplicación.