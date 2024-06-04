El patrón Presentational and Container (Presentacional y Contenedor) es un patrón de diseño común en el desarrollo de aplicaciones frontend, especialmente en bibliotecas como React, pero también se puede aplicar en Angular. Este patrón divide los componentes en dos categorías: componentes presentacionales y componentes contenedores.

- **Componentes Presentacionales**: Son responsables de la presentación y la visualización de datos. No contienen lógica de negocio y no interactúan directamente con el estado de la aplicación.
- **Componentes Contenedores**: Son responsables de la lógica de negocio, obtención de datos y manejo del estado. Pasan los datos a los componentes presentacionales a través de propiedades (inputs).

A continuación, te muestro cómo implementar este patrón en Angular.

## Implementación del Patrón Presentacional y Contenedor en Angular

### 1. Crear el Componente Presentacional

El componente presentacional solo se encarga de la presentación de los datos. No tiene lógica de negocio y recibe los datos a través de @Input().

```typescript
// src/app/components/user-profile/user-profile.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  template: `
    <div>
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
      <p>Phone: {{ user.phone }}</p>
    </div>
  `
})
export class UserProfileComponent {
  @Input() user: { name: string, email: string, phone: string };
}
```

### 2. Crear el Componente Contenedor

El componente contenedor maneja la lógica de negocio, obtiene los datos y los pasa al componente presentacional.

```typescript
// src/app/components/user-container/user-container.component.ts
import { Component, OnInit } from '@angular/core';
import { UserService } from '../../services/user.service';

@Component({
  selector: 'app-user-container',
  template: `
    <app-user-profile [user]="user"></app-user-profile>
  `
})
export class UserContainerComponent implements OnInit {
  user: { name: string, email: string, phone: string };

  constructor(private userService: UserService) { }

  ngOnInit() {
    this.userService.getUser().subscribe(data => {
      this.user = data;
    });
  }
}
```

### 3. Crear el Servicio para Obtener Datos

El servicio obtiene los datos desde una API o una fuente de datos.

```typescript
// src/app/services/user.service.ts
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  getUser(): Observable<{ name: string, email: string, phone: string }> {
    const user = { name: 'John Doe', email: 'john.doe@example.com', phone: '123-456-7890' };
    return of(user); // Aquí podrías usar HttpClient para obtener los datos de una API real
  }
}
```

### 4. Declarar los Componentes y Servicios en el Módulo de la Aplicación

Asegúrate de declarar los componentes y servicios en el módulo de la aplicación.

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { UserProfileComponent } from './components/user-profile/user-profile.component';
import { UserContainerComponent } from './components/user-container/user-container.component';
import { UserService } from './services/user.service';

@NgModule({
  declarations: [
    AppComponent,
    UserProfileComponent,
    UserContainerComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [UserService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 5. Usar el Componente Contenedor en la Aplicación

Finalmente, usa el componente contenedor en tu aplicación.

```typescript
// src/app/app.component.html
<app-user-container></app-user-container>
```

### Explicación

1. **Componente Presentacional**: `UserProfileComponent` recibe los datos del usuario a través de @Input() y los muestra.
2. **Componente Contenedor**: `UserContainerComponent` maneja la obtención de datos y pasa los datos al componente presentacional.
3. **Servicio**: `UserService` proporciona los datos del usuario. En un escenario real, este servicio usaría `HttpClient` para obtener datos desde una API.
4. **Módulo de Aplicación**: `AppModule` declara y proporciona los componentes y el servicio.

### Beneficios del Patrón Presentacional y Contenedor

- **Separación de Preocupaciones**: Claramente separa la lógica de negocio y la presentación.
- **Reusabilidad**: Los componentes presentacionales pueden ser fácilmente reutilizados en diferentes contenedores.
- **Mantenimiento**: Facilita el mantenimiento al aislar cambios en la lógica de negocio y la presentación.

Este patrón es útil para crear aplicaciones escalables y mantenibles al separar la lógica de negocio de la presentación.