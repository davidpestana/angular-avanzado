El patrón Higher-Order Component (HOC) es un patrón de diseño muy común en React, pero puede ser adaptado a Angular utilizando directivas, componentes, y servicios. Un Higher-Order Component en Angular se puede lograr creando un componente que envuelve a otros componentes, proporcionando funcionalidad adicional sin modificar el componente original. A continuación, te mostraré cómo implementar un patrón HOC en Angular.

## Implementación de HOC en Angular

### 1. Crear un servicio para manejar la lógica común

Primero, crearemos un servicio para encapsular la lógica común que queremos compartir entre componentes.

```typescript
// src/app/services/hoc.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class HocService {
  private message: string = 'Hello from HOC Service';

  getMessage(): string {
    return this.message;
  }
}
```

### 2. Crear una directiva para envolver el componente

Luego, crearemos una directiva que aplicará la lógica del HOC al componente que envuelve.

```typescript
// src/app/directives/hoc.directive.ts
import { Directive, ViewContainerRef, ComponentFactoryResolver, Input, OnInit } from '@angular/core';
import { HocService } from '../services/hoc.service';

@Directive({
  selector: '[appHoc]'
})
export class HocDirective implements OnInit {
  @Input('appHoc') component: any;

  constructor(
    private vcRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver,
    private hocService: HocService
  ) {}

  ngOnInit() {
    const factory = this.resolver.resolveComponentFactory(this.component);
    const componentRef = this.vcRef.createComponent(factory);
    componentRef.instance.message = this.hocService.getMessage();
  }
}
```

### 3. Crear el componente que será envuelto

Creamos un componente que será envuelto por el HOC.

```typescript
// src/app/components/original/original.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-original',
  template: `<h1>{{ message }}</h1>`
})
export class OriginalComponent {
  @Input() message: string;
}
```

### 4. Utilizar la directiva HOC en un componente host

Finalmente, utilizamos la directiva HOC en un componente host para aplicar la funcionalidad adicional al componente original.

```typescript
// src/app/components/host/host.component.ts
import { Component } from '@angular/core';
import { OriginalComponent } from '../original/original.component';

@Component({
  selector: 'app-host',
  template: `<ng-container *appHoc="originalComponent"></ng-container>`
})
export class HostComponent {
  originalComponent = OriginalComponent;
}
```

### 5. Declarar las directivas y componentes en el módulo de la aplicación

Asegúrate de declarar las directivas y componentes en el módulo de la aplicación.

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { HocDirective } from './directives/hoc.directive';
import { OriginalComponent } from './components/original/original.component';
import { HostComponent } from './components/host/host.component';

@NgModule({
  declarations: [
    AppComponent,
    HocDirective,
    OriginalComponent,
    HostComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Explicación

1. **Servicio**: `HocService` proporciona una funcionalidad común que puede ser compartida entre componentes.
2. **Directiva**: `HocDirective` actúa como el HOC, envolviendo el componente original y añadiendo funcionalidad adicional (en este caso, inyectando un mensaje).
3. **Componente Original**: `OriginalComponent` es el componente al que se le aplicará la lógica adicional.
4. **Componente Host**: `HostComponent` utiliza la directiva HOC para envolver el componente original y añadir la funcionalidad del HOC.

### Ejecución

Para ver el patrón HOC en acción, ejecuta tu aplicación Angular y navega al componente host. Verás que el mensaje del servicio HOC se muestra en el componente original.

```sh
ng serve
```

Este enfoque te permite reutilizar la lógica común de manera eficiente sin modificar los componentes individuales, siguiendo el principio de separación de preocupaciones.