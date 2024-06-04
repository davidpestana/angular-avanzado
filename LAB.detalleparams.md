## Laboratorio: Extensión - Páginas de Detalle para Personajes y Localizaciones

### Paso 6: Implementación de las Páginas de Detalle

1. **Detalle de Personaje** (`src/app/features/characters/detail/detail.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ActivatedRoute } from '@angular/router';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-character-detail',
      templateUrl: './detail.component.html',
      styleUrls: ['./detail.component.css']
    })
    export class CharacterDetailComponent implements OnInit {
      character: any;

      constructor(
        private route: ActivatedRoute,
        private rickAndMortyService: RickAndMortyService
      ) {}

      ngOnInit(): void {
        const id = +this.route.snapshot.paramMap.get('id');
        this.rickAndMortyService.getCharacter(id).subscribe(data => {
          this.character = data;
        });
      }
    }
    ```

    ```html
    <!-- src/app/features/characters/detail/detail.component.html -->
    <div class="character-detail" *ngIf="character">
      <h2>{{ character.name }}</h2>
      <img [src]="character.image" alt="{{ character.name }}">
      <p>Status: {{ character.status }}</p>
      <p>Species: {{ character.species }}</p>
      <p>Gender: {{ character.gender }}</p>
      <p>Origin: <a [routerLink]="['/locations', character.origin.url.split('/').pop()]">{{ character.origin.name }}</a></p>
      <p>Location: <a [routerLink]="['/locations', character.location.url.split('/').pop()]">{{ character.location.name }}</a></p>
    </div>
    ```

2. **Detalle de Localización** (`src/app/features/locations/detail/detail.component.ts`):
    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ActivatedRoute } from '@angular/router';
    import { RickAndMortyService } from 'src/app/shared/rick-and-morty.service';

    @Component({
      selector: 'app-location-detail',
      templateUrl: './detail.component.html',
      styleUrls: ['./detail.component.css']
    })
    export class LocationDetailComponent implements OnInit {
      location: any;

      constructor(
        private route: ActivatedRoute,
        private rickAndMortyService: RickAndMortyService
      ) {}

      ngOnInit(): void {
        const id = +this.route.snapshot.paramMap.get('id');
        this.rickAndMortyService.getLocation(id).subscribe(data => {
          this.location = data;
        });
      }
    }
    ```

    ```html
    <!-- src/app/features/locations/detail/detail.component.html -->
    <div class="location-detail" *ngIf="location">
      <h2>{{ location.name }}</h2>
      <p>Type: {{ location.type }}</p>
      <p>Dimension: {{ location.dimension }}</p>
      <h3>Residents</h3>
      <ul>
        <li *ngFor="let resident of location.residents">
          <a [routerLink]="['/characters', resident.split('/').pop()]">{{ resident.split('/').pop() }}</a>
        </li>
      </ul>
    </div>
    ```

### Paso 7: Configuración del Módulo y Rutas para las Páginas de Detalle

1. **Configuración de rutas en el módulo de personajes** (`src/app/features/characters/characters-routing.module.ts`):
    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';
    import { CharactersContainerComponent } from './container/container.component';
    import { CharacterDetailComponent } from './detail/detail.component';

    const routes: Routes = [
      { path: '', component: CharactersContainerComponent },
      { path: ':id', component: CharacterDetailComponent }
    ];

    @NgModule({
      imports: [RouterModule.forChild(routes)],
      exports: [RouterModule]
    })
    export class CharactersRoutingModule { }
    ```

2. **Configuración de rutas en el módulo de localizaciones** (`src/app/features/locations/locations-routing.module.ts`):
    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';
    import { LocationsContainerComponent } from './container/container.component';
    import { LocationDetailComponent } from './detail/detail.component';

    const routes: Routes = [
      { path: '', component: LocationsContainerComponent },
      { path: ':id', component: LocationDetailComponent }
    ];

    @NgModule({
      imports: [RouterModule.forChild(routes)],
      exports: [RouterModule]
    })
    export class LocationsRoutingModule { }
    ```

3. **Agregar rutas importadas en los módulos**:
    ```typescript
    // src/app/features/characters/characters.module.ts
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { CharactersRoutingModule } from './characters-routing.module';
    import { CharactersContainerComponent } from './container/container.component';
    import { CharactersPresenterComponent } from './presenter/presenter.component';
    import { CharacterDetailComponent } from './detail/detail.component';

    @NgModule({
      declarations: [
        CharactersContainerComponent,
        CharactersPresenterComponent,
        CharacterDetailComponent
      ],
      imports: [
        CommonModule,
        CharactersRoutingModule
      ]
    })
    export class CharactersModule { }
    ```

    ```typescript
    // src/app/features/locations/locations.module.ts
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { LocationsRoutingModule } from './locations-routing.module';
    import { LocationsContainerComponent } from './container/container.component';
    import { LocationsPresenterComponent } from './presenter/presenter.component';
    import { LocationDetailComponent } from './detail/detail.component';

    @NgModule({
      declarations: [
        LocationsContainerComponent,
        LocationsPresenterComponent,
        LocationDetailComponent
      ],
      imports: [
        CommonModule,
        LocationsRoutingModule
      ]
    })
    export class LocationsModule { }
    ```

### Paso 8: Actualización de los Presentadores para Incluir Enlaces a las Páginas de Detalle

1. **Actualización del presentador de personajes** (`src/app/features/characters/presenter/presenter.component.html`):
    ```html
    <div class="character-grid" *ngIf="characters.results">
      <div class="character-card" *ngFor="let character of characters.results">
        <h3><a [routerLink]="['/characters', character.id]">{{ character.name }}</a></h3>
        <img [src]="character.image" alt="{{ character.name }}">
      </div>
    </div>
    ```

2. **Actualización del presentador de localizaciones** (`src/app/features/locations/presenter/presenter.component.html`):
    ```html
    <div class="location-list" *ngIf="locations.results">
      <div class="location-card" *ngFor="let location of locations.results">
        <h3><a [routerLink]="['/locations', location.id]">{{ location.name }}</a></h3>
        <p>{{ location.type }} - {{ location.dimension }}</p>
      </div>
    </div>
    ```

### Paso 9: Estilización de las Páginas de Detalle

1. **Estilos adicionales** (`src/styles.css`):
    ```css
    .character-detail, .location-detail {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      padding: 20px;
      max-width: 600px;
      margin: 20px auto;
      text-align: center;
    }

    .character-detail img {
      border-radius: 50%;
      width: 150px;
      height: 150px;
      object-fit: cover;
      margin-bottom: 20px;
    }

    .location-detail ul {
      list-style-type: none;
      padding: 0;
    }

    .location-detail ul li {
      margin: 10px 0;
    }
    ```

### Paso 10: Ejecución del Proyecto

Inicia el servidor de desarrollo:
```sh
ng serve
```

Abre tu navegador y navega a `http://localhost:4200/characters` para ver la lista de personajes y `http://localhost:4200/locations` para ver la lista de localizaciones. Puedes hacer clic en un personaje o una localización para ver sus detalles.