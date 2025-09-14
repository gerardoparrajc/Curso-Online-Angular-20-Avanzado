
# 3. Change Detection avanzado en Angular 20

## 3.1 Conceptos fundamentales del Change Detection en Angular

El sistema de **Change Detection** (detección de cambios) es uno de los pilares de Angular. Es el mecanismo que permite que la interfaz de usuario se mantenga sincronizada con el estado de la aplicación: cada vez que los datos cambian, Angular decide qué partes de la vista deben actualizarse y cuándo hacerlo.

### ¿Qué es Change Detection?

Change Detection es el proceso por el cual Angular detecta cambios en los datos y actualiza la vista en consecuencia. Cuando modificas una variable, recibes datos de una API, o el usuario interactúa con la aplicación, Angular se encarga de reflejar esos cambios en la interfaz.

### ¿Cómo funciona internamente?

Angular organiza la aplicación en una **árbol de componentes**. Cada componente tiene su propio contexto de detección de cambios. Cuando ocurre un evento (por ejemplo, un click, una petición HTTP, un timer, etc.), Angular inicia un ciclo de detección de cambios que recorre el árbol y verifica si los datos usados en las plantillas han cambiado.

En versiones anteriores, Angular usaba principalmente **Zone.js** para interceptar eventos asíncronos y disparar la detección de cambios automáticamente. Con Angular 20 y la llegada de Signals, el sistema se ha optimizado para ser más preciso y eficiente, actualizando solo lo necesario.

### Tipos de Change Detection

Angular ofrece dos estrategias principales:

- **Default (por defecto):**
	- Angular revisa todos los componentes desde la raíz cada vez que ocurre un evento relevante.
	- Es seguro y sencillo, pero puede ser menos eficiente en aplicaciones grandes.

- **OnPush:**
	- Angular solo revisa el componente cuando cambian sus inputs, se dispara un evento en el propio componente, o cambia un Signal usado en la plantilla.
	- Es mucho más eficiente, especialmente cuando trabajas con datos inmutables y Signals.

### ¿Qué dispara la detección de cambios?

Algunos ejemplos de acciones que disparan Change Detection:
- Eventos del usuario (click, input, submit, etc.).
- Respuestas de peticiones HTTP.
- Timers (`setTimeout`, `setInterval`).
- Cambios en Signals o Observables usados en la plantilla.

### Ejemplo básico

```ts
import { Component } from '@angular/core';

@Component({
	selector: 'app-ejemplo',
	template: `
		<button (click)="incrementar()">Incrementar</button>
		<p>Valor: {{ valor }}</p>
	`
})
export class EjemploComponent {
	valor = 0;
	incrementar() {
		this.valor++;
		// Angular detecta el cambio y actualiza la vista
	}
}
```

Cada vez que el usuario pulsa el botón, Angular detecta el cambio en `valor` y actualiza el `<p>` correspondiente.

### Change Detection y Signals

Con Signals, Angular sabe exactamente qué partes de la interfaz dependen de cada dato. Cuando un Signal cambia, solo se actualizan los componentes y plantillas que realmente lo usan, evitando recálculos innecesarios y mejorando el rendimiento.

**Ejemplo con Signal:**

```ts
import { signal } from '@angular/core';

valor = signal(0);

function incrementar() {
	valor.update(v => v + 1);
}
```

En la plantilla:
```html
<button (click)="incrementar()">Incrementar</button>
<p>Valor: {{ valor() }}</p>
```

Ahora, solo el `<p>` que depende de `valor()` se actualiza cuando el Signal cambia.


## 3.2 Zoneless Angular: funcionamiento interno sin Zone.js

Uno de los cambios más importantes en Angular 20 es la posibilidad de trabajar **sin Zone.js**. Este enfoque, conocido como "Zoneless Angular", representa una evolución significativa en la forma en que Angular gestiona la detección de cambios y la reactividad interna.

### ¿Qué es Zone.js y por qué se usaba?

Zone.js es una librería que intercepta todas las operaciones asíncronas (eventos, timers, promesas, peticiones HTTP, etc.) en JavaScript. Angular la utilizaba para saber cuándo debía ejecutar la detección de cambios, sin que el desarrollador tuviera que preocuparse por ello.

**Ventaja:** Automatizaba la reactividad y simplificaba el desarrollo.

**Desventaja:** Añadía complejidad, sobrecarga y podía dificultar la depuración o el rendimiento en aplicaciones grandes.

### ¿Qué significa Zoneless Angular?

En Angular 20, puedes desactivar Zone.js y aprovechar el nuevo sistema de Signals y Change Detection reactivo. Esto implica que Angular ya no depende de interceptar todos los eventos asíncronos, sino que actualiza la vista solo cuando cambian los datos que realmente afectan a la interfaz.

### ¿Cómo funciona la detección de cambios sin Zone.js?

- **Signals:** Cuando usas Signals, Angular sabe exactamente qué partes de la interfaz dependen de cada dato. Al cambiar un Signal, solo se actualizan los componentes y plantillas que lo usan.
- **Eventos explícitos:** La detección de cambios se dispara por eventos del usuario (click, input, etc.), cambios en Signals, o llamadas explícitas a métodos como `markDirty()`.
- **Sin sobrecarga global:** No se interceptan todos los eventos asíncronos, lo que reduce el trabajo innecesario y mejora el rendimiento.

### Ventajas de Zoneless Angular

- **Rendimiento mejorado:** Menos recálculos y menos trabajo innecesario.
- **Depuración más sencilla:** El flujo de actualización es más predecible y fácil de seguir.
- **Menos dependencias:** Puedes eliminar Zone.js de tu proyecto si no lo necesitas.
- **Reactividad declarativa:** Signals y el nuevo Change Detection permiten una reactividad más clara y controlada.

### Ejemplo práctico

Supón que tienes un componente que actualiza su estado mediante Signals:

```ts
import { signal } from '@angular/core';

contador = signal(0);

function incrementar() {
	contador.update(v => v + 1);
}
```

En la plantilla:
```html
<button (click)="incrementar()">Sumar</button>
<p>Contador: {{ contador() }}</p>
```

Sin Zone.js, Angular actualiza solo el `<p>` cuando el Signal `contador` cambia, sin recorrer todo el árbol de componentes ni depender de eventos globales.

### ¿Cuándo conviene usar Zoneless Angular?

- Cuando tu aplicación usa principalmente Signals y datos inmutables.
- Si buscas el máximo rendimiento y control sobre la reactividad.
- En proyectos donde la depuración y la predictibilidad son clave.

### Consideraciones y migración

- Si tu app depende de librerías que requieren Zone.js, revisa la compatibilidad antes de desactivarlo.
- Puedes migrar gradualmente, usando Signals y OnPush, y desactivando Zone.js cuando estés listo.



## 3.3 Estrategias `OnPush` en coexistencia con Signals

La estrategia `OnPush` ha sido durante años la clave para optimizar el rendimiento en Angular, ya que limita la detección de cambios a situaciones concretas: cuando cambian los inputs del componente, se dispara un evento en el propio componente, o se marca manualmente como sucio (`markDirty()`).

Con la llegada de Signals en Angular 20, `OnPush` y Signals pueden coexistir y potenciarse mutuamente, permitiendo aplicaciones aún más eficientes y reactivas.

### ¿Cómo funciona `OnPush`?

Cuando un componente usa `ChangeDetectionStrategy.OnPush`, Angular solo actualiza la vista si:
- Cambia un input del componente.
- Se dispara un evento en el componente (click, input, etc.).
- Se llama manualmente a métodos como `markDirty()`.

Esto reduce el número de comprobaciones y mejora el rendimiento, especialmente en aplicaciones grandes.

### Signals y `OnPush`: ¿cómo interactúan?

Los Signals introducen una reactividad granular: Angular sabe exactamente qué partes de la plantilla dependen de cada Signal. Cuando un Signal cambia, solo se actualizan los componentes y vistas que realmente lo usan, incluso si el componente está en modo `OnPush`.

**Ventaja:**
- No necesitas preocuparte por marcar el componente como sucio: el cambio en el Signal lo hace automáticamente.
- Puedes combinar datos inmutables (inputs) y Signals para obtener lo mejor de ambos mundos.

### Ejemplo práctico

```ts
import { Component, signal, ChangeDetectionStrategy } from '@angular/core';

@Component({
	selector: 'app-contador',
	changeDetection: ChangeDetectionStrategy.OnPush,
	template: `
		<button (click)="incrementar()">Sumar</button>
		<p>Contador: {{ contador() }}</p>
	`
})
export class ContadorComponent {
	contador = signal(0);
	incrementar() {
		this.contador.update(v => v + 1);
	}
}
```

En este ejemplo, el componente está en modo `OnPush`, pero el cambio en el Signal `contador` actualiza la vista automáticamente, sin necesidad de marcar el componente como sucio.

### Buenas prácticas en coexistencia

- Usa `OnPush` en todos los componentes salvo casos muy específicos.
- Prefiere Signals para el estado local y derivaciones reactivas.
- Mantén los inputs inmutables y usa Signals para datos que cambian frecuentemente.
- Evita mutar objetos/arrays en sitio: crea nuevas referencias para que Angular detecte los cambios.

