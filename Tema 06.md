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


## 6.2. Providers funcionales vs providers clásicos: diferencias y casos de uso

La **inyección de dependencias** en Angular siempre ha sido muy flexible. Desde las primeras versiones, hemos podido declarar servicios y decidir cómo se crean, qué alcance tienen y cómo se sustituyen por implementaciones alternativas.  

En Angular 20, además de los **providers clásicos**, contamos con los **providers funcionales**, una forma más moderna y declarativa de configurar la inyección. Entender ambos enfoques es clave para equipos que migran proyectos o que quieren aprovechar las ventajas de la nueva sintaxis.


### 6.2.1. Providers clásicos

Los providers clásicos son los que se han usado desde siempre en Angular. Se configuran mediante objetos que indican **qué token se provee** y **cómo se resuelve**.  

Ejemplos típicos:

```ts
import { NgModule } from '@angular/core';
import { LoggerService } from './logger.service';

// Ejemplo clásico en un módulo o componente
@NgModule({
  providers: [
    LoggerService, // shorthand de { provide: LoggerService, useClass: LoggerService }
    { provide: 'API_URL', useValue: 'https://api.midominio.com' },
    { provide: LoggerService, useClass: BetterLoggerService },
    { provide: LoggerService, useExisting: OtherLoggerService },
    { provide: LoggerService, useFactory: () => new LoggerService(true) }
  ]
})
export class AppModule {}
```

Características:  
- Se basan en objetos de configuración (`provide`, `useClass`, `useValue`, `useFactory`, `useExisting`).  
- Son muy expresivos y permiten sustituir implementaciones fácilmente.  
- Han sido la base de Angular desde sus inicios.  

### 6.2.2. Providers funcionales

Con Angular moderno, especialmente desde Angular 15 en adelante, se introdujo una nueva forma de declarar providers: **los providers funcionales**.  

En lugar de objetos, se usan **funciones auxiliares** que simplifican la sintaxis y mejoran la legibilidad.  

Ejemplo:

```ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { importProvidersFrom } from '@angular/core';

export const appConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor])
    ),
    provideRouter(routes),
  ]
};
```

Características:  
- Usan funciones como `provideHttpClient()`, `provideRouter()`, `provideAnimations()`.  
- Son más declarativos y fáciles de leer.  
- Permiten configurar dependencias complejas con menos código repetitivo.  
- Están pensados para el mundo **standalone**, donde ya no dependemos de NgModules.  

### 6.2.3. Diferencias clave

| Aspecto | Providers clásicos | Providers funcionales |
|---------|-------------------|------------------------|
| **Sintaxis** | Basada en objetos `{ provide, useClass, useValue, useFactory }` | Basada en funciones (`provideX()`) |
| **Legibilidad** | Verbosa en configuraciones complejas | Más concisa y declarativa |
| **Compatibilidad** | Disponible desde Angular 2 | Introducidos en Angular 15, consolidados en Angular 20 |
| **Uso típico** | Sustitución de clases, valores estáticos, factories personalizadas | Configuración de servicios del framework (HTTP, Router, Animations, i18n) |
| **Migración** | Sigue siendo válido y soportado | Recomendado para nuevos proyectos standalone |

### 6.2.4. Casos de uso

#### Cuándo usar providers clásicos
- Cuando necesitas **control fino** sobre cómo se crea un servicio.  
- Para **aliasing** (`useExisting`) o **factories personalizadas**.  
- En proyectos legacy que aún usan NgModules.  

Ejemplo:

```ts
{ provide: LoggerService, useFactory: () => new LoggerService(true) }
```

#### Cuándo usar providers funcionales
- En **aplicaciones nuevas standalone**.  
- Para configurar **servicios del framework** (HTTP, Router, Animations, i18n).  
- Cuando quieres **menos boilerplate** y una sintaxis más clara.  

Ejemplo:

```ts
provideHttpClient(withInterceptors([authInterceptor]))
```


## 6.3. Jerarquía avanzada de inyección y modificadores (`@Optional`, `@Self`, `@SkipSelf`)

La **inyección de dependencias (DI)** en Angular se basa en un sistema jerárquico de inyectores. Esto significa que cuando un componente solicita un servicio, Angular no busca únicamente en un único lugar, sino que recorre una **cadena de inyectores** hasta encontrar una coincidencia.  

Comprender esta jerarquía es fundamental para aplicaciones grandes, donde distintos componentes pueden necesitar **instancias distintas** de un mismo servicio o, por el contrario, compartir una única instancia global.

### 6.3.1. La jerarquía de inyectores en Angular

En Angular existen principalmente dos niveles de inyectores:

- **EnvironmentInjector (nivel de aplicación)**: es el inyector raíz, creado al arrancar la aplicación. Los servicios con `providedIn: 'root'` viven aquí y son compartidos en toda la app.  
- **ElementInjector (nivel de componente/directiva)**: cada componente/directiva puede tener su propio inyector local, definido en su propiedad `providers`.  

Cuando Angular necesita resolver una dependencia:  
1. Busca primero en el **inyector local** del componente.  
2. Si no la encuentra, sube en la jerarquía hacia el padre.  
3. Si llega al inyector raíz y tampoco está, lanza un error `NullInjectorError`.  

### 6.3.2. Modificadores de inyección

Angular nos da decoradores especiales para **alterar este comportamiento por defecto**.  

#### 🔹 `@Self()`

Indica que la dependencia debe resolverse **únicamente en el inyector local** del componente/directiva.  
- Si no existe allí, Angular lanza un error.  
- Útil cuando queremos asegurarnos de que un servicio se provea de forma explícita en ese componente.  

```ts
constructor(@Self() private logger: LoggerService) {}
```

👉 Aquí, si `LoggerService` no está en los `providers` del componente, fallará aunque exista en el inyector raíz.

#### 🔹 `@SkipSelf()`

Indica que Angular debe **saltar el inyector local** y empezar la búsqueda en el padre.  
- Útil cuando queremos evitar una redefinición local y usar la instancia de un nivel superior.  

```ts
constructor(@SkipSelf() private logger: LoggerService) {}
```

👉 Aquí, aunque el componente tenga su propio `LoggerService` en `providers`, Angular usará el del padre.

#### 🔹 `@Optional()`

Indica que la dependencia es **opcional**.  
- Si Angular no encuentra el servicio en la jerarquía, en lugar de lanzar un error, inyecta `null`.  
- Muy útil para dependencias que pueden o no estar presentes.  

```ts
constructor(@Optional() private analytics?: AnalyticsService) {}
```

👉 Aquí, si `AnalyticsService` no está registrado en ningún inyector, la propiedad será `undefined` y el componente seguirá funcionando.

### 6.3.3. Ejemplo práctico combinando modificadores

Imaginemos un escenario con un componente padre y un hijo, y un servicio `LoggerService`:

```ts
@Component({
  selector: 'app-parent',
  template: `<app-child></app-child>`,
  providers: [LoggerService] // Proveedor en el padre
})
export class ParentComponent {}

@Component({
  selector: 'app-child',
  template: `<p>Child works!</p>`,
  providers: [LoggerService] // Proveedor en el hijo
})
export class ChildComponent {
  constructor(
    @Self() private localLogger: LoggerService,
    @SkipSelf() private parentLogger: LoggerService,
    @Optional() private maybeAnalytics?: AnalyticsService
  ) {
    console.log('Local logger:', this.localLogger);
    console.log('Parent logger:', this.parentLogger);
    console.log('Analytics:', this.maybeAnalytics);
  }
}
```

- `@Self()` → obtiene el `LoggerService` definido en el hijo.  
- `@SkipSelf()` → ignora el del hijo y obtiene el del padre.  
- `@Optional()` → si `AnalyticsService` no existe en ningún inyector, no rompe la app.  

### 6.3.4. Casos de uso reales

- **`@Self()`**: cuando queremos garantizar que un servicio se provea de forma explícita en un componente (ej. un `FormControlDirective` que necesita su propio `NgControl`).  
- **`@SkipSelf()`**: útil en directivas que se comunican con un servicio del componente padre (ej. `ControlContainer` en formularios reactivos).  
- **`@Optional()`**: ideal para servicios que son opcionales, como un `LoggerService` que solo se inyecta en entornos de desarrollo.  


## 6.4. Uso de `InjectionToken` para inyecciones condicionales y seguras

En Angular, la inyección de dependencias se basa en **tokens**: identificadores que el inyector utiliza para saber qué instancia debe entregar. Cuando inyectamos una clase (por ejemplo, `UserService`), la propia clase actúa como token.  

Pero ¿qué ocurre si queremos inyectar algo que **no es una clase**? Por ejemplo:  
- Una **configuración** (un objeto con parámetros).  
- Un **valor primitivo** (string, number, boolean).  
- Una **interfaz** de TypeScript (que no existe en tiempo de ejecución).  

En estos casos, necesitamos un **`InjectionToken`**: un objeto especial que Angular puede usar como identificador único y seguro.

### 6.4.1. Creación de un `InjectionToken`

Un `InjectionToken` se crea con el constructor `new InjectionToken<T>()`, donde `T` es el tipo de dato que queremos inyectar.

```ts
import { InjectionToken } from '@angular/core';

export interface AppConfig {
  apiUrl: string;
  featureFlag: boolean;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');
```

👉 Aquí hemos creado un token llamado `APP_CONFIG` que representa un objeto de configuración de tipo `AppConfig`.

### 6.4.2. Proveer un valor con un `InjectionToken`

Podemos asociar un valor al token en los `providers` de la aplicación:

```ts
import { APP_CONFIG } from './app.config';

export const appConfig: AppConfig = {
  apiUrl: 'https://api.midominio.com',
  featureFlag: true
};

export const appProviders = [
  { provide: APP_CONFIG, useValue: appConfig }
];
```

### 6.4.3. Inyectar el valor en un componente o servicio

```ts
import { Component, inject } from '@angular/core';
import { APP_CONFIG, AppConfig } from './app.config';

@Component({
  selector: 'app-root',
  template: `<p>API: {{ config.apiUrl }}</p>`
})
export class AppComponent {
  config = inject(APP_CONFIG); // ✅ inyección segura y tipada
}
```

👉 Angular garantiza que el valor inyectado corresponde al tipo `AppConfig`, lo que evita errores en tiempo de compilación.

### 6.4.4. `InjectionToken` con factorías y lógica condicional

Un `InjectionToken` también puede definirse con una **factoría**, lo que permite crear el valor dinámicamente e incluso inyectar otros servicios dentro de esa factoría.

```ts
import { InjectionToken, inject } from '@angular/core';
import { EnvironmentService } from './environment.service';

export const API_URL = new InjectionToken<string>('api.url', {
  providedIn: 'root',
  factory: () => {
    const env = inject(EnvironmentService);
    return env.isProd ? 'https://api.prod.com' : 'https://api.dev.com';
  }
});
```

👉 Aquí el valor del token `API_URL` depende de si estamos en producción o en desarrollo.  
Esto nos da **inyecciones condicionales** sin necesidad de escribir lógica repetida en cada componente.

### 6.4.5. Ejemplo práctico: configuración multi-entorno

Supongamos que tenemos distintos endpoints según el entorno:

```ts
export const FEATURE_FLAG = new InjectionToken<boolean>('feature.flag', {
  providedIn: 'root',
  factory: () => {
    return window.location.hostname.includes('staging');
  }
});
```

En un componente:

```ts
@Component({
  selector: 'app-feature',
  template: `
    @if (featureFlag) {
      <p>Funcionalidad beta activada 🚀</p>
    }
  `
})
export class FeatureComponent {
  featureFlag = inject(FEATURE_FLAG);
}
```

👉 De esta forma, el componente se adapta automáticamente al entorno sin necesidad de condicionales dispersos en el código.

### 6.4.6. Buenas prácticas con `InjectionToken`

- **Usa `InjectionToken` para interfaces y valores primitivos**: nunca intentes inyectar directamente un string o un número, ya que no son únicos.  
- **Centraliza la configuración**: define tokens para parámetros globales (API URLs, flags, claves de terceros).  
- **Aprovecha las factorías**: permiten lógica condicional y valores calculados en tiempo de ejecución.  
- **Evita duplicados**: recuerda que cada `new InjectionToken()` crea un identificador único; si lo defines en varios archivos, Angular los tratará como tokens distintos.  


