# 4. Directivas y control flow moderno

## 4.1 Introducción al nuevo control flow con `@if`, `@for` y `@switch`

Uno de los cambios más significativos que trae **Angular 20** es la introducción del **nuevo sistema de control de flujo en plantillas**, que reemplaza y moderniza a las clásicas directivas estructurales (`*ngIf`, `*ngFor`, `*ngSwitch`).  

Este nuevo enfoque no solo simplifica la sintaxis, sino que también mejora la **legibilidad**, la **seguridad de tipos** y el **rendimiento**. Ahora, en lugar de usar directivas con asterisco, Angular nos ofrece **bloques de control nativos** con una sintaxis más clara y cercana a otros lenguajes de programación.

### 4.1.1. ¿Por qué un nuevo control de flujo?

Hasta Angular 19, el control de flujo en plantillas se basaba en directivas estructurales con un asterisco (`*`). Aunque funcionaban bien, tenían algunas limitaciones:

- La sintaxis podía resultar confusa para principiantes (`*ngIf="condición; else otraPlantilla"`).  
- La composición de condiciones y bucles anidados era poco legible.  
- No existía un soporte tan fuerte de **TypeScript** en las expresiones de plantilla.  

Con Angular 20, el equipo introduce un **lenguaje de plantillas más expresivo**, que hace que el código sea más fácil de leer, mantener y depurar.

### 4.1.2. El bloque `@if`

El nuevo `@if` sustituye a `*ngIf`.  
Ejemplo clásico con `*ngIf`:

```html
<div *ngIf="isLoggedIn; else loginBlock">
  Bienvenido, usuario
</div>
<ng-template #loginBlock>
  <p>Por favor, inicia sesión</p>
</ng-template>
```

Con el nuevo `@if`:

```html
@if (isLoggedIn) {
  <div>Bienvenido, usuario</div>
} @else {
  <p>Por favor, inicia sesión</p>
}
```

**Ventajas**:
- Sintaxis más clara y natural.  
- El bloque `@else` se integra directamente, sin necesidad de `ng-template`.  
- Mejor soporte de autocompletado y tipos en editores.

### 4.1.3. El bloque `@for`

El nuevo `@for` reemplaza a `*ngFor`.  
Ejemplo clásico:

```html
<ul>
  <li *ngFor="let user of users; trackBy: trackById">
    {{ user.name }}
  </li>
</ul>
```

Con `@for`:

```html
<ul>
  @for (user of users; track user.id) {
    <li>{{ user.name }}</li>
  }
</ul>
```

**Ventajas**:
- La cláusula `track` es más explícita y clara que `trackBy`.  
- La sintaxis se parece más a un bucle tradicional.  
- Permite anidar bucles y condiciones de forma más legible.

### 4.1.4. El bloque `@switch`

El nuevo `@switch` sustituye a `*ngSwitch`.  
Ejemplo clásico:

```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'loading'">Cargando...</p>
  <p *ngSwitchCase="'success'">Datos cargados</p>
  <p *ngSwitchDefault>Error</p>
</div>
```

Con `@switch`:

```html
@switch (status) {
  @case ('loading') {
    <p>Cargando...</p>
  }
  @case ('success') {
    <p>Datos cargados</p>
  }
  @default {
    <p>Error</p>
  }
}
```

**Ventajas**:
- Sintaxis más limpia y estructurada.  
- Los bloques `@case` y `@default` son más fáciles de leer.  
- Se acerca a la sintaxis de un `switch` en TypeScript.

### 4.1.5. Beneficios globales del nuevo control flow

- **Legibilidad**: el código se parece más a estructuras de control de TypeScript.  
- **Menos plantillas auxiliares**: ya no necesitas `ng-template` para `else` o `switch`.  
- **Mejor integración con Signals**: los bloques reaccionan de forma más natural a cambios en datos reactivos.  
- **Mayor consistencia**: la sintaxis es uniforme entre `@if`, `@for` y `@switch`.

