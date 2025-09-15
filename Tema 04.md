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

## 4.2 Diferencias y ventajas respecto a `*ngIf` y `*ngFor` tradicionales

Con Angular 20, el nuevo **control de flujo con `@if` y `@for`** no es simplemente un cambio estético: representa una evolución importante respecto a las directivas estructurales clásicas `*ngIf` y `*ngFor`.  
Veamos en detalle cuáles son las diferencias y qué ventajas aportan en proyectos reales.

### 4.2.1. Sintaxis más clara y natural

- **Antes**:  
  ```html
  <div *ngIf="isLoggedIn; else loginBlock">
    Bienvenido
  </div>
  <ng-template #loginBlock>
    <p>Por favor, inicia sesión</p>
  </ng-template>
  ```

- **Ahora**:  
  ```html
  @if (isLoggedIn) {
    <div>Bienvenido</div>
  } @else {
    <p>Por favor, inicia sesión</p>
  }
  ```

👉 La nueva sintaxis elimina la necesidad de `ng-template` y se parece más a un `if/else` de TypeScript, lo que mejora la **legibilidad** y reduce el código “ceremonial”.

### 4.2.2. Eliminación del asterisco y plantillas implícitas

- Con `*ngIf` y `*ngFor`, el asterisco (`*`) era un atajo para crear un `ng-template` oculto.  
- Esto generaba confusión, sobre todo para quienes se iniciaban en Angular.  

Con `@if` y `@for`, el bloque es explícito y no hay plantillas ocultas: lo que ves en el HTML es exactamente lo que Angular procesa.

### 4.2.3. Mejor soporte de tipos y tooling

- El nuevo control flow está **integrado en el compilador** de Angular, lo que permite:
  - Autocompletado más preciso en editores.
  - Errores de compilación más claros.
  - Mejor integración con **Signals** y datos reactivos.

Esto significa menos errores en tiempo de ejecución y más ayuda en tiempo de desarrollo.

### 4.2.4. `@for` y el nuevo algoritmo de *diffing*

- `*ngFor` usaba un algoritmo de comparación menos eficiente, que podía provocar renders innecesarios.  
- `@for` introduce un **nuevo algoritmo de diffing** que mejora hasta un **90% el rendimiento en listas grandes**.  
- Además, la cláusula `track` es más clara que `trackBy`:

  ```html
  @for (user of users; track user.id) {
    <li>{{ user.name }}</li>
  }
  ```

👉 Esto hace que el renderizado de listas sea más rápido y predecible.

### 4.2.5. Nuevas capacidades: `@empty`

Con `*ngFor`, si querías mostrar un mensaje cuando la lista estaba vacía, debías combinarlo con un `*ngIf`.  
Ahora, `@for` incluye directamente un bloque `@empty`:

```html
@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <p>No hay elementos disponibles</p>
}
```

Esto simplifica mucho el código y lo hace más expresivo.

### 4.2.6. Ventajas globales

| Aspecto                  | `*ngIf` / `*ngFor` (clásicos) | `@if` / `@for` (Angular 20) |
|---------------------------|-------------------------------|-----------------------------|
| Sintaxis                  | Basada en directivas con `*`  | Bloques nativos, estilo TS  |
| Else / casos alternativos | Requiere `ng-template`        | Bloques `@else`, `@empty`   |
| Legibilidad               | Menor, más verbosa            | Mayor, más natural          |
| Rendimiento en listas     | Algoritmo clásico             | Nuevo diffing, más rápido   |
| Tooling y tipos           | Limitado                      | Mejor soporte en compilador |
| Futuro                    | En proceso de deprecación     | ✅ Nuevo estándar recomendado |

## 4.3 Directivas de atributos reactivas integradas con Signals

Hasta ahora hemos visto cómo los **bloques de control de flujo** (`@if`, `@for`, `@switch`) se integran de forma natural con Signals. Pero Angular 20 no se queda ahí: también podemos aprovechar la reactividad de Signals en las **directivas de atributos**, es decir, aquellas que modifican el comportamiento o el estilo de un elemento existente sin alterar su estructura.

Esto abre la puerta a un patrón muy poderoso: **directivas reactivas**, que responden automáticamente a cambios en Signals sin necesidad de suscripciones manuales ni `async pipe`.

### 4.3.1. ¿Qué son las directivas de atributos reactivas?

Una **directiva de atributo** es aquella que se aplica sobre un elemento HTML para modificar su apariencia o comportamiento. Ejemplos clásicos son `ngClass`, `ngStyle` o `ngModel`.

Con Signals, podemos crear directivas que:

- Escuchen cambios en un Signal.
- Actualicen automáticamente el DOM.
- Mantengan el código más declarativo y limpio.

### 4.3.2. Ejemplo básico: directiva `highlightOn`

Imaginemos que queremos resaltar un elemento cuando un Signal booleano esté activo.

```ts
import { Directive, ElementRef, effect, input } from '@angular/core';

@Directive({
  selector: '[highlightOn]',
  standalone: true
})
export class HighlightOnDirective {
  // Recibimos un Signal como input
  highlightOn = input.required<boolean>();

  constructor(private el: ElementRef) {
    // Creamos un efecto reactivo
    effect(() => {
      if (this.highlightOn()) {
        this.el.nativeElement.style.backgroundColor = 'yellow';
      } else {
        this.el.nativeElement.style.backgroundColor = 'transparent';
      }
    });
  }
}
```

Uso en plantilla:

```html
<div [highlightOn]="isHighlighted">
  Este texto se resalta automáticamente
</div>
```

Donde `isHighlighted` es un **Signal** en el componente:

```ts
isHighlighted = signal(false);
```

### 4.3.3. Ventajas frente a directivas tradicionales

- **Sin suscripciones manuales**: no necesitamos `subscribe()` ni `ngOnDestroy`.  
- **Reactividad declarativa**: el `effect()` se encarga de escuchar el Signal y actualizar el DOM.  
- **Menos boilerplate**: el código es más corto y expresivo.  
- **Mejor integración con el compilador**: Angular sabe qué Signals afectan a la directiva y optimiza el *Change Detection*.

### 4.3.4. Ejemplo avanzado: directiva `debounceInput`

Podemos crear una directiva que convierta el valor de un `<input>` en un Signal reactivo con *debounce* integrado.

```ts
import { Directive, ElementRef, output, effect, signal } from '@angular/core';

@Directive({
  selector: 'input[debounceInput]',
  standalone: true
})
export class DebounceInputDirective {
  private el = inject(ElementRef<HTMLInputElement>);
  private valueSignal = signal('');

  // Exponemos un output reactivo
  value = output<string>();

  constructor() {
    // Escuchamos cambios en el input
    this.el.nativeElement.addEventListener('input', (e: Event) => {
      const target = e.target as HTMLInputElement;
      this.valueSignal.set(target.value);
    });

    // Creamos un efecto con debounce manual
    let timeout: any;
    effect(() => {
      const current = this.valueSignal();
      clearTimeout(timeout);
      timeout = setTimeout(() => this.value.emit(current), 300);
    });
  }
}
```

Uso en plantilla:

```html
<input debounceInput (value)="onSearch($event)" />
```

Ahora cada vez que el usuario escribe, el evento `value` se emite con un retardo de 300ms, sin necesidad de RxJS ni `FormControl`.

### 4.3.5. Buenas prácticas

- Usa `effect()` dentro de la directiva para reaccionar a Signals.  
- Prefiere `input()` y `output()` en lugar de `@Input()` y `@Output()` cuando trabajes con Signals.  
- Mantén las directivas **pequeñas y específicas**: una directiva = un comportamiento.  
- Evita lógica compleja en la directiva; delega en servicios si es necesario.  

## 4.4 Host Directives: reutilización y encapsulación de lógica transversal

En aplicaciones grandes, es común que distintos componentes necesiten **comportamientos repetidos**: validaciones, estilos dinámicos, accesibilidad, manejo de eventos globales, etc.

Hasta ahora, la solución típica era crear **directivas de atributos** y aplicarlas manualmente en cada plantilla. Sin embargo, esto podía generar **duplicación de código** y dificultar la **encapsulación**.

Con Angular 20, gracias a la **Directive Composition API**, disponemos de las **Host Directives**: una forma de **inyectar directivas directamente en un componente** para reutilizar lógica transversal sin necesidad de aplicarlas explícitamente en la plantilla.

### 4.4.1. ¿Qué son las Host Directives?

Las **Host Directives** permiten que un componente “herede” el comportamiento de una o varias directivas, aplicándolas automáticamente a su elemento host.  

En otras palabras:  
- Son directivas que se **componen dentro de un componente**.  
- Se aplican de forma **estática en tiempo de compilación**.  
- Sus **host bindings, listeners e inputs/outputs** se integran en el componente.  

Esto significa que puedes encapsular lógica común en directivas y luego **reutilizarla en múltiples componentes** sin repetir código ni ensuciar las plantillas.

### 4.4.2. Ejemplo básico

Supongamos que tenemos una directiva que añade un tooltip:

```ts
import { Directive, HostListener, input } from '@angular/core';

@Directive({
  selector: '[tooltip]',
  standalone: true
})
export class TooltipDirective {
  text = input<string>('');

  @HostListener('mouseenter', ['$event.target'])
  showTooltip(el: HTMLElement) {
    // Lógica para mostrar tooltip
    console.log('Mostrar tooltip:', this.text());
  }

  @HostListener('mouseleave')
  hideTooltip() {
    console.log('Ocultar tooltip');
  }
}
```

Ahora queremos que un componente `UserCard` siempre tenga este comportamiento, sin necesidad de escribir `[tooltip]` en la plantilla.

```ts
import { Component } from '@angular/core';
import { TooltipDirective } from './tooltip.directive';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `<div class="card">Contenido de la tarjeta</div>`,
  hostDirectives: [
    {
      directive: TooltipDirective,
      inputs: ['text: tooltipText'] // Exponemos el input con un alias
    }
  ]
})
export class UserCardComponent {}
```

Uso en plantilla:

```html
<app-user-card tooltipText="Información del usuario"></app-user-card>
```

👉 El componente `UserCard` **hereda automáticamente** la lógica de `TooltipDirective`.  
No necesitamos aplicarla manualmente en la plantilla.

### 4.4.3. Ventajas de las Host Directives

- **Reutilización real**: encapsulas lógica transversal (tooltips, accesibilidad, validaciones, estilos dinámicos) en directivas y las aplicas en múltiples componentes.  
- **Plantillas más limpias**: no necesitas añadir atributos extra en cada uso.  
- **Encapsulación**: el componente expone solo los inputs/outputs que decidas.  
- **Composición flexible**: puedes aplicar varias directivas a un mismo componente.  
- **Consistencia**: todos los componentes que usan la misma host directive comparten el mismo comportamiento.

### 4.4.4. Ejemplo avanzado: accesibilidad

Imagina que quieres que todos tus botones personalizados tengan soporte de accesibilidad (`aria-label`, `role`, etc.).  

```ts
@Directive({
  selector: '[a11y]',
  standalone: true
})
export class AccessibilityDirective {
  label = input<string>('');
  role = input<string>('button');

  constructor(private el: ElementRef) {
    effect(() => {
      this.el.nativeElement.setAttribute('role', this.role());
      this.el.nativeElement.setAttribute('aria-label', this.label());
    });
  }
}
```

Ahora, cualquier componente de botón puede heredar esta directiva:

```ts
@Component({
  selector: 'app-primary-button',
  standalone: true,
  template: `<button class="btn-primary"><ng-content /></button>`,
  hostDirectives: [
    {
      directive: AccessibilityDirective,
      inputs: ['label: ariaLabel', 'role: ariaRole']
    }
  ]
})
export class PrimaryButtonComponent {}
```

Uso:

```html
<app-primary-button ariaLabel="Guardar cambios"></app-primary-button>
```

## 5. Buenas prácticas

- **Usa Host Directives para lógica transversal**: accesibilidad, estilos comunes, tooltips, validaciones.  
- **No abuses**: si la directiva solo se usa en un lugar, probablemente no necesite ser host directive.  
- **Expón solo lo necesario**: controla qué inputs/outputs se heredan para no sobrecargar la API del componente.  
- **Combínalas con Signals**: los `input()` y `effect()` hacen que la lógica sea aún más reactiva y declarativa.  
