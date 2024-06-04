

¡Claro! Aquí tienes el laboratorio completo con una introducción y objetivos claros. Puedes copiar y pegar todo el contenido para guiar a los alumnos.

---

## Laboratorio: Creación de una Aplicación Angular con Lazy Loading y el Patrón de Presentación-Contenedor

### Introducción

En este laboratorio, aprenderemos a crear una aplicación Angular con dos páginas: una para listar personajes y otra para listar localizaciones de la serie Rick y Morty. Utilizaremos lazy loading para cargar los módulos de forma eficiente y aplicaremos el patrón de presentación-contenedor para organizar nuestro código. Además, estilizaremos nuestra aplicación para que tenga una apariencia atractiva.

### Objetivos

1. **Crear una aplicación Angular** que utilice lazy loading para cargar módulos de forma eficiente.
2. **Implementar el patrón de presentación-contenedor** para separar la lógica de negocio de la presentación.
3. **Consumir la API de Rick y Morty** para obtener datos de personajes y localizaciones.
4. **Aplicar estilos CSS** para mejorar la apariencia de la aplicación.

### Paso 1: Configuración del Proyecto Angular

1. **Instalación de Angular CLI**:
    ```sh
    npm install -g @angular/cli
    ```

2. **Creación del Proyecto**:
    ```sh
    ng new RickAndMortyApp
    cd RickAndMortyApp
    ```

3. **Instalación de Angular Material (opcional, para mejor UI)**:
    ```sh
    ng add @angular/material
    ```

### Paso 2: Creación de Módulos y Componentes

1. **Crear módulos para Personajes y Localización**:
    ```sh
    ng generate module features/characters --route characters --module app.module
    ng generate module features/locations --route locations --module app.module
    ```

2. **Crear componentes para la presentación y el contenedor en cada módulo**:
    ```sh
    ng generate component features/characters/container
    ng generate component features/characters/presenter
    ng generate component features/locations/container
    ng generate component features/locations/presenter
    ```

### Paso 3: Configuración de Lazy Loading y Rutas

Edita `src/app/app-routing.module.ts` para incluir las rutas:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'characters', loadChildren: () => import('./features/characters/characters.module').then(m => m.CharactersModule) },
  { path: 'locations', loadChildren: () => import('./features/locations/locations.module').then(m => m.LocationsModule) },
  { path: '', redirectTo: '/characters', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Paso 4: Implementación del Servicio para la API de Rick y Morty

1. **Crear un servicio compartido**:
    ```sh
    ng generate service shared/rickAndMorty
    ```

2. **Implementar el servicio** (`src/app/shared/rick-and-morty.service.ts`):
    ```typescript
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    import { Observable } from 'rxjs';

    @Injectable({
      providedIn: 'root'
    })
    export class RickAndMortyService {
      private baseUrl = 'https://rickandmortyapi.com/api';

      constructor(private http: HttpClient) { }

      getCharacters(): Observable<any> {
        return this.http.get(`${this.baseUrl}/character`);
      }

      getLocations(): Observable<any> {
        return this.http.get(`${this.baseUrl}/location`);
      }
    }
    ```

### Paso 5: Implementación de los Contenedores y Presentadores

1. **Contenedor de Personajes** (`src/app/features/characters/container/container.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-characters-container',
      templateUrl: './container.component.html',
      styleUrls: ['./container.component.css']
    })
    export class CharactersContainerComponent implements OnInit {
      characters$ = this.rickAndMortyService.getCharacters();

      constructor(private rickAndMortyService: RickAndMortyService) { }

      ngOnInit(): void {}
    }
    ```

    ```html
    <!-- src/app/features/characters/container/container.component.html -->
    <div class="container">
      <h2>Characters</h2>
      <app-characters-presenter [characters]="characters$ | async"></app-characters-presenter>
    </div>
    ```

2. **Presentador de Personajes** (`src/app/features/characters/presenter/presenter.component.ts`):
    ```typescript
    import { Component, Input } from '@angular/core';

    @Component({
      selector: 'app-characters-presenter',
      templateUrl: './presenter.component.html',
      styleUrls: ['./presenter.component.css']
    })
    export class CharactersPresenterComponent {
      @Input() characters: any;
    }
    ```

    ```html
    <!-- src/app/features/characters/presenter/presenter.component.html -->
    <div class="character-grid" *ngIf="characters.results">
      <div class="character-card" *ngFor="let character of characters.results">
        <h3>{{ character.name }}</h3>
        <img [src]="character.image" alt="{{ character.name }}">
      </div>
    </div>
    ```

3. **Contenedor de Localizaciones** (`src/app/features/locations/container/container.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-locations-container',
      templateUrl: './container.component.html',
      styleUrls: ['./container.component.css']
    })
    export class LocationsContainerComponent implements OnInit {
      locations$ = this.rickAndMortyService.getLocations();

      constructor(private rickAndMortyService: RickAndMortyService) { }

      ngOnInit(): void {}
    }
    ```

    ```html
    <!-- src/app/features/locations/container/container.component.html -->
    <div class="container">
      <h2>Locations</h2>
      <app-locations-presenter [locations]="locations$ | async"></app-locations-presenter>
    </div>
    ```

4. **Presentador de Localizaciones** (`src/app/features/locations/presenter/presenter.component.ts`):
    ```typescript
    import { Component, Input } from '@angular/core';

    @Component({
      selector: 'app-locations-presenter',
      templateUrl: './presenter.component.html',
      styleUrls: ['./presenter.component.css']
    })
    export class LocationsPresenterComponent {
      @Input() locations: any;
    }
    ```

    ```html
    <!-- src/app/features/locations/presenter/presenter.component.html -->
    <div class="location-list" *ngIf="locations.results">
      <div class="location-card" *ngFor="let location of locations.results">
        <h3>{{ location.name }}</h3>
        <p>{{ location.type }} - {{ location.dimension }}</p>
      </div>
    </div>
    ```

### Paso 6: Configuración del Módulo y Plantillas

1. **Configuración de módulos**:
    ```typescript
    // src/app/features/characters/characters.module.ts
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { RouterModule, Routes } from '@angular/router';
    import { CharactersContainerComponent } from './container/container.component';
    import { CharactersPresenterComponent } from './presenter/presenter.component';

    const routes: Routes = [
      { path: '', component: CharactersContainerComponent }
    ];

    @NgModule({
      declarations: [
        CharactersContainerComponent,
        CharactersPresenterComponent
      ],
      imports: [
        CommonModule,
        RouterModule.forChild(routes)
      ]
    })
    export class CharactersModule { }
    ```

    ```typescript
    // src/app/features/locations/locations.module.ts
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { RouterModule, Routes } from '@angular/router';
    import { LocationsContainerComponent } from './container/container.component';
    import { LocationsPresenterComponent } from './presenter/presenter.component';

    const routes: Routes = [
      { path: '', component: LocationsContainerComponent }
    ];

    @NgModule({
      declarations: [
        LocationsContainerComponent,
        LocationsPresenterComponent
      ],
      imports: [
        CommonModule,
        RouterModule.forChild(routes)
      ]
    })
    export class LocationsModule { }
    ```

2. **Configuración del `AppModule`**:
    ```typescript
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';
    import { HttpClientModule } from '@angular/common/http';
    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';

    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        HttpClientModule


      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

### Paso 7: Estilización con CSS

1. **Estilos globales** (`src/styles.css`):
    ```css
    body {
      font-family: 'Roboto', sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f5f5f5;
    }

    .container {
      padding: 20px;
    }

    h2 {
      text-align: center;
      margin-bottom: 20px;
    }

    .character-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 20px;
      padding: 20px;
    }

    .character-card {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      text-align: center;
      padding: 20px;
      transition: transform 0.3s;
    }

    .character-card:hover {
      transform: translateY(-10px);
    }

    .character-card img {
      border-radius: 50%;
      width: 100px;
      height: 100px;
      object-fit: cover;
      margin-bottom: 10px;
    }

    .location-list {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    .location-card {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      padding: 20px;
      transition: transform 0.3s;
    }

    .location-card:hover {
      transform: translateY(-10px);
    }
    ```

### Paso 8: Ejecución del Proyecto

Inicia el servidor de desarrollo:
```sh
ng serve
```

Abre tu navegador y navega a `http://localhost:4200/characters` para ver la lista de personajes y `http://localhost:4200/locations` para ver la lista de localizaciones.

### Conclusión

Con esto, has creado una aplicación Angular con lazy loading, aplicando el patrón de presentación-contenedor y añadiendo estilos CSS para mejorar la apariencia. Puedes seguir mejorando el proyecto agregando más funcionalidades, estilos y manejo de errores.