¡Claro! A continuación, te proporciono una guía paso a paso para crear una aplicación en Angular con lazy loading, que muestre dos páginas: una para "Personajes" y otra para "Localización", usando la API de Rick y Morty y aplicando el patrón de presentación-contenedor.

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
    <div *ngIf="characters.results">
      <div *ngFor="let character of characters.results">
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
    <div *ngIf="locations.results">
      <div *ngFor="let location of locations.results">
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

### Paso 7: Ejecución del Proyecto

Inicia el servidor de desarrollo:
```sh
ng serve
```

Abre tu navegador y navega a `http://localhost:4200/characters` para ver la lista de personajes y `http://localhost:4200/locations` para ver la lista de localizaciones.

### Conclusión

Con esto, has creado una aplicación Angular con lazy loading y aplicado el patrón de presentación-contenedor. Puedes seguir mejorando el proyecto agregando más funcionalidades, estilos y manejo de errores.