## Laboratorio: Refactorización del Servicio de Angular con `BehaviorSubject` para Caché y Paginación

### Descripción

En este laboratorio, vamos a refactorizar el servicio de nuestra aplicación Angular para gestionar la paginación y el caché utilizando `BehaviorSubject`. Esta técnica permitirá que nuestra aplicación consuma menos recursos y mejore la experiencia del usuario al reducir las solicitudes innecesarias a la API.

### Objetivos

1. **Refactorizar el servicio de Angular** para usar `BehaviorSubject` y caché.
2. **Implementar la gestión de la paginación** en el servicio.
3. **Mejorar la eficiencia** de la aplicación al reducir las solicitudes innecesarias a la API.
4. **Mantener la funcionalidad existente** de la aplicación, asegurando que las páginas de personajes y localizaciones sigan funcionando correctamente.

### Paso 1: Crear un Proyecto Angular (Si no tienes uno)

Si aún no tienes el proyecto configurado, sigue estos pasos. De lo contrario, puedes saltar al Paso 2.

1. **Instalación de Angular CLI**:
    ```sh
    npm install -g @angular/cli
    ```

2. **Creación del Proyecto**:
    ```sh
    ng new RickAndMortyApp
    cd RickAndMortyApp
    ```

### Paso 2: Refactorización del Servicio

Vamos a modificar el servicio `RickAndMortyService` para utilizar `BehaviorSubject` y gestionar el caché y la paginación.

1. **Modificar el Servicio** (`src/app/shared/rick-and-morty.service.ts`):
    ```typescript
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    import { BehaviorSubject, Observable } from 'rxjs';
    import { map } from 'rxjs/operators';

    @Injectable({
      providedIn: 'root'
    })
    export class RickAndMortyService {
      private baseUrl = 'https://rickandmortyapi.com/api';
      private charactersSubject = new BehaviorSubject<any>(null);
      private locationsSubject = new BehaviorSubject<any>(null);
      private charactersCache: { [page: number]: any } = {};
      private locationsCache: { [page: number]: any } = {};

      constructor(private http: HttpClient) { }

      getCharacters(page: number = 1): Observable<any> {
        if (this.charactersCache[page]) {
          this.charactersSubject.next(this.charactersCache[page]);
        } else {
          this.http.get(`${this.baseUrl}/character?page=${page}`).subscribe(data => {
            this.charactersCache[page] = data;
            this.charactersSubject.next(data);
          });
        }
        return this.charactersSubject.asObservable();
      }

      getCharacter(id: number): Observable<any> {
        return this.http.get(`${this.baseUrl}/character/${id}`);
      }

      getLocations(page: number = 1): Observable<any> {
        if (this.locationsCache[page]) {
          this.locationsSubject.next(this.locationsCache[page]);
        } else {
          this.http.get(`${this.baseUrl}/location?page=${page}`).subscribe(data => {
            this.locationsCache[page] = data;
            this.locationsSubject.next(data);
          });
        }
        return this.locationsSubject.asObservable();
      }

      getLocation(id: number): Observable<any> {
        return this.http.get(`${this.baseUrl}/location/${id}`);
      }
    }
    ```

### Paso 3: Actualización de los Componentes para Manejar la Paginación

1. **Actualizar el Contenedor de Personajes** (`src/app/features/characters/container/container.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-characters-container',
      templateUrl: './container.component.html',
      styleUrls: ['./container.component.css']
    })
    export class CharactersContainerComponent implements OnInit {
      characters: any;
      currentPage = 1;

      constructor(private rickAndMortyService: RickAndMortyService) { }

      ngOnInit(): void {
        this.loadCharacters();
      }

      loadCharacters(page: number = 1): void {
        this.currentPage = page;
        this.rickAndMortyService.getCharacters(page).subscribe(data => {
          this.characters = data;
        });
      }
    }
    ```

    ```html
    <!-- src/app/features/characters/container/container.component.html -->
    <div class="container">
      <h2>Characters</h2>
      <app-characters-presenter [characters]="characters?.results"></app-characters-presenter>
      <div class="pagination">
        <button (click)="loadCharacters(currentPage - 1)" [disabled]="!characters?.info.prev">Previous</button>
        <button (click)="loadCharacters(currentPage + 1)" [disabled]="!characters?.info.next">Next</button>
      </div>
    </div>
    ```

2. **Actualizar el Contenedor de Localizaciones** (`src/app/features/locations/container/container.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-locations-container',
      templateUrl: './container.component.html',
      styleUrls: ['./container.component.css']
    })
    export class LocationsContainerComponent implements OnInit {
      locations: any;
      currentPage = 1;

      constructor(private rickAndMortyService: RickAndMortyService) { }

      ngOnInit(): void {
        this.loadLocations();
      }

      loadLocations(page: number = 1): void {
        this.currentPage = page;
        this.rickAndMortyService.getLocations(page).subscribe(data => {
          this.locations = data;
        });
      }
    }
    ```

    ```html
    <!-- src/app/features/locations/container/container.component.html -->
    <div class="container">
      <h2>Locations</h2>
      <app-locations-presenter [locations]="locations?.results"></app-locations-presenter>
      <div class="pagination">
        <button (click)="loadLocations(currentPage - 1)" [disabled]="!locations?.info.prev">Previous</button>
        <button (click)="loadLocations(currentPage + 1)" [disabled]="!locations?.info.next">Next</button>
      </div>
    </div>
    ```

### Paso 4: Estilización Adicional

1. **Agregar estilos para la paginación** (`src/styles.css`):
    ```css
    .pagination {
      display: flex;
      justify-content: center;
      margin-top: 20px;
    }

    .pagination button {
      background-color: #007bff;
      color: white;
      border: none;
      padding: 10px 20px;
      margin: 0 5px;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s;
    }

    .pagination button:disabled {
      background-color: #cccccc;
      cursor: not-allowed;
    }

    .pagination button:not(:disabled):hover {
      background-color: #0056b3;
    }
    ```

### Paso 5: Ejecución del Proyecto

Inicia el servidor de desarrollo:
```sh
ng serve
```

Abre tu navegador y navega a `http://localhost:4200/characters` para ver la lista de personajes y `http://localhost:4200/locations` para ver la lista de localizaciones. Usa los botones de paginación para navegar entre las páginas.

### Conclusión

En este laboratorio, hemos refactorizado el servicio de Angular para utilizar `BehaviorSubject` y gestionar el caché y la paginación. Esto mejora la eficiencia de la aplicación al reducir las solicitudes innecesarias a la API, proporcionando una mejor experiencia de usuario.
