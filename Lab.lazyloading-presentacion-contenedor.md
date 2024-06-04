### Laboratorio: Creación de una Aplicación Angular con Módulos, Lazy Loading y Patrón Presentacional y Contenedor

En este laboratorio, crearás una aplicación Angular con dos páginas: una para mostrar una lista de personajes y otra para mostrar una lista de localizaciones usando la API de Rick and Morty. Utilizarás módulos, lazy loading y el patrón presentacional y contenedor.

#### Requisitos
- Node.js y npm instalados
- Angular CLI instalado (`npm install -g @angular/cli`)

#### Paso 1: Crear el Proyecto Angular
Primero, crea un nuevo proyecto Angular.

```sh
ng new rick-and-morty-app
cd rick-and-morty-app
```

#### Paso 2: Crear Módulos y Componentes
Crea módulos y componentes para las dos páginas (personajes y localizaciones).

```sh
ng generate module pages/characters --route characters --module app.module
ng generate component pages/characters/characters-list
ng generate component pages/characters/characters-container

ng generate module pages/locations --route locations --module app.module
ng generate component pages/locations/locations-list
ng generate component pages/locations/locations-container
```

#### Paso 3: Crear el Servicio para Obtener Datos de la API
Crea un servicio para interactuar con la API de Rick and Morty.

```sh
ng generate service services/rick-and-morty
```

```typescript
// src/app/services/rick-and-morty.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class RickAndMortyService {
  private baseUrl = 'https://rickandmortyapi.com/api';

  constructor(private http: HttpClient) {}

  getCharacters(): Observable<any> {
    return this.http.get(`${this.baseUrl}/character`);
  }

  getLocations(): Observable<any> {
    return this.http.get(`${this.baseUrl}/location`);
  }
}
```

#### Paso 4: Configurar Rutas en AppRoutingModule
Asegúrate de que las rutas están configuradas para lazy loading.

```typescript
// src/app/app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'characters', loadChildren: () => import('./pages/characters/characters.module').then(m => m.CharactersModule) },
  { path: 'locations', loadChildren: () => import('./pages/locations/locations.module').then(m => m.LocationsModule) },
  { path: '', redirectTo: '/characters', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

#### Paso 5: Configurar Rutas en Módulos de Páginas
Configura las rutas para los módulos de personajes y localizaciones.

```typescript
// src/app/pages/characters/characters-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { CharactersContainerComponent } from './characters-container/characters-container.component';

const routes: Routes = [
  { path: '', component: CharactersContainerComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class CharactersRoutingModule { }
```

```typescript
// src/app/pages/locations/locations-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { LocationsContainerComponent } from './locations-container/locations-container.component';

const routes: Routes = [
  { path: '', component: LocationsContainerComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class LocationsRoutingModule { }
```

#### Paso 6: Implementar los Componentes Contenedores
Implementa la lógica en los componentes contenedores para obtener datos y pasarlos a los componentes presentacionales.

```typescript
// src/app/pages/characters/characters-container/characters-container.component.ts
import { Component, OnInit } from '@angular/core';
import { RickAndMortyService } from '../../../services/rick-and-morty.service';

@Component({
  selector: 'app-characters-container',
  template: `<app-characters-list [characters]="characters"></app-characters-list>`
})
export class CharactersContainerComponent implements OnInit {
  characters: any[] = [];

  constructor(private rickAndMortyService: RickAndMortyService) {}

  ngOnInit() {
    this.rickAndMortyService.getCharacters().subscribe(data => {
      this.characters = data.results;
    });
  }
}
```

```typescript
// src/app/pages/locations/locations-container/locations-container.component.ts
import { Component, OnInit } from '@angular/core';
import { RickAndMortyService } from '../../../services/rick-and-morty.service';

@Component({
  selector: 'app-locations-container',
  template: `<app-locations-list [locations]="locations"></app-locations-list>`
})
export class LocationsContainerComponent implements OnInit {
  locations: any[] = [];

  constructor(private rickAndMortyService: RickAndMortyService) {}

  ngOnInit() {
    this.rickAndMortyService.getLocations().subscribe(data => {
      this.locations = data.results;
    });
  }
}
```

#### Paso 7: Implementar los Componentes Presentacionales
Implementa los componentes presentacionales para mostrar los datos.

```typescript
// src/app/pages/characters/characters-list/characters-list.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-characters-list',
  template: `
    <div *ngFor="let character of characters">
      <h3>{{ character.name }}</h3>
      <img [src]="character.image" alt="{{ character.name }}">
      <p>Status: {{ character.status }}</p>
      <p>Species: {{ character.species }}</p>
    </div>
  `
})
export class CharactersListComponent {
  @Input() characters: any[];
}
```

```typescript
// src/app/pages/locations/locations-list/locations-list.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-locations-list',
  template: `
    <div *ngFor="let location of locations">
      <h3>{{ location.name }}</h3>
      <p>Type: {{ location.type }}</p>
      <p>Dimension: {{ location.dimension }}</p>
    </div>
  `
})
export class LocationsListComponent {
  @Input() locations: any[];
}
```

#### Paso 8: Actualizar Módulos de Características
Actualiza los módulos de características para incluir los nuevos componentes y el enrutamiento.

```typescript
// src/app/pages/characters/characters.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CharactersRoutingModule } from './characters-routing.module';
import { CharactersContainerComponent } from './characters-container/characters-container.component';
import { CharactersListComponent } from './characters-list/characters-list.component';

@NgModule({
  declarations: [
    CharactersContainerComponent,
    CharactersListComponent
  ],
  imports: [
    CommonModule,
    CharactersRoutingModule
  ]
})
export class CharactersModule { }
```

```typescript
// src/app/pages/locations/locations.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LocationsRoutingModule } from './locations-routing.module';
import { LocationsContainerComponent } from './locations-container/locations-container.component';
import { LocationsListComponent } from './locations-list/locations-list.component';

@NgModule({
  declarations: [
    LocationsContainerComponent,
    LocationsListComponent
  ],
  imports: [
    CommonModule,
    LocationsRoutingModule
  ]
})
export class LocationsModule { }
```

#### Paso 9: Probar la Aplicación
Ejecuta la aplicación para verificar que todo funcione correctamente.

```sh
ng serve
```

Navega a `http://localhost:4200/characters` para ver la lista de personajes y a `http://localhost:4200/locations` para ver la lista de localizaciones.

### Conclusión
En este laboratorio, has aprendido a crear una aplicación Angular que utiliza módulos, lazy loading y el patrón presentacional y contenedor. La aplicación obtiene datos de la API de Rick and Morty y los muestra en dos páginas diferentes. Este enfoque mejora la organización del código y facilita el mantenimiento y la escalabilidad de la aplicación.