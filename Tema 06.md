# 6. Inyección de dependencias y Standalone APIs

## 6.1. Introducción a los Standalone APIs y la inyección moderna en Angular 20

Uno de los cambios más profundos en la evolución de Angular ha sido la simplificación de su modelo de arquitectura. Durante años, los **NgModules** fueron el corazón del framework: todo componente, directiva o pipe debía declararse en un módulo, y los servicios se registraban en sus `providers`. Esto funcionaba, pero añadía una capa de complejidad que muchas veces no aportaba valor.  

Con la llegada de Angular 14 se introdujeron los **Standalone Components**, y a partir de Angular 19 esta idea se consolidó: **todos los artefactos son standalone por defecto**. Esto significa que ya no necesitas marcar explícitamente `standalone: true` en los decoradores. En Angular 20, este modelo es la norma, y los NgModules han pasado a ser opcionales y, en la práctica, prescindibles en la mayoría de los proyectos.

### 6.1.1. ¿Qué significa que todo sea *standalone*?

Un componente, directiva o pipe puede existir y usarse sin necesidad de estar declarado en un módulo. Basta con importarlo allí donde lo necesites.  

Ejemplo de un componente standalone en Angular 20:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>Hola desde Angular 20</h1>`
})
export class HelloComponent {}
```

Este componente puede usarse directamente en una ruta o importarse en otro componente sin necesidad de un NgModule. Esto simplifica la arquitectura y hace que el código sea más directo y fácil de mantener.

### 6.1.2. La inyección de dependencias en Angular 20

La **inyección de dependencias (DI)** sigue siendo uno de los pilares del framework. Angular mantiene un **inyector jerárquico** que se encarga de entregar instancias de servicios allí donde se necesiten.  

Lo que ha cambiado es la forma de trabajar con esa inyección:  
- Los servicios pueden declararse con `providedIn: 'root'` para estar disponibles en toda la aplicación, o limitarse a un componente o ruta concreta.  
- Ya no dependemos de NgModules para registrar servicios.  
- Podemos usar la función `inject()` como alternativa moderna a la inyección por constructor.  

### 6.1.3. `inject()`: la forma moderna de acceder a dependencias

La función `inject()` permite obtener una instancia de un servicio sin necesidad de declararlo en el constructor. Sin embargo, es importante entender que **solo puede usarse dentro de un contexto de inyección**.  

Según la documentación oficial, los contextos válidos son:  
- El constructor de un componente, directiva, pipe o servicio.  
- La inicialización de propiedades en esas clases.  
- Una función de fábrica (`useFactory`) de un provider o un `InjectionToken`.  
- Funciones que Angular ejecuta en un contexto de inyección, como guards de router (`CanActivateFn`, `ResolveFn`).  
- Bloques ejecutados explícitamente con `runInInjectionContext()`.  

#### Ejemplo válido en un componente

```ts
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-profile',
  template: `<p>Usuario: {{ user.name }}</p>`
})
export class ProfileComponent {
  private userService = inject(UserService); // ✅ válido en inicialización de propiedad
  user = this.userService.getUser();
}
```

#### Ejemplo válido en un guard de router

```ts
import { CanActivateFn } from '@angular/router';
import { AuthService } from './auth.service';
import { inject } from '@angular/core';

export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService); // ✅ válido en un guard
  return auth.isLoggedIn();
};
```

#### Ejemplo inválido

```ts
// ❌ Esto lanzará NG0203 porque no hay contexto de inyección
const userService = inject(UserService);

export function helper() {
  return userService.getUser();
}
```

Si necesitas usar `inject()` fuera de un contexto natural, puedes envolver la llamada con `runInInjectionContext()` y pasarle un inyector (por ejemplo, el `EnvironmentInjector`).

### 6.1.4. Inyección en rutas y *scoped providers*

Otra novedad poderosa es que ahora podemos **definir servicios directamente en las rutas**. Esto permite que un servicio exista solo mientras esa ruta esté activa, reduciendo consumo de memoria y mejorando el rendimiento.

```ts
import { Routes } from '@angular/router';
import { DashboardComponent } from './dashboard.component';
import { DashboardService } from './dashboard.service';

export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    providers: [DashboardService] // servicio disponible solo en esta ruta
  }
];
```

### 6.1.5. Beneficios del modelo moderno

1. **Menos complejidad**: ya no necesitamos NgModules para organizar dependencias.  
2. **Mayor flexibilidad**: podemos decidir el alcance de un servicio (global, por componente, por ruta).  
3. **Mejor rendimiento**: los servicios se crean solo cuando son necesarios.  
4. **Código más limpio**: `inject()` evita constructores largos y repetitivos, siempre que se use en un contexto válido.  


