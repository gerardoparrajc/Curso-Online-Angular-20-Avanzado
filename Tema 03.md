
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


## 3.4 Métodos clásicos (`markForCheck`, `detach`, `detectChanges`) en modo legacy

A pesar de los avances en Angular 20 con Signals y el nuevo Change Detection, los métodos clásicos siguen siendo relevantes en proyectos legacy o cuando se necesita un control manual sobre la actualización de la vista. Estos métodos forman parte de la API de `ChangeDetectorRef` y permiten gestionar la detección de cambios de forma explícita.

### 3.4.1 `markForCheck()`

Este método se usa principalmente en componentes con estrategia `OnPush`. Cuando Angular no detecta automáticamente un cambio (por ejemplo, tras modificar datos fuera del flujo normal de Angular), puedes llamar a `markForCheck()` para indicar que el componente debe ser revisado en el próximo ciclo de detección de cambios.

**Ejemplo:**
```ts
import { ChangeDetectorRef } from '@angular/core';

constructor(private cdr: ChangeDetectorRef) {}

actualizarDatosExternos() {
	// Modificas datos desde fuera de Angular
	this.valor = obtenerValorDeFuera();
	this.cdr.markForCheck(); // Fuerza la comprobación
}
```

### 3.4.2 `detectChanges()`

Este método fuerza la detección de cambios de manera inmediata en el componente y sus hijos. Es útil cuando necesitas actualizar la vista tras un cambio que Angular no detecta automáticamente.

**Ejemplo:**
```ts
import { ChangeDetectorRef } from '@angular/core';

constructor(private cdr: ChangeDetectorRef) {}

actualizarManual() {
	this.valor = nuevoValor;
	this.cdr.detectChanges(); // Actualiza la vista ahora mismo
}
```

### 3.4.3 `detach()`

Permite "desconectar" el componente del árbol de detección de cambios. Una vez llamado, el componente deja de actualizarse automáticamente hasta que se vuelva a conectar con `reattach()`.

**Ejemplo:**
```ts
import { ChangeDetectorRef } from '@angular/core';

constructor(private cdr: ChangeDetectorRef) {}

pausarActualizaciones() {
	this.cdr.detach(); // El componente deja de actualizarse
}

reanudarActualizaciones() {
	this.cdr.reattach(); // Vuelve a conectarse al árbol
}
```

### ¿Cuándo usar estos métodos?

- En aplicaciones legacy que no usan Signals ni Zoneless Angular.
- Cuando necesitas controlar manualmente la actualización de la vista por motivos de rendimiento o integración con librerías externas.
- En casos donde los cambios ocurren fuera del flujo de Angular (por ejemplo, callbacks de librerías JS, Web Workers, etc.).

### Consideraciones

- El uso excesivo de estos métodos puede complicar el mantenimiento y la depuración.
- En Angular 20, Signals y el nuevo Change Detection suelen hacer innecesario el uso manual de estos métodos en la mayoría de los casos.
- Si migras a Signals, revisa dónde usas estos métodos y evalúa si puedes simplificar la lógica.


## 3.5 Uso de `effect()` y `computed()` como mecanismo moderno de gestión de cambios

Con Angular 20, la gestión de cambios en la interfaz ha evolucionado gracias a los nuevos mecanismos reactivos: `effect()` y `computed()`. Estas herramientas permiten una reactividad más precisa, declarativa y eficiente, eliminando la necesidad de gestionar manualmente la detección de cambios en la mayoría de los casos.

### ¿Qué es `computed()`?

`computed()` permite definir valores derivados a partir de uno o varios Signals. Cada vez que cambian los Signals de los que depende, el valor calculado se actualiza automáticamente y solo se recalcula si realmente hay un cambio relevante.

**Ejemplo:**
```ts
import { signal, computed } from '@angular/core';

const precio = signal(100);
const cantidad = signal(2);

const total = computed(() => precio() * cantidad());
```

En la plantilla:
```html
<input type="number" [value]="precio()" (input)="precio.set($event.target.value)">
<input type="number" [value]="cantidad()" (input)="cantidad.set($event.target.value)">
<p>Total: {{ total() }}</p>
```

El valor de `total()` se actualiza automáticamente cuando cambian `precio` o `cantidad`, sin necesidad de gestionar la detección de cambios manualmente.

### ¿Qué es `effect()`?

`effect()` permite ejecutar acciones externas (side effects) cada vez que cambian los Signals de los que depende. Es ideal para tareas como guardar datos, enviar logs, actualizar el almacenamiento local, etc.

**Ejemplo:**
```ts
import { signal, effect } from '@angular/core';

const contador = signal(0);

effect(() => {
	console.log('El contador cambió:', contador());
});

function incrementar() {
	contador.update(v => v + 1);
}
```

Cada vez que el usuario llama a `incrementar()`, el efecto se ejecuta y muestra el nuevo valor en consola.

### Ventajas frente a métodos clásicos

- No necesitas llamar a `markForCheck`, `detectChanges` ni preocuparte por la estrategia de Change Detection.
- La reactividad es automática y granular: solo se actualiza lo que realmente depende de los datos cambiados.
- El código es más declarativo y fácil de mantener.

### Buenas prácticas

- Usa `computed()` para valores derivados y lógica de negocio.
- Usa `effect()` para acciones externas y sincronización con el mundo exterior.
- Evita mezclar lógica de presentación y efectos en los mismos métodos.
- Documenta los efectos para facilitar la depuración.


## 3.6 Estrategias para minimizar recomputaciones innecesarias

En aplicaciones Angular modernas, especialmente con Signals y `computed()`, es fundamental evitar recomputaciones innecesarias para mantener el rendimiento óptimo. Aunque Angular 20 ya optimiza mucho el proceso, existen buenas prácticas y patrones que ayudan a reducir aún más el trabajo de la detección de cambios.

### 3.6.1 Usa datos inmutables

Siempre que modifiques arrays u objetos, crea una nueva referencia en vez de mutar el dato en sitio. Esto permite a Angular detectar el cambio de forma eficiente y evitar recomputaciones accidentales.

**Ejemplo:**
```ts
// Incorrecto: muta el array en sitio
items().push('nuevo');
// Correcto: crea un nuevo array
items.update(list => [...list, 'nuevo']);
```

### 3.6.2 Divide los `computed()` en piezas pequeñas y reutilizables

En vez de crear un único `computed()` grande que dependa de muchos Signals, divide la lógica en varios `computed()` pequeños y claros. Así, solo se recalcula la parte que realmente cambió.

**Ejemplo:**
```ts
const productos = signal<Product[]>([]);
const filtro = signal<string>('');

const productosFiltrados = computed(() =>
	productos().filter(p => p.nombre.includes(filtro()))
);
const total = computed(() =>
	productosFiltrados().reduce((acc, p) => acc + p.precio, 0)
);
```

### 3.6.3 Evita dependencias innecesarias en `computed()` y `effect()`

Solo lee los Signals que realmente necesitas dentro de cada `computed()` o `effect()`. Si lees un Signal por error, Angular lo considerará una dependencia y recalculará el valor cada vez que cambie.

**Ejemplo:**
```ts
const a = signal(1);
const b = signal(2);
// Solo depende de 'a', no de 'b'
const suma = computed(() => a());
```

### 3.6.4 Usa `untracked()` para lecturas puntuales

Si necesitas leer el valor de un Signal dentro de un `computed()` o `effect()` pero no quieres que se convierta en dependencia reactiva, usa `untracked()`.

**Ejemplo:**
```ts
import { untracked } from '@angular/core';

const usuario = signal({ nombre: 'Ana', rol: 'admin' });
const rolActual = computed(() => untracked(usuario).rol);
```

### 3.6.5 Evita ciclos de dependencias

No hagas que un `computed()` dependa directa o indirectamente de sí mismo. Esto puede provocar bucles infinitos y recomputaciones innecesarias.

### 3.6.6 Efectos perezosos y selectivos

Recuerda que los `computed()` y los efectos (`effect()`) son perezosos: solo se ejecutan si alguien los lee o los usa. Aprovecha esto para crear cálculos que solo se activan cuando realmente se necesitan (por ejemplo, pestañas, paneles, etc.).

### 3.6.7 Agrupa dependencias en objetos o Signals derivados

Si tienes varios parámetros que determinan una petición o cálculo, agrúpalos en un solo `computed()` o Signal. Así, solo se recalcula cuando realmente cambia el conjunto de parámetros.

**Ejemplo:**
```ts
const pagina = signal(1);
const tamaño = signal(20);
const filtros = signal({ categoria: 'A', stock: true });

const parametros = computed(() => ({
	pagina: pagina(),
	tamaño: tamaño(),
	...filtros()
}));
```

### 3.6.8 Monitoriza y perfila tu aplicación

Utiliza las herramientas de desarrollo de Angular y el navegador para identificar recomputaciones excesivas. Si detectas cálculos que se disparan demasiado, revisa las dependencias y la estructura de tus Signals y `computed()`.


