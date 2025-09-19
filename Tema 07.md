# 7. Routing avanzado en Angular 20

## 7.1. Configuración moderna de rutas con `provideRouter` y Standalone Components

El enrutamiento en Angular siempre ha sido el mecanismo que permite a los usuarios navegar entre diferentes vistas sin recargar la página. Tradicionalmente, esto se hacía importando `RouterModule.forRoot()` dentro de un NgModule. Sin embargo, con la llegada de los **Standalone Components** y la API de **Functional Providers**, Angular 20 simplifica y moderniza este proceso.  

### 7.1.1. El cambio de paradigma: de NgModules a `provideRouter`

Antes:  
```ts
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

Ahora, en Angular 20:  
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app.component';
import { routes } from './app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes)
  ]
});
```

👉 Ya no necesitamos un `AppRoutingModule`. La configuración es más directa y declarativa.

### 7.1.2. Definición de rutas con Standalone Components

Cada ruta apunta directamente a un componente standalone.  

Ejemplo en `app.routes.ts`:

```ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { NotFoundComponent } from './not-found/not-found.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'user/:id', component: UserProfileComponent },
  { path: '**', component: NotFoundComponent }
];
```

Características:  
- **Rutas estáticas**: `path: ''` → muestra `HomeComponent`.  
- **Rutas dinámicas**: `path: 'user/:id'` → muestra `UserProfileComponent` con un parámetro.  
- **Ruta comodín**: `path: '**'` → captura cualquier URL no definida y muestra `NotFoundComponent`.  

### 7.1.3. Lazy Loading con `loadComponent`

Una de las grandes ventajas del modelo standalone es que podemos cargar componentes bajo demanda, sin necesidad de módulos.  

```ts
export const routes: Routes = [
  {
    path: 'settings',
    loadComponent: () =>
      import('./settings/settings.component').then(m => m.SettingsComponent)
  }
];
```

👉 Esto reduce el tamaño inicial del bundle y mejora el rendimiento en aplicaciones grandes.

### 7.1.4. Integración con Functional Providers

El enrutador puede configurarse con **opciones adicionales** usando funciones auxiliares:

```ts
import { provideRouter, withHashLocation } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withHashLocation())
  ]
});
```

- `withHashLocation()` → usa `#/ruta` en lugar de `ruta` (útil en entornos donde el servidor no soporta rutas limpias).  
- También existen `withDebugTracing()`, `withPreloading()`, etc., para ajustar el comportamiento del router.  

### 7.1.5. Beneficios del modelo moderno

- **Menos boilerplate**: no más `AppRoutingModule`.  
- **Standalone-first**: cada componente puede ser directamente una ruta.  
- **Lazy loading simplificado**: con `loadComponent` no hacen falta módulos intermedios.  
- **Configuración declarativa**: `provideRouter` y sus helpers (`withX`) hacen que el código sea más legible y mantenible.  


## 7.2. Lazy loading optimizado con `loadChildren` y rutas dinámicas

El **lazy loading** (carga diferida) es una técnica fundamental en Angular para mejorar el rendimiento: en lugar de cargar toda la aplicación al inicio, solo se cargan los módulos o componentes cuando el usuario realmente los necesita. Esto reduce el *bundle* inicial, acelera el arranque y optimiza el uso de recursos.  

En Angular 20, el lazy loading se integra de forma natural con el modelo **standalone** y se potencia con `loadChildren`, que permite cargar módulos o componentes bajo demanda, incluso de forma condicional o dinámica.

### 7.2.1. Lazy loading clásico con `loadChildren`

En versiones anteriores, `loadChildren` se usaba para cargar módulos enteros. En Angular 20, sigue siendo válido, pero ahora puede apuntar tanto a **módulos** como a **componentes standalone**.

Ejemplo con un módulo:

```ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  }
];
```

En `admin.routes.ts`:

```ts
import { Routes } from '@angular/router';
import { AdminDashboardComponent } from './admin-dashboard.component';

export const ADMIN_ROUTES: Routes = [
  { path: '', component: AdminDashboardComponent }
];
```

👉 Aquí, el módulo de administración solo se carga cuando el usuario navega a `/admin`.

### 7.2.2. Lazy loading de componentes standalone

Con Angular moderno, ya no es necesario crear módulos intermedios. Podemos cargar directamente un componente standalone:

```ts
export const routes: Routes = [
  {
    path: 'settings',
    loadComponent: () =>
      import('./settings/settings.component').then(m => m.SettingsComponent)
  }
];
```

👉 Esto simplifica la arquitectura y reduce el *boilerplate*.

### 7.2.3. Rutas dinámicas con `loadChildren`

En aplicaciones enterprise, a menudo necesitamos que las rutas dependan de **condiciones dinámicas**:  
- El rol del usuario.  
- Un *feature flag*.  
- El entorno (dev, staging, prod).  

Podemos usar `loadChildren` con factorías que devuelvan rutas distintas según la lógica.

Ejemplo: rutas basadas en rol de usuario

```ts
export const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => {
      const role = localStorage.getItem('role');
      if (role === 'admin') {
        return import('./admin/admin.routes').then(m => m.ADMIN_ROUTES);
      }
      return import('./user/user.routes').then(m => m.USER_ROUTES);
    }
  }
];
```

👉 Aquí, Angular carga dinámicamente el conjunto de rutas adecuado según el rol del usuario.

### 7.2.4. Optimización con `canLoad` y `canMatch`

- **`canLoad`**: evita que un módulo se cargue si el usuario no cumple ciertas condiciones (ej. no está autenticado).  
- **`canMatch`**: decide si una ruta debe coincidir o no antes de cargarla, lo que permite mayor flexibilidad.  

Ejemplo con `canLoad`:

```ts
{
  path: 'reports',
  loadChildren: () => import('./reports/reports.routes').then(m => m.REPORTS_ROUTES),
  canLoad: [AuthGuard]
}
```

👉 Esto asegura que el módulo de reportes no se cargue si el usuario no está autenticado.

### 7.2.5. Estrategias de pre-carga combinadas

El lazy loading puede complementarse con **estrategias de pre-carga** (`PreloadAllModules` o estrategias personalizadas) para cargar en segundo plano los módulos que probablemente se necesiten pronto, sin afectar el arranque inicial.  

Ejemplo:

```ts
import { provideRouter, withPreloading, PreloadAllModules } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
});
```

👉 Esto mejora la experiencia de usuario: la primera navegación a un módulo lazy-loaded será instantánea.


## 7.3. Estrategias de precarga inteligente y bajo demanda con Signals

La **precarga de rutas** es una técnica que permite a Angular cargar en segundo plano módulos o componentes que el usuario todavía no ha visitado, anticipándose a sus acciones. Esto mejora la experiencia de usuario, ya que cuando finalmente navega a esa ruta, el contenido ya está disponible en memoria.  

En Angular clásico, la precarga se gestionaba con estrategias como `PreloadAllModules` o estrategias personalizadas basadas en RxJS. En Angular 20, gracias a la introducción de **Signals**, podemos implementar **estrategias de precarga más inteligentes, reactivas y contextuales**, que se adaptan dinámicamente al comportamiento del usuario y a las condiciones de la aplicación.

### 7.3.1. Precarga tradicional vs precarga con Signals

- **Tradicional**:  
  - Se definía una estrategia global (`PreloadAllModules` o una clase que implementaba `PreloadingStrategy`).  
  - La lógica dependía de observables y suscripciones manuales.  
  - Era difícil adaptar la precarga a condiciones cambiantes en tiempo real.  

- **Con Signals**:  
  - Podemos usar **valores reactivos** que cambian automáticamente según el estado de la aplicación.  
  - La precarga se activa o desactiva en función de señales como: rol del usuario, conectividad de red, interacción previa, o incluso *feature flags*.  
  - La lógica es más declarativa y predecible, sin necesidad de gestionar suscripciones manuales.  

### 7.3.2. Ejemplo básico: precarga condicional con Signals

Supongamos que queremos precargar el módulo de **reportes** solo si el usuario tiene rol de administrador.

```ts
import { signal, effect } from '@angular/core';
import { Router, Route } from '@angular/router';

const isAdmin = signal(false); // valor reactivo que cambia según login

export function smartPreload(router: Router, routes: Route[]) {
  effect(() => {
    if (isAdmin()) {
      // precargar rutas de admin
      const adminRoute = routes.find(r => r.path === 'admin');
      if (adminRoute?.loadChildren) {
        router.preloader.preloadRoute(adminRoute);
      }
    }
  });
}
```

👉 Aquí, cuando `isAdmin` cambia a `true`, Angular automáticamente dispara la precarga del módulo de administración. No necesitamos suscripciones manuales ni lógica imperativa.

### 7.3.3. Precarga bajo demanda (on-demand)

Otra estrategia útil es la **precarga bajo demanda**, donde el sistema no carga rutas hasta que una señal específica lo indique.  

Ejemplo: precargar el módulo de **checkout** solo cuando el usuario añade un producto al carrito.

```ts
const hasItemsInCart = signal(false);

export function preloadCheckout(router: Router, routes: Route[]) {
  effect(() => {
    if (hasItemsInCart()) {
      const checkoutRoute = routes.find(r => r.path === 'checkout');
      if (checkoutRoute?.loadChildren) {
        router.preloader.preloadRoute(checkoutRoute);
      }
    }
  });
}
```

👉 Esto asegura que el flujo de compra sea inmediato cuando el usuario decida pagar, sin retrasos por carga de módulos.

### 7.3.4. Precarga predictiva con Signals

Podemos ir más allá y usar Signals para implementar **precarga predictiva**:  
- Basada en la navegación previa del usuario.  
- Basada en métricas de uso (ej. si el 80% de los usuarios que visitan `/products` luego van a `/checkout`).  
- Basada en condiciones externas (ej. buena conexión de red → precargar más rutas; conexión lenta → precargar menos).  

Ejemplo simplificado:

```ts
const lastVisited = signal<string | null>(null);

export function predictivePreload(router: Router, routes: Route[]) {
  effect(() => {
    if (lastVisited() === 'products') {
      const checkoutRoute = routes.find(r => r.path === 'checkout');
      if (checkoutRoute?.loadChildren) {
        router.preloader.preloadRoute(checkoutRoute);
      }
    }
  });
}
```

👉 Aquí, si el usuario visita `/products`, automáticamente se precarga `/checkout`, anticipando la siguiente acción.

### 7.3.5. Integración con `provideRouter` y estrategias personalizadas

Podemos integrar estas estrategias directamente en la configuración del router:

```ts
import { provideRouter, withPreloading } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withPreloading({
      preload: (route, load) => {
        // lógica basada en signals
        if (route.data?.['preload'] && isAdmin()) {
          return load();
        }
        return false;
      }
    }))
  ]
});
```

👉 Aquí combinamos la API moderna de `withPreloading` con Signals para decidir dinámicamente qué rutas precargar.

### 7.3.6. Beneficios de usar Signals en precarga

- **Reactividad declarativa**: no necesitamos suscripciones manuales ni operadores RxJS complejos.  
- **Adaptabilidad**: la precarga responde automáticamente a cambios en el estado de la aplicación.  
- **Optimización de recursos**: precargamos solo lo necesario, en el momento adecuado.  
- **Mejor experiencia de usuario**: las rutas críticas están listas antes de que el usuario las necesite.  


## 7.4. Precarga condicional en entornos de red lenta o móvil

La **precarga de rutas** es una técnica muy útil para mejorar la experiencia de usuario, pero no siempre conviene aplicarla de manera indiscriminada. En entornos con **redes lentas (2G/3G, conexiones inestables)** o en dispositivos móviles con **modo ahorro de datos**, precargar módulos puede ser contraproducente:  
- Aumenta el consumo de datos.  
- Puede saturar la conexión y ralentizar la navegación inicial.  
- Genera frustración en usuarios con planes de datos limitados.  

Por ello, Angular permite definir **estrategias de precarga personalizadas**, y en Angular 20 podemos enriquecerlas con **Signals** y APIs modernas del navegador para detectar el estado de la red.

### 7.4.1. APIs del navegador para detectar condiciones de red

Los navegadores modernos exponen la API **Network Information** (`navigator.connection`), que nos da información como:  
- `effectiveType`: tipo de red (`4g`, `3g`, `2g`, `slow-2g`).  
- `saveData`: si el usuario activó el modo ahorro de datos.  
- `downlink`: ancho de banda estimado en Mbps.  

Ejemplo:

```ts
const connection = (navigator as any).connection;
console.log(connection.effectiveType); // "4g", "3g", etc.
console.log(connection.saveData); // true o false
```

👉 Con esta información podemos decidir si precargar o no rutas.

### 7.4.2. Estrategia de precarga personalizada

Podemos implementar una clase que extienda `PreloadingStrategy` y condicione la precarga según el estado de la red.

```ts
import { Injectable } from '@angular/core';
import { Route, PreloadingStrategy } from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class NetworkAwarePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    const connection = (navigator as any).connection;
    const saveData = connection?.saveData;
    const slowConnection = connection?.effectiveType?.includes('2g') || connection?.effectiveType === '3g';

    if (saveData || slowConnection) {
      // No precargar en redes lentas o con ahorro de datos
      return of(null);
    }

    // Precargar solo si la ruta tiene data.preload = true
    return route.data?.['preload'] ? load() : of(null);
  }
}
```

En la configuración del router:

```ts
import { provideRouter, withPreloading } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withPreloading(NetworkAwarePreloadingStrategy))
  ]
});
```

### 7.4.3. Precarga condicional con Signals

Podemos combinar la API de red con **Signals** para que la estrategia sea reactiva.  

```ts
import { signal, effect } from '@angular/core';

const isSlowNetwork = signal(false);

function detectNetwork() {
  const connection = (navigator as any).connection;
  if (connection) {
    isSlowNetwork.set(connection.saveData || connection.effectiveType.includes('2g') || connection.effectiveType === '3g');
  }
}

detectNetwork();
```

Ahora podemos usar `isSlowNetwork()` dentro de la estrategia de precarga o incluso en componentes para decidir si precargar rutas bajo demanda.

### 7.4.4. Ejemplo práctico: precarga selectiva en móvil

```ts
export const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.routes').then(m => m.DASHBOARD_ROUTES),
    data: { preload: true }
  },
  {
    path: 'analytics',
    loadChildren: () => import('./analytics/analytics.routes').then(m => m.ANALYTICS_ROUTES),
    data: { preload: false }
  }
];
```

- En **red rápida**: se precarga `dashboard` automáticamente.  
- En **red lenta o móvil con ahorro de datos**: no se precarga nada, y los módulos se cargan solo cuando el usuario navega a ellos.  

### 7.4.5. Beneficios de la precarga condicional

- **Optimización de recursos**: no saturamos la red en condiciones adversas.  
- **Mejor experiencia de usuario**: la aplicación responde mejor en móviles y redes lentas.  
- **Flexibilidad**: podemos decidir qué rutas precargar según contexto (rol, dispositivo, red).  
- **Escalabilidad**: fácil de extender con Signals, feature flags o métricas de uso.  


## 7.5. Functional Guards (`canActivate`, `canMatch`) con `inject()`

Los **guards** en Angular son funciones que actúan como **puntos de control** en el enrutador: determinan si un usuario puede acceder a una ruta, si debe redirigirse a otra, o si una ruta debe siquiera considerarse como candidata durante el proceso de *matching*.  

En versiones anteriores, los guards se implementaban como **clases** que implementaban interfaces (`CanActivate`, `CanLoad`, etc.). En Angular moderno, estos **class-based guards están deprecados** en favor de los **Functional Guards**, que son simplemente funciones puras que aprovechan `inject()` para acceder a servicios.  

### 7.5.1. ¿Qué son los Functional Guards?

- Son **funciones** en lugar de clases.  
- Se definen como constantes tipadas (`CanActivateFn`, `CanMatchFn`, etc.).  
- Usan `inject()` para acceder a servicios como `AuthService`, `Router` o cualquier dependencia registrada en el inyector.  
- Son más concisos, fáciles de testear y se integran mejor con el modelo **standalone**.  

### 7.5.2. Ejemplo de `canActivate` funcional

Un caso típico: proteger rutas que requieren autenticación.

```ts
import { CanActivateFn, Router } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isLoggedIn()) {
    return true;
  }
  return router.createUrlTree(['/login']);
};
```

👉 Aquí:  
- Si el usuario está autenticado, devuelve `true` y permite la navegación.  
- Si no lo está, devuelve un `UrlTree` para redirigir al login (en lugar de `false`, que bloquearía sin redirigir).  

Uso en rutas:

```ts
export const routes: Routes = [
  { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] }
];
```

### 7.5.3. Ejemplo de `canMatch` funcional

`canMatch` decide si una ruta puede **coincidir** durante el proceso de matching. A diferencia de `canActivate`, si devuelve `false`, Angular simplemente prueba con otras rutas en lugar de bloquear la navegación.  

Ejemplo: habilitar una ruta solo si un *feature flag* está activo.

```ts
import { CanMatchFn } from '@angular/router';
import { inject } from '@angular/core';
import { FeatureFlagsService } from './feature-flags.service';

export const analyticsGuard: CanMatchFn = (route, segments) => {
  const flags = inject(FeatureFlagsService);
  return flags.isEnabled('analytics');
};
```

Uso en rutas:

```ts
export const routes: Routes = [
  {
    path: 'analytics',
    loadComponent: () => import('./analytics/analytics.component').then(m => m.AnalyticsComponent),
    canMatch: [analyticsGuard]
  }
];
```

👉 Si el flag `analytics` está desactivado, Angular ignora esta ruta y sigue evaluando otras coincidencias (por ejemplo, una ruta comodín `**`).  

### 7.5.4. Diferencias clave entre `canActivate` y `canMatch`

| Guard        | Momento de ejecución | Comportamiento si devuelve `false` | Uso típico |
|--------------|----------------------|------------------------------------|------------|
| **canActivate** | Antes de activar una ruta ya seleccionada | Bloquea la navegación o redirige | Autenticación, autorización |
| **canMatch**   | Durante el proceso de matching de rutas | Ignora la ruta y prueba con otras | Feature flags, A/B testing, rutas condicionales |

### 7.5.5. Ventajas de los Functional Guards con `inject()`

- **Menos código**: no necesitamos clases ni decoradores.  
- **Mayor claridad**: la lógica está contenida en una función pura.  
- **Testabilidad**: se pueden probar como funciones normales.  
- **Integración con Signals**: podemos usar Signals dentro de guards para decisiones reactivas (ej. precargar rutas solo si una señal indica que el usuario está online).  
- **Standalone-friendly**: encajan perfectamente en aplicaciones sin NgModules.  

## 7.6. Functional Resolvers para la carga previa de datos

En muchas aplicaciones, los componentes necesitan datos antes de renderizarse. Si el componente se muestra primero y luego hace la petición, el usuario ve pantallas vacías o *spinners* que afectan la experiencia. Los **Resolvers** solucionan este problema: permiten que Angular **espere a que los datos estén listos antes de activar la ruta**.  

En Angular 20, los resolvers han evolucionado hacia un modelo **funcional**, más simple y alineado con el enfoque **standalone** y el uso de `inject()`.

### 7.6.1. ¿Qué es un Functional Resolver?

- Es una **función** que implementa el tipo `ResolveFn<T>`.  
- Se ejecuta **antes de activar una ruta**.  
- Puede usar `inject()` para acceder a servicios y obtener datos.  
- Devuelve un valor, un `Promise` o un `Observable`.  
- El resultado se inyecta en el componente a través de `ActivatedRoute.data`.  

👉 En lugar de clases que implementan `Resolve<T>`, ahora basta con una función pura.

### 7.6.2. Ejemplo básico de Functional Resolver

Supongamos que queremos precargar los datos de un usuario antes de mostrar su perfil:

```ts
import { ResolveFn } from '@angular/router';
import { inject } from '@angular/core';
import { UserService } from './user.service';

export const userResolver: ResolveFn<any> = (route, state) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;
  return userService.getUserById(id); // puede devolver Observable o Promise
};
```

Configuración en rutas:

```ts
export const routes: Routes = [
  {
    path: 'user/:id',
    component: UserProfileComponent,
    resolve: { user: userResolver }
  }
];
```

En el componente:

```ts
constructor(private route: ActivatedRoute) {
  this.user = this.route.snapshot.data['user'];
}
```

👉 El componente recibe el usuario ya cargado, sin necesidad de mostrar un *spinner* inicial.

### 7.6.3. Ejemplo avanzado: múltiples resolvers

Podemos usar varios resolvers en la misma ruta:

```ts
export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    resolve: {
      stats: statsResolver,
      notifications: notificationsResolver
    }
  }
];
```

👉 El router esperará a que **todos los resolvers** terminen antes de activar la ruta.

### 7.6.4. Manejo de errores en Functional Resolvers

Si un resolver falla (ej. error en la API), Angular puede:  
- Cancelar la navegación.  
- Redirigir a otra ruta.  
- Devolver un valor alternativo.  

Ejemplo con redirección:

```ts
export const safeUserResolver: ResolveFn<any> = (route, state) => {
  const userService = inject(UserService);
  const router = inject(Router);

  return userService.getUserById(route.paramMap.get('id')!).pipe(
    catchError(() => {
      router.navigate(['/users']);
      return of(null);
    })
  );
};
```

### 7.6.5. Beneficios de los Functional Resolvers

- **Menos boilerplate**: no necesitamos clases ni decoradores.  
- **Mayor claridad**: la lógica de carga está contenida en funciones puras.  
- **Mejor experiencia de usuario**: los datos están listos antes de renderizar.  
- **Integración con Signals**: podemos combinar resolvers con Signals para decisiones reactivas (ej. cargar datos solo si un flag está activo).  
- **Standalone-friendly**: encajan perfectamente en aplicaciones sin NgModules.  


## 7.7. Creación y uso de Functional Interceptors en el flujo HTTP

Los **interceptors** en Angular son una especie de *middleware* que se ejecuta en cada petición y respuesta HTTP. Permiten aplicar lógica transversal sin tener que repetir código en cada servicio o componente. Ejemplos típicos:  
- Añadir cabeceras de autenticación.  
- Manejar errores de forma centralizada.  
- Implementar *retry* con backoff exponencial.  
- Medir tiempos de respuesta.  
- Mostrar u ocultar un *loading spinner*.  

En Angular 20, los **Functional Interceptors** sustituyen al modelo clásico basado en clases (`HttpInterceptor`). Son más simples, predecibles y encajan perfectamente con el enfoque **standalone**.

### 7.7.1. ¿Qué es un Functional Interceptor?

- Es una **función pura** que implementa el tipo `HttpInterceptorFn`.  
- Recibe dos parámetros:  
  - `req`: la solicitud saliente (`HttpRequest`).  
  - `next`: una función que representa el siguiente paso en la cadena de interceptores.  
- Devuelve un `Observable<HttpEvent<any>>`, que puede ser transformado antes de llegar al consumidor.  

👉 A diferencia de los interceptores clásicos, no necesitamos crear clases ni usar `@Injectable()`.

### 7.7.2. Ejemplo básico: interceptor de logging

```ts
import { HttpInterceptorFn } from '@angular/common/http';

export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  console.log('Petición saliente a:', req.url);
  return next(req);
};
```

👉 Este interceptor simplemente registra en consola cada petición antes de enviarla.

### 7.7.3. Ejemplo práctico: añadir token de autenticación

```ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req);
};
```

👉 Aquí usamos `inject()` para acceder a `AuthService` y añadir el token a todas las solicitudes.

### 7.7.4. Manejo centralizado de errores

```ts
import { HttpInterceptorFn } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError(error => {
      console.error('Error HTTP:', error);
      return throwError(() => error);
    })
  );
};
```

👉 Este interceptor captura errores de red o de servidor y los centraliza en un único punto.

### 7.7.5. Registro de interceptores en Angular 20

Los interceptores se configuran al **proveer HttpClient** con `withInterceptors`:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { AppComponent } from './app.component';
import { loggingInterceptor } from './logging.interceptor';
import { authInterceptor } from './auth.interceptor';
import { errorInterceptor } from './error.interceptor';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([loggingInterceptor, authInterceptor, errorInterceptor])
    )
  ]
});
```

👉 El orden importa: se ejecutan en el orden en que se declaran.  

### 7.7.6. Beneficios de los Functional Interceptors

- **Menos boilerplate**: no necesitamos clases ni decoradores.  
- **Mayor claridad**: la lógica está contenida en funciones puras.  
- **Standalone-friendly**: encajan perfectamente en aplicaciones sin NgModules.  
- **Predecibles**: el orden de ejecución es más fácil de razonar.  
- **Testables**: se pueden probar como funciones normales, sin necesidad de Angular TestBed.  


## 7.8. Rutas hijas y segmentación modular en aplicaciones grandes

En aplicaciones enterprise, el enrutamiento no se limita a unas pocas rutas principales. A medida que la aplicación crece, necesitamos **organizar las rutas en jerarquías** y **segmentar la aplicación en áreas funcionales**. Angular 20 ofrece un modelo muy flexible para lograrlo, combinando **rutas hijas**, **layouts compartidos**, **lazy loading** y **Standalone Components**.  

### 7.8.1. ¿Qué son las rutas hijas?

Las **rutas hijas** (o *nested routes*) permiten definir rutas dentro de otras rutas. Esto es útil cuando una sección de la aplicación tiene su propia navegación interna.  

Ejemplo básico:

```ts
export const routes: Routes = [
  {
    path: 'users',
    component: UsersComponent,
    children: [
      { path: 'profile/:id', component: UserProfileComponent },
      { path: 'settings', component: UserSettingsComponent }
    ]
  }
];
```

👉 Aquí, `UsersComponent` actúa como **contenedor** y dentro de él se renderizan las rutas hijas (`profile` y `settings`) en un `<router-outlet>` secundario.

### 7.8.2. El papel de `<router-outlet>`

Para que las rutas hijas funcionen, el **componente padre** debe incluir un `<router-outlet>` en su plantilla. Este elemento es el **punto de anclaje** donde Angular renderiza dinámicamente el componente hijo correspondiente a la ruta activa.  

Plantilla de `UsersComponent`:

```html
<h2>Gestión de usuarios</h2>

<nav>
  <a routerLink="profile/1">Perfil Usuario 1</a>
  <a routerLink="settings">Configuración</a>
</nav>

<!-- Aquí se renderizan los hijos -->
<router-outlet></router-outlet>
```

👉 Cuando el usuario navega a `/users/profile/1`, Angular carga `UserProfileComponent` dentro del `<router-outlet>` de `UsersComponent`.

### 7.8.3. Segmentación modular en aplicaciones grandes

En proyectos enterprise, es común dividir la aplicación en **módulos funcionales** o **áreas de negocio**:  
- `admin` → gestión de usuarios, roles, permisos.  
- `shop` → catálogo, carrito, checkout.  
- `reports` → paneles de estadísticas.  

Cada área puede tener su propio conjunto de rutas hijas y cargarse de forma diferida (*lazy loading*).  

Ejemplo con `loadChildren`:

```ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  },
  {
    path: 'shop',
    loadChildren: () =>
      import('./shop/shop.routes').then(m => m.SHOP_ROUTES)
  }
];
```

En `admin.routes.ts`:

```ts
export const ADMIN_ROUTES: Routes = [
  {
    path: '',
    component: AdminLayoutComponent,
    children: [
      { path: 'users', component: UsersComponent },
      { path: 'roles', component: RolesComponent }
    ]
  }
];
```

👉 Así, cada módulo tiene su propia jerarquía de rutas y un layout independiente.

### 7.8.4. Layouts compartidos con rutas hijas

Un patrón muy común es tener un **layout padre** (con header, sidebar, footer) y dentro de él un `router-outlet` para las vistas hijas:

```ts
export const routes: Routes = [
  {
    path: '',
    component: MainLayoutComponent,
    children: [
      { path: 'dashboard', component: DashboardComponent },
      { path: 'reports', component: ReportsComponent },
      { path: 'settings', component: SettingsComponent }
    ]
  }
];
```

Plantilla de `MainLayoutComponent`:

```html
<header>Mi aplicación</header>
<aside>Menú lateral</aside>

<main>
  <!-- Aquí se cargan las rutas hijas -->
  <router-outlet></router-outlet>
</main>

<footer>© 2025</footer>
```

👉 Todas las rutas hijas (`/dashboard`, `/reports`, `/settings`) comparten el mismo layout.

### 7.8.5. Múltiples `router-outlet` (rutas auxiliares)

Angular permite tener más de un `router-outlet` en la misma vista, lo que habilita **rutas auxiliares** o vistas paralelas.

```html
<router-outlet></router-outlet>
<router-outlet name="sidebar"></router-outlet>
```

En la configuración de rutas:

```ts
export const routes: Routes = [
  { path: 'chat', component: ChatComponent, outlet: 'sidebar' }
];
```

👉 Esto permite cargar `ChatComponent` en el outlet lateral mientras otra ruta principal está activa.

### 7.8.6. Buenas prácticas

- **Usar rutas hijas** para secciones con navegación interna (ej. panel de administración).  
- **Dividir la aplicación en módulos funcionales** y cargarlos con `loadChildren`.  
- **Incluir siempre `<router-outlet>` en el componente padre** de rutas hijas.  
- **Usar layouts compartidos** para mantener consistencia visual en secciones grandes.  
- **Evitar jerarquías demasiado profundas**: mejor agrupar rutas en niveles lógicos.  
- **Documentar la jerarquía de rutas y outlets** en proyectos enterprise para evitar confusión.  


## 7.9. Comparativa entre módulos y standalone en el lazy loading

El **lazy loading** (carga diferida) es una técnica fundamental en Angular para mejorar el rendimiento: permite cargar partes de la aplicación solo cuando el usuario las necesita, reduciendo el *bundle* inicial y acelerando el arranque.  

En Angular, este patrón ha evolucionado:  
- En el modelo clásico, se implementaba a través de **NgModules**.  
- En el modelo moderno (Angular 15+ y consolidado en Angular 20), se puede aplicar directamente sobre **Standalone Components** y configuraciones funcionales.  

### 7.9.1. Lazy loading clásico con NgModules

En versiones anteriores, el lazy loading se implementaba cargando **módulos de características** completos mediante la propiedad `loadChildren`.  

Ejemplo:

```ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.module').then(m => m.AdminModule)
  }
];
```

Características:  
- Se cargaba un **módulo entero** (con sus componentes, directivas, pipes y servicios).  
- Requería mantener un `AdminModule` y un `AdminRoutingModule`.  
- Era más verboso y generaba *boilerplate*.  
- Encajaba bien en arquitecturas modulares, pero añadía complejidad innecesaria en proyectos pequeños o medianos.  

### 7.9.2. Lazy loading moderno con Standalone Components

Con Angular 20, ya no es necesario crear módulos intermedios. Podemos cargar directamente un **Standalone Component** con `loadComponent`, o incluso un conjunto de rutas con `loadChildren`.  

Ejemplo con `loadComponent`:

```ts
export const routes: Routes = [
  {
    path: 'settings',
    loadComponent: () =>
      import('./settings/settings.component').then(m => m.SettingsComponent)
  }
];
```

Ejemplo con `loadChildren` apuntando a rutas standalone:

```ts
export const routes: Routes = [
  {
    path: 'shop',
    loadChildren: () =>
      import('./shop/shop.routes').then(m => m.SHOP_ROUTES)
  }
];
```

Características:  
- Se cargan **componentes standalone** directamente, sin necesidad de módulos.  
- Menos *boilerplate* y más claridad.  
- Mejor integración con **Functional Providers** (`provideRouter`, `provideHttpClient`, etc.).  
- Permite segmentar la aplicación en áreas funcionales sin depender de NgModules.  

### 7.9.3. Diferencias clave

| Aspecto | Lazy loading con NgModules | Lazy loading con Standalone |
|---------|-----------------------------|-----------------------------|
| **Unidad de carga** | Módulo completo (`FeatureModule`) | Componente standalone o conjunto de rutas |
| **Sintaxis** | `loadChildren: () => import('...').then(m => m.FeatureModule)` | `loadComponent` o `loadChildren` con rutas standalone |
| **Boilerplate** | Requiere `Module` + `RoutingModule` | Solo el componente o archivo de rutas |
| **Flexibilidad** | Buena para arquitecturas modulares clásicas | Ideal para apps modernas, más granular |
| **Compatibilidad** | Sigue siendo válido en Angular 20 | Recomendado para nuevos proyectos |

### 7.9.4. Casos de uso

- **NgModules (clásico)**  
  - Migraciones de proyectos legacy.  
  - Librerías de terceros que aún exponen NgModules.  
  - Equipos acostumbrados a la organización modular tradicional.  

- **Standalone (moderno)**  
  - Nuevos proyectos en Angular 20.  
  - Aplicaciones que buscan simplicidad y menor tiempo de arranque.  
  - Escenarios donde se quiere cargar **solo un componente** o un conjunto mínimo de rutas.  


## 7.10. Herramientas de análisis y debugging de rendimiento en el enrutamiento

El enrutamiento es uno de los puntos críticos en aplicaciones Angular enterprise:  
- Cada navegación implica **resolución de rutas, guards, resolvers, precarga y renderizado de componentes**.  
- Un mal diseño puede generar **tiempos de carga elevados**, **navegaciones lentas** o **cargas innecesarias de módulos**.  

Por ello, Angular y el ecosistema moderno ofrecen varias herramientas para **analizar, depurar y optimizar el rendimiento del enrutamiento**.

### 7.10.1. Angular DevTools

[Angular DevTools](https://angular.dev/tools/devtools) es la extensión oficial para Chrome y Firefox que añade un panel específico dentro de las DevTools del navegador.  

Características clave:  
- **Inspector de componentes y rutas**: permite visualizar la jerarquía de componentes y cómo se relacionan con las rutas activas.  
- **Profiler de rendimiento**: mide el tiempo que tarda Angular en ejecutar detección de cambios y navegación entre rutas.  
- **Debugging de router**: muestra qué rutas se activan, qué guards y resolvers se ejecutan y cuánto tardan.  

👉 Es ideal para detectar cuellos de botella en la navegación y confirmar si la precarga o lazy loading están funcionando como se espera.

### 7.10.2. Tracing y logging del Router

Angular Router expone opciones de **tracing** que permiten registrar en consola cada paso del proceso de enrutamiento.  

Ejemplo de activación:

```ts
import { provideRouter, withDebugTracing } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withDebugTracing())
  ]
});
```

Esto genera logs detallados en la consola del navegador:  
- Coincidencia de rutas.  
- Ejecución de guards y resolvers.  
- Carga de módulos lazy.  
- Eventos de navegación (`NavigationStart`, `NavigationEnd`, `NavigationError`).  

👉 Muy útil en entornos de desarrollo para entender por qué una ruta no se activa o por qué tarda demasiado.

### 7.10.3. Medición de tiempos de navegación

Podemos suscribirnos a los eventos del Router para medir tiempos de navegación:

```ts
import { Router, Event, NavigationStart, NavigationEnd } from '@angular/router';

constructor(private router: Router) {
  let start: number;

  this.router.events.subscribe((event: Event) => {
    if (event instanceof NavigationStart) {
      start = performance.now();
    }
    if (event instanceof NavigationEnd) {
      const duration = performance.now() - start;
      console.log(`Navegación completada en ${duration} ms`);
    }
  });
}
```

👉 Esto permite instrumentar métricas personalizadas y enviarlas a herramientas de monitoreo (ej. Grafana, Datadog).

### 7.10.4. Estrategias de profiling con el navegador

Además de Angular DevTools, podemos usar las **DevTools nativas del navegador**:  
- **Performance tab**: grabar una navegación y analizar cuánto tiempo se dedica a scripting, rendering y painting.  
- **Coverage tab**: identificar código no utilizado en módulos precargados.  
- **Network tab**: verificar qué bundles se cargan en cada navegación y su tamaño.  

👉 Esto ayuda a confirmar si el lazy loading realmente está reduciendo el *bundle* inicial.

### 7.10.5. Integración con Signals para debugging

En Angular 20, podemos usar **Signals** para exponer el estado del enrutador y depurarlo de forma reactiva.  

Ejemplo: exponer la ruta activa como signal y loguearla en cada cambio:

```ts
import { signal, effect } from '@angular/core';
import { Router, NavigationEnd } from '@angular/router';

const currentRoute = signal<string>('');

constructor(router: Router) {
  router.events.subscribe(event => {
    if (event instanceof NavigationEnd) {
      currentRoute.set(event.urlAfterRedirects);
    }
  });

  effect(() => {
    console.log('Ruta activa:', currentRoute());
  });
}
```

👉 Esto permite depurar de forma declarativa cómo cambia la navegación en tiempo real.

### 7.10.6. Buenas prácticas de debugging y análisis

- **Activar tracing solo en desarrollo**, nunca en producción.  
- **Combinar Angular DevTools con métricas personalizadas** para tener una visión completa.  
- **Medir tiempos de guards y resolvers**: si tardan demasiado, considerar precarga o caching.  
- **Revisar bundles en Network tab**: confirmar que los módulos lazy no se cargan antes de tiempo.  
- **Automatizar métricas de navegación** en entornos enterprise para detectar degradaciones de rendimiento.  
