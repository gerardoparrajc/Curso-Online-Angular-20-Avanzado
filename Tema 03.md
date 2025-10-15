
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


## 3.7 Casos prácticos en los que deshabilitar la propagación del árbol es útil

En Angular 20, la propagación del árbol de Change Detection puede deshabilitarse en componentes concretos para optimizar el rendimiento y evitar actualizaciones innecesarias en partes de la interfaz que no dependen de cambios frecuentes. Esta técnica es especialmente útil en escenarios donde la UI es muy grande, contiene zonas estáticas o se integra con librerías externas.

### ¿Qué significa deshabilitar la propagación del árbol?

Implica que un componente y sus hijos dejan de recibir notificaciones automáticas de cambio desde sus ancestros. Solo se actualizan si:
- Cambia un Signal usado en su plantilla.
- Se dispara un evento local.
- Se marca manualmente como sucio (`markDirty()`, `detectChanges()`).

### Casos prácticos

#### 1. Zonas de la UI que no cambian nunca

Por ejemplo, un pie de página, un banner o un menú estático. Si sabes que esos componentes no dependen de datos reactivos, puedes deshabilitar la propagación para evitar que Angular los revise en cada ciclo.

**Ejemplo:**
```ts
@Component({
	selector: 'app-footer',
	changeDetection: ChangeDetectionStrategy.OnPush,
	template: `<footer>© 2025 Mi Empresa</footer>`
})
export class FooterComponent {}
```

#### 2. Integración con librerías externas que gestionan su propio DOM

Si usas un gráfico, mapa o widget que actualiza el DOM por sí mismo (por ejemplo, D3.js, Leaflet, etc.), deshabilitar la propagación evita que Angular intente actualizar esa zona y mejora la compatibilidad y el rendimiento.

#### 3. Listas muy grandes con virtualización

En componentes que muestran miles de elementos (por ejemplo, tablas virtualizadas), puedes deshabilitar la propagación en los elementos hijos para que solo se actualicen los visibles o los que realmente cambian.

#### 4. Paneles, pestañas o modales ocultos

Si tienes paneles o pestañas que no están visibles, puedes deshabilitar la propagación mientras están ocultos y reactivarla solo cuando se muestran, evitando cálculos innecesarios.

#### 5. Animaciones o transiciones gestionadas fuera de Angular

Si una parte de la UI se actualiza por animaciones CSS o JavaScript externo, deshabilitar la propagación evita conflictos y mejora la fluidez.

### ¿Cómo se deshabilita y reactiva la propagación?

En Angular clásico, se usaba `detach()` y `reattach()` del `ChangeDetectorRef`. En Angular 20, puedes usar la nueva API reactiva o estrategias de Change Detection específicas para cada componente.

**Ejemplo:**
```ts
constructor(private cdr: ChangeDetectorRef) {}

ngOnInit() {
	this.cdr.detach(); // Deshabilita la propagación
}

mostrarPanel() {
	this.cdr.reattach(); // Reactiva la propagación
	this.cdr.detectChanges(); // Actualiza la vista
}
```

### Recomendaciones

- Usa esta técnica solo en zonas que realmente no necesitan actualizarse frecuentemente.
- Documenta bien los componentes donde la propagación está deshabilitada para evitar confusiones.
- Si usas Signals, aprovecha su granularidad para limitar aún más las actualizaciones.



## 3.8 Identificación de operaciones costosas y optimización del renderizado

En aplicaciones Angular avanzadas, el rendimiento puede verse afectado por operaciones costosas en el ciclo de Change Detection y el renderizado de la interfaz. Identificar estos puntos críticos y aplicar técnicas de optimización es clave para mantener la experiencia de usuario fluida y eficiente.

### 3.8.1 ¿Qué son operaciones costosas?

Son cálculos, renderizados o actualizaciones que consumen mucho tiempo o recursos, como:
- Filtrar, ordenar o mapear grandes listas en cada ciclo de cambio.
- Renderizar componentes complejos o con muchos elementos.
- Ejecutar lógica pesada en plantillas o métodos de componentes.
- Realizar peticiones HTTP o cálculos asíncronos en bucles de renderizado.

### 3.8.2 Cómo identificar operaciones costosas

- Usa las herramientas de desarrollo del navegador (Performance, Profiler) para analizar el tiempo de renderizado y los ciclos de Change Detection.
- Utiliza el modo de desarrollo de Angular y activa el flag de profiling para ver qué componentes se recalculan más.
- Añade logs temporales en métodos, `computed()` y `effect()` para detectar llamadas excesivas.
- Observa el comportamiento de la UI: si hay lentitud al interactuar, al cambiar datos o al navegar entre vistas, puede haber operaciones costosas.

### 3.8.3 Técnicas de optimización

#### 1. Memoización de cálculos

Usa `computed()` para cachear resultados de cálculos derivados y evitar repetir operaciones pesadas.

**Ejemplo:**
```ts
const productos = signal<Product[]>([]);
const filtro = signal<string>('');
const productosFiltrados = computed(() =>
	productos().filter(p => p.nombre.includes(filtro()))
);
```

#### 2. Virtualización de listas

Muestra solo los elementos visibles en pantalla usando librerías de virtual scroll o componentes personalizados. Así, Angular solo renderiza lo necesario.

#### 3. Componentes con Change Detection selectivo

Usa `OnPush` y Signals para limitar la actualización a los componentes que realmente cambian.

#### 4. Evita lógica pesada en plantillas

Realiza cálculos en métodos, `computed()` o servicios, no directamente en la plantilla. Así, el renderizado es más rápido y predecible.

#### 5. Divide componentes grandes en subcomponentes

Fragmenta la UI en piezas pequeñas y reutilizables, cada una con su propio ciclo de Change Detection. Esto reduce el impacto de operaciones costosas en el árbol global.

#### 6. Usa trackBy en listas

En estructuras de control de flujo, como `@for()`, utiliza `track` para evitar recrear elementos innecesariamente.

**Ejemplo:**
```html
@for(item of items(); track item.id) {
	<li>{{ item.nombre }}</li>
}
```

#### 7. Debounce y throttle en eventos

Si tienes eventos que disparan cálculos o peticiones, usa técnicas de debounce o throttle para limitar la frecuencia de ejecución.

#### 8. Optimiza peticiones HTTP y cálculos asíncronos

Agrupa, cachea o limita las peticiones y cálculos que se disparan en respuesta a cambios de estado.

### 3.8.4 Recomendaciones finales

- Perfila tu aplicación regularmente y revisa los puntos críticos.
- Aplica optimizaciones solo donde realmente impactan el rendimiento.
- Documenta las zonas optimizadas para facilitar el mantenimiento.
- Aprovecha Signals, `computed()` y `effect()` para gestionar la reactividad de forma eficiente.



## 3.9 Integración con Angular DevTools para profiling y debugging de Change Detection

Cuando trabajamos con **Angular Signals** y optimizaciones de rendimiento, entender **cuándo** y **por qué** se ejecuta el *Change Detection* es clave.  
Aquí es donde **Angular DevTools** se convierte en nuestro aliado: una extensión oficial para Chrome y Firefox que nos permite **inspeccionar, medir y depurar** el ciclo de vida de nuestra aplicación.

### 3.9.1. Instalación y requisitos

1. **Instala la extensión** desde la [Chrome Web Store](https://chrome.google.com/webstore/detail/angular-devtools) o [Firefox Add-ons](https://addons.mozilla.org/es/firefox/addon/angular-devtools/).
2. Abre las herramientas de desarrollo del navegador y busca la pestaña **Angular**.


### 3.9.2. Explorando la aplicación con *Component Explorer*

El **Component Explorer** muestra la jerarquía de componentes como un árbol interactivo:

- **Selecciona un componente** para ver:
  - Sus *Inputs* y *Outputs*.
  - Sus propiedades internas.
  - El estado actual de sus Signals.
- Esto es muy útil para:
  - Ver cómo cambian los datos en tiempo real.
  - Confirmar si un cambio en un Signal realmente provoca un *render*.

💡 *Tip*: Si un componente se actualiza sin que esperes que lo haga, revisa aquí si algún *Input* o Signal ha cambiado.

### 3.9.3. Analizando el *Change Detection* con el **Profiler**

El **Profiler** es la joya de Angular DevTools para entender el rendimiento:

1. Ve a la pestaña **Profiler**.
2. Pulsa **Start recording**.
3. Interactúa con tu aplicación (haz clics, cambia Signals, navega).
4. Pulsa **Stop recording**.

Verás:

- **Barras de tiempo**: cada barra representa un ciclo de *Change Detection*.  
  - Barras más altas = más tiempo de procesamiento.
  - Colores amarillos o rojos = posibles cuellos de botella.
- **Flame Graph**: muestra qué componentes consumen más tiempo.
  - El ancho indica duración.
  - La profundidad indica jerarquía de llamadas.
- **Detalles por componente**: al seleccionar una barra, verás:
  - Tiempo total de *Change Detection*.
  - Qué lo disparó (evento, Signal, HTTP, etc.).
  - Componentes y directivas implicadas.

### 3.9.4. Detectando problemas comunes

Con el Profiler puedes descubrir:

- **Actualizaciones innecesarias**: componentes que se vuelven a renderizar sin que sus datos cambien.
- **Bucles de renderizado**: Signals o eventos que disparan *Change Detection* repetidamente.
- **Componentes pesados**: aquellos que tardan demasiado en procesar su vista.

Ejemplo:  
Si ves que un `UserListComponent` se actualiza cada vez que mueves el ratón sobre otro elemento, probablemente haya un evento global o un Signal mal gestionado.

### 3.9.5. Estrategias de optimización

- Usa **`ChangeDetectionStrategy.OnPush`** en componentes que no necesitan actualizarse con cada cambio global.
- Divide componentes grandes en otros más pequeños y específicos.
- Evita mutar objetos o arrays directamente; crea nuevas referencias para Signals.
- Controla la frecuencia de actualizaciones con *debounce* o *throttle* en eventos intensivos.

### 3.9.6 Caso de estudio: Signal mal gestionado y *Change Detection* excesivo

**El escenario inicial**

Imagina que tenemos un componente que muestra una lista de usuarios y un contador de clics. El contador está implementado como un **Signal global** que se actualiza en cada clic… pero sin aislar su efecto.

```ts
import { Component, signal } from '@angular/core';

const clickCount = signal(0);

@Component({
  selector: 'app-user-list',
  template: `
    <button (click)="increment()">Clics: {{ clickCount() }}</button>
    <ul>
		@for(user of users; track $index) {
			<li>
				{{ user.name }}
			</li>
		}
    </ul>
  `
})
export class UserListComponent {
  users = Array.from({ length: 5000 }, (_, i) => ({ name: `Usuario ${i + 1}` }));
  clickCount = clickCount;

  increment() {
    this.clickCount.update(c => c + 1);
  }
}
```

**Problema**:  
Cada vez que hacemos clic, **toda la lista de 5000 usuarios se vuelve a renderizar**, aunque no haya cambiado.

---

**Detectando el problema con Angular DevTools**

1. Abrimos **Angular DevTools** y vamos a la pestaña **Profiler**.
2. Pulsamos **Start recording**.
3. Hacemos un par de clics en el botón.
4. Pulsamos **Stop recording**.

**Qué vemos**:
- El *Flame Graph* muestra que `UserListComponent` se vuelve a renderizar completo en cada clic.
- El tiempo de *Change Detection* es alto y crece con el tamaño de la lista.
- El disparador del cambio es el Signal `clickCount`, que está en el mismo componente que la lista.

---

**Optimizando el código**

La solución es **aislar el Signal** para que solo afecte a la parte necesaria de la vista.

```ts
@Component({
  selector: 'app-click-counter',
  template: `<button (click)="increment()">Clics: {{ clickCount() }}</button>`
})
export class ClickCounterComponent {
  clickCount = clickCount;
  increment() {
    this.clickCount.update(c => c + 1);
  }
}

@Component({
  selector: 'app-user-list',
  template: `
    <app-click-counter />
	<ul>
		@for(user of users; track $index) {
			<li>
				{{ user.name }}
			</li>
		}
	</ul>
  `
})
export class UserListComponent {
  users = Array.from({ length: 5000 }, (_, i) => ({ name: `Usuario ${i + 1}` }));
}
```

**Qué cambia**:
- El contador está en un componente separado.
- Ahora, al hacer clic, solo se vuelve a renderizar `ClickCounterComponent`.
- La lista permanece intacta.

---

**Verificando la mejora con DevTools**

Repetimos el *profiling*:

- El *Flame Graph* muestra que solo el componente del contador se actualiza.
- El tiempo de *Change Detection* se reduce drásticamente.
- La experiencia del usuario es más fluida.

## 3.10 Buenas prácticas de monitorización y diagnóstico en aplicaciones enterprise 

En aplicaciones **enterprise**, el rendimiento y la estabilidad no son opcionales: son requisitos críticos. Una aplicación puede funcionar bien en desarrollo, pero en producción se enfrenta a **miles de usuarios concurrentes, cargas variables y entornos distribuidos**.  
Por eso, además de escribir buen código, necesitamos **monitorizar, diagnosticar y anticipar problemas**.


### 3.10.1. Instrumentación desde el inicio

- **No esperes a tener problemas**: define desde el primer sprint cómo vas a medir el rendimiento.
- Añade **logs estructurados** (JSON, con nivel de severidad) en lugar de simples `console.log`.
- Usa **interceptores HTTP** para registrar:
  - Latencia de peticiones.
  - Errores de red.
  - Respuestas lentas o con códigos inesperados.

Ejemplo de interceptor de logging:

```ts
import { HttpRequest, HttpHandlerFn, HttpEvent, HttpEventType } from '@angular/common/http';
import { Observable, tap } from 'rxjs';

// Interceptor funcional para logging de peticiones y tiempos de respuesta
export function loggingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn
): Observable<HttpEvent<unknown>> => {
  const start = performance.now();

  return next(req).pipe(
    tap(event => {
      if (event.type === HttpEventType.Response) {
        const duration = performance.now() - start;
        console.info(`[HTTP] ${req.method} ${req.url} - ${duration.toFixed(2)}ms`);
      }
    },
    error => {
      console.error(`[HTTP ERROR] ${req.method} ${req.url}`, error);
    })
  );
};

```

### 3.10.2. Uso de Angular DevTools en producción controlada

- Angular DevTools es ideal en desarrollo, pero en entornos enterprise también puedes habilitarlo en **entornos de staging** para reproducir problemas reales.
- Haz *profiling* de:
  - Ciclos de *Change Detection*.
  - Componentes que consumen más tiempo.
  - Señales que disparan renders innecesarios.

### 3.10.3. Integración con herramientas de observabilidad

En aplicaciones grandes, no basta con logs locales. Necesitamos **plataformas centralizadas**:

- **Application Performance Monitoring (APM)**:  
  Ejemplos: Azure Application Insights, Elastic APM, Datadog, New Relic.  
  Permiten:
  - Trazar peticiones de extremo a extremo (frontend → backend).
  - Detectar cuellos de botella en tiempo real.
  - Alertar automáticamente ante errores críticos.

- **Error tracking**:  
  Herramientas como Sentry o Rollbar capturan excepciones no controladas y las agrupan por frecuencia, versión y usuario afectado.

### 3.10.4. Estrategias de logging y métricas

- **Separar niveles de log**: `debug`, `info`, `warn`, `error`.  
  Así puedes filtrar según el entorno.
- **Correlación de logs**: añade un *requestId* o *traceId* a cada petición para seguirla en todo el sistema.
- **Métricas clave a monitorizar**:
  - Tiempo de renderizado inicial.
  - Latencia de peticiones HTTP.
  - Errores de usuario (formularios, validaciones).
  - Uso de memoria y CPU en dispositivos cliente.

### 3.10.5. Diagnóstico proactivo

- **Alertas tempranas**: configura umbrales (ej. si el tiempo medio de respuesta supera 2s).
- **Dashboards en tiempo real**: gráficos de rendimiento y errores accesibles al equipo.
- **Pruebas de carga periódicas**: simula tráfico masivo para anticipar problemas antes de que ocurran.

### 3.10.6. Buenas prácticas específicas para Angular + Signals

- Evita Signals globales que disparen renders en cascada.  
- Usa `ChangeDetectionStrategy.OnPush` en componentes críticos.  
- Monitoriza con DevTools qué Signals provocan más *Change Detection*.  
- Registra métricas de uso de Signals en componentes de alto tráfico.
