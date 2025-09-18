# 5. Formularios avanzados en Angular 20

## 5.1 Formularios fuertemente tipados (Typed Forms) y sus ventajas

Uno de los avances más importantes en la evolución de los formularios de Angular es la llegada de los **formularios fuertemente tipados** (*Typed Forms*). Esta característica, introducida inicialmente en Angular 14 y consolidada en versiones posteriores hasta Angular 20, responde a una necesidad muy concreta: **garantizar la seguridad de tipos en los formularios reactivos** y mejorar la experiencia de desarrollo en proyectos grandes y complejos.

---

### 5.1.1. ¿Qué son los Typed Forms?

En versiones anteriores de Angular, los formularios reactivos (`FormGroup`, `FormControl`, `FormArray`) eran **flexibles pero no estrictamente tipados**. Esto significaba que podías cometer errores como:

- Referirte a un control que no existía en el `FormGroup`.  
- Asignar un valor de tipo incorrecto a un `FormControl` (por ejemplo, un número en un campo que esperaba un string).  
- No tener autocompletado ni ayuda del IDE al trabajar con los valores del formulario.  

Con los **Typed Forms**, cada control, grupo o array está **estrictamente tipado**, lo que permite a TypeScript detectar errores en tiempo de compilación en lugar de en tiempo de ejecución.

---

### 5.1.2. Ejemplo comparativo

#### Antes (formularios no tipados)

```ts
profileForm = new FormGroup({
  firstName: new FormControl(''),
  age: new FormControl(0)
});

// Podrías cometer errores como:
profileForm.controls['fristName'].setValue('Ana'); // ¡typo! No da error en compilación
profileForm.controls['age'].setValue('texto');     // Valor incorrecto, falla en runtime
```

#### Ahora (formularios tipados)

```ts
interface ProfileForm {
  firstName: FormControl<string>;
  age: FormControl<number>;
}

profileForm = new FormGroup<ProfileForm>({
  firstName: new FormControl('', { nonNullable: true }),
  age: new FormControl(0, { nonNullable: true })
});

// Errores detectados en compilación:
profileForm.controls['fristName'].setValue('Ana'); // ❌ Error: no existe 'fristName'
profileForm.controls['age'].setValue('texto');     // ❌ Error: se esperaba un número
```

👉 Con Typed Forms, el compilador de TypeScript se convierte en tu primera línea de defensa contra errores comunes.

---

### 5.1.3. Ventajas principales

#### Seguridad de tipos
- Evita errores de asignación de valores incorrectos.  
- Detecta referencias a controles inexistentes.  

#### Productividad en el IDE
- Autocompletado de nombres de controles.  
- Sugerencias de métodos y propiedades válidas.  
- Reducción de tiempo de depuración.  

#### Refactorización más segura
- Si cambias el nombre de un control en el `FormGroup`, el compilador te avisará en todos los lugares donde se usa.  
- Esto es especialmente útil en proyectos enterprise con formularios grandes y complejos.  

#### Validaciones más claras
- Los validadores personalizados también se benefician de la tipificación, ya que reciben valores del tipo correcto.  

Ejemplo:

```ts
function adultValidator(control: FormControl<number>) {
  return control.value >= 18 ? null : { ageInvalid: true };
}
```

#### Escalabilidad
- En aplicaciones grandes, donde los formularios pueden tener decenas de campos, la tipificación estricta reduce errores y facilita el mantenimiento a largo plazo.  

---

### 5.1.4. Compatibilidad y adopción gradual

- Angular permite **usar formularios tipados y no tipados en paralelo**, lo que facilita la migración progresiva en proyectos legacy.  
- No es necesario reescribir todos los formularios de golpe: puedes empezar a tipar los nuevos y migrar los antiguos poco a poco.  

## 5.2 Creación de formularios híbridos (Typed + Signals)

Los **formularios híbridos** en Angular 20 combinan dos de las innovaciones más potentes del framework en los últimos años:  
- La **tipificación estricta** de los **Typed Forms**, que garantiza seguridad de tipos y autocompletado en el IDE.  
- La **reactividad declarativa** de los **Signals**, que permiten que el estado del formulario se integre de forma natural en el flujo reactivo de la aplicación.  

Este enfoque híbrido ofrece una experiencia de desarrollo más **segura, expresiva y reactiva**, ideal para aplicaciones enterprise donde los formularios suelen ser complejos y críticos.

### 5.2.1. ¿Por qué formularios híbridos?

En los formularios reactivos tradicionales, incluso con tipado, la reactividad se gestionaba principalmente con **Observables** (`valueChanges`, `statusChanges`). Esto implicaba:  
- Suscripciones manuales.  
- Posible necesidad de desuscribirse para evitar fugas de memoria.  
- Código más imperativo.  

Con Signals, podemos **exponer el estado del formulario como Signals** y reaccionar automáticamente a cambios de valores, estados de validación o flags como `dirty`, `touched` o `valid`.

---

### 5.2.2. Ejemplo básico: Typed Form + Signal

```ts
import { Component, signal, effect } from '@angular/core';
import { ReactiveFormsModule, FormControl, FormGroup, Validators } from '@angular/forms';

interface LoginForm {
  email: FormControl<string>;
  password: FormControl<string>;
}

@Component({
  selector: 'app-login',
  imports: [ReactiveFormsModule]
  template: `
    <form [formGroup]="form">
      <input formControlName="email" placeholder="Email" />
      <input formControlName="password" type="password" placeholder="Password" />
      <button [disabled]="!isValid()">Entrar</button>
    </form>
    @if (isDirty()) {
      <p>El formulario ha sido modificado</p>
    }
  `
})
export class LoginComponent {
  form = new FormGroup<LoginForm>({
    email: new FormControl('', { nonNullable: true, validators: [Validators.required, Validators.email] }),
    password: new FormControl('', { nonNullable: true, validators: [Validators.required, Validators.minLength(6)] })
  });

  // Signals derivados del formulario
  emailValue = signal(this.form.controls.email.value);
  passwordValue = signal(this.form.controls.password.value);
  isValid = signal(this.form.valid);
  isDirty = signal(this.form.dirty);

  constructor() {
    // Sincronizamos los Signals con el formulario
    this.form.valueChanges.subscribe(value => {
      this.emailValue.set(value.email ?? '');
      this.passwordValue.set(value.password ?? '');
      this.isValid.set(this.form.valid);
      this.isDirty.set(this.form.dirty);
    });

    // Ejemplo de efecto reactivo
    effect(() => {
      if (!this.isValid()) {
        console.log('Formulario inválido');
      }
    });
  }
}
```

👉 Aquí vemos cómo un formulario tipado se convierte en **fuente de Signals**, lo que nos permite usar `effect()` para reaccionar automáticamente a cambios de estado.

### 5.2.3. Ventajas del enfoque híbrido

- **Seguridad de tipos**: los campos del formulario están estrictamente tipados.  
- **Reactividad declarativa**: los Signals permiten reaccionar a cambios sin suscripciones manuales dispersas.  
- **Menos boilerplate**: se reducen las necesidades de `ngOnDestroy` y `unsubscribe`.  
- **Integración natural con la UI**: los Signals pueden usarse directamente en bindings (`[disabled]="!isValid()"`).  
- **Escalabilidad**: en formularios grandes, se pueden derivar Signals específicos para secciones concretas, mejorando la legibilidad.  

### 5.2.4. Ejemplo avanzado: validaciones reactivas con Signals

Podemos incluso crear validaciones personalizadas que dependan de Signals externos:

```ts
@Component({
  selector: 'app-register',
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form">
        <input formControlName="password" type="password" placeholder="Contraseña" />
        <input formControlName="confirmPassword" type="password" placeholder="Confirmar contraseña" />

        @if (!passwordsMatch()) {
            <p>Las contraseñas no coinciden</p>
        }
    </form>
  `
})
export class RegisterComponent {
  form = new FormGroup({
    password: new FormControl('', { nonNullable: true }),
    confirmPassword: new FormControl('', { nonNullable: true })
  });

  password = signal(this.form.controls.password.value);
  confirmPassword = signal(this.form.controls.confirmPassword.value);

  passwordsMatch = signal(true);

  constructor() {
    this.form.valueChanges.subscribe(value => {
      this.password.set(value.password ?? '');
      this.confirmPassword.set(value.confirmPassword ?? '');
      this.passwordsMatch.set(this.password() === this.confirmPassword());
    });
  }
}
```

👉 Aquí, la validación de contraseñas se convierte en un **Signal derivado**, lo que simplifica la lógica y la hace más declarativa.

### 5.2.5. Buenas prácticas

- **Encapsula lógica en Signals derivados**: evita recalcular validaciones en múltiples lugares.  
- **Usa `effect()` para efectos secundarios** (logs, notificaciones, activación de botones).  
- **Mantén la tipificación estricta**: define interfaces para tus formularios y evita `any`.  
- **Migra progresivamente**: puedes empezar con formularios tipados y añadir Signals poco a poco.  

## 5.3 Validaciones síncronas y asíncronas aplicadas con Signals

En Angular 20, los **Signals** no solo sirven para gestionar el estado de forma reactiva, sino que también se integran de manera natural con los **formularios tipados** y sus validaciones. Esto abre la puerta a un modelo de validación más **declarativo, predecible y seguro**, donde las reglas de negocio se expresan como funciones puras y los resultados se propagan automáticamente a la interfaz de usuario.

### 5.3.1. Validaciones síncronas con Signals

Las validaciones síncronas son aquellas que pueden evaluarse inmediatamente, sin necesidad de esperar a un proceso externo (como una petición HTTP). Ejemplos típicos:  
- Campos obligatorios.  
- Longitud mínima o máxima.  
- Comparación entre dos valores (ej. contraseña y confirmación).  

#### Ejemplo: validador de edad mínima

```ts
import { FormControl } from '@angular/forms';
import { signal, effect } from '@angular/core';

function minAgeValidator(min: number) {
  return (control: FormControl<number>) => {
    return control.value >= min ? null : { minAge: true };
  };
}

const ageControl = new FormControl(0, { 
  nonNullable: true, 
  validators: [minAgeValidator(18)] 
});

// Signal derivado del estado de validación
const isAdult = signal(ageControl.valid);

ageControl.valueChanges.subscribe(value => {
  isAdult.set(ageControl.valid);
});

// Reactividad declarativa
effect(() => {
  console.log(isAdult() ? 'Edad válida' : 'Debe ser mayor de edad');
});
```

👉 Aquí, el Signal `isAdult` refleja automáticamente el estado de validación del control, y cualquier cambio en el valor dispara la reevaluación.

### 5.3.2. Validaciones asíncronas con Signals

Las validaciones asíncronas requieren consultar una fuente externa, como un servicio HTTP o una base de datos. Ejemplo clásico: comprobar si un nombre de usuario ya existe.  

En Angular 20, podemos combinar **Observables** con **Signals** usando el helper `toSignal()`, lo que nos permite integrar validaciones asíncronas en el flujo reactivo sin necesidad de suscripciones manuales.

#### Ejemplo: validador de nombre de usuario único

```ts
import { Component, signal, effect, inject } from '@angular/core';
import { FormControl, FormGroup, Validators } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';

interface RegisterForm {
  username: FormControl<string>;
}

@Component({
  selector: 'app-register',
  standalone: true,
  template: `
    <form [formGroup]="form">
      <label>
        Nombre de usuario:
        <input formControlName="username" />
      </label>

      <p *ngIf="usernameControl.pending">Comprobando disponibilidad...</p>
      <p *ngIf="!isAvailable()">El nombre de usuario ya está en uso</p>
      <p *ngIf="isAvailable()">Nombre de usuario disponible ✅</p>

      <button [disabled]="!form.valid">Registrar</button>
    </form>
  `
})
export class RegisterComponent {
  private http = inject(HttpClient);

  form = new FormGroup<RegisterForm>({
    username: new FormControl<string>('', {
      nonNullable: true,
      validators: [Validators.required, Validators.minLength(3)],
      asyncValidators: [this.usernameAvailableValidator()],
      updateOn: 'blur'
    })
  });

  usernameControl = this.form.controls.username;

  // Signal que refleja la disponibilidad (derivado del validador asíncrono)
  isAvailable = signal(true);

  constructor() {
    // Creamos un Signal a partir de statusChanges (Observable)
    const statusSignal = toSignal(this.usernameControl.statusChanges, {
      initialValue: this.usernameControl.status
    });

    // Efecto reactivo: cada vez que cambia el estado, actualizamos isAvailable
    effect(() => {
      const status = statusSignal();
      const taken = this.usernameControl.hasError('usernameTaken');
      this.isAvailable.set(status === 'VALID' && !taken);
      console.log('isAvailable:', this.isAvailable());
    });
  }

  // Validador asíncrono: consulta al backend
  private usernameAvailableValidator(): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      const value = control.value?.trim();
      if (!value) return of(null);

      return timer(300).pipe(
        switchMap(() =>
          this.http.get<{ available: boolean }>(`/api/users/check/${encodeURIComponent(value)}`)
        ),
        map(res => (res.available ? null : { usernameTaken: true })),
        catchError(() => of(null))
      );
    };
  }
}

```

#### Qué hace este componente

- **Typed Form**: el `FormControl<string>` garantiza seguridad de tipos.  
- **AsyncValidator**: sigue devolviendo un `Observable<ValidationErrors | null>`, como exige Angular.  
- **toSignal()**: convertimos `statusChanges` en un Signal (`statusSignal`).  
- **Signal derivado (`isAvailable`)**: refleja en tiempo real si el usuario está disponible.  
- **UI declarativa**: la plantilla consume `isAvailable()` directamente, sin suscripciones manuales.  
 

#### Ventajas de este enfoque

- **Reactividad total**: el estado de validación fluye como Signals.  
- **Menos boilerplate**: no hay `subscribe/unsubscribe` manuales.  
- **Integración natural con la UI**: la plantilla usa Signals como si fueran propiedades.  
- **Escalabilidad**: puedes derivar múltiples Signals (ej. `isPending`, `hasErrors`, `errorMessage`) para formularios grandes.  
  


### 5.3.3. Ventajas de usar Signals en validaciones

- **Reactividad declarativa**: no necesitas suscripciones manuales dispersas, los Signals se actualizan automáticamente.  
- **Menos boilerplate**: se reduce la necesidad de `ngOnDestroy` o `unsubscribe`.  
- **Integración natural con la UI**: puedes usar los Signals directamente en plantillas (`[disabled]="!isAvailable()"`).  
- **Escalabilidad**: en formularios grandes, puedes derivar Signals específicos para cada regla de validación.  
- **Compatibilidad**: puedes seguir usando validadores clásicos, pero ahora con la posibilidad de exponer su estado como Signals.  

### 5.3.4. Buenas prácticas

- **Encapsula validaciones en funciones puras**: facilita su testeo y reutilización.  
- **Usa Signals derivados para estados de validación**: evita recalcular en múltiples lugares.  
- **Combina `toSignal()` con validaciones asíncronas**: simplifica la integración con servicios HTTP.  
- **Muestra feedback inmediato en la UI**: los Signals permiten reflejar estados como `pending`, `valid` o `invalid` en tiempo real.  

## 5.4 Personalización de mensajes de error reactivos y dinámicos

En Angular 20, la gestión de errores en formularios da un salto cualitativo gracias a la combinación de **Typed Forms**, **Signals** y el **nuevo control de flujo en plantillas**.  
Ya no es necesario llenar la vista de múltiples condiciones con `*ngIf`: ahora podemos centralizar la lógica de errores en **Signals derivados** y mostrarlos de forma **declarativa y dinámica** con `@if` y `@switch`.

### 5.4.1. El problema de los mensajes estáticos

Tradicionalmente, los mensajes de error se escribían así:

```html
<div *ngIf="control.errors?.required">Campo obligatorio</div>
<div *ngIf="control.errors?.minlength">Mínimo 6 caracteres</div>
<div *ngIf="control.errors?.email">Formato inválido</div>
```

Esto genera **duplicación de código**, poca flexibilidad y plantillas difíciles de mantener.

---

### 5.4.2. Creación de un Signal de error

Para que los mensajes sean **reactivos**, necesitamos que dependan de un Signal. Como `control.errors` no es un Signal, debemos engancharlo a los **observables del control** (`statusChanges`, `valueChanges`) usando `toSignal`.

```ts
import { computed } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { FormControl } from '@angular/forms';

export function createErrorMessage(control: FormControl<any>) {
  const statusSig = toSignal(control.statusChanges, { initialValue: control.status });
  const valueSig = toSignal(control.valueChanges, { initialValue: control.value });

  return computed(() => {
    // Dependencias explícitas
    statusSig();
    valueSig();

    const touched = control.touched || control.dirty;
    const errors = control.errors;

    if (!touched || !errors) return '';

    if (errors['required']) return 'Este campo es obligatorio';
    if (errors['minlength']) return `Mínimo ${errors['minlength'].requiredLength} caracteres`;
    if (errors['email']) return 'Formato de correo inválido';
    if (errors['usernameTaken']) return 'El nombre de usuario ya está en uso';

    return '';
  });
}
```

### 5.4.3. Uso en un componente

```ts
import { Component } from '@angular/core';
import { FormControl, Validators } from '@angular/forms';
import { createErrorMessage } from './error-messages.helper';

@Component({
  selector: 'app-username',
  standalone: true,
  templateUrl: './username.component.html'
})
export class UsernameComponent {
  username = new FormControl('', {
    nonNullable: true,
    validators: [Validators.required, Validators.minLength(3)]
  });

  usernameError = createErrorMessage(this.username);
}
```

### 5.4.4. Plantilla con nuevos bloques de control

```html
<input formControlName="username" placeholder="Nombre de usuario" />

@if (usernameError()) {
  <p class="error">{{ usernameError() }}</p>
}
```

👉 El bloque `@if` muestra el mensaje solo cuando el Signal `usernameError()` devuelve un texto distinto de vacío.

### 5.4.5. Ejemplo con `@switch` para múltiples errores

Si queremos mostrar mensajes distintos según el tipo de error, podemos usar `@switch`:

```html
<input formControlName="email" placeholder="Correo electrónico" />

@switch (true) {
  @case (emailCtrl.touched && emailCtrl.hasError('required')) {
    <p class="error">El correo es obligatorio</p>
  }
  @case (emailCtrl.touched && emailCtrl.hasError('email')) {
    <p class="error">Formato de correo inválido</p>
  }
  @default {
    <!-- Sin errores -->
  }
}
```

### 5.4.6. Ventajas de este enfoque

- **Reactividad real**: los mensajes cambian automáticamente al variar el estado del control.  
- **Plantillas más limpias**: menos condiciones repetidas, más expresividad.  
- **Centralización**: la lógica de errores se concentra en un helper o servicio.  
- **Escalabilidad**: fácil de extender a catálogos de mensajes e internacionalización.  
- **Compatibilidad**: funciona con Typed Forms, validadores síncronos y asíncronos.  

## 5.5 Integración con RxJS: validaciones y sincronización con Observables

Aunque Angular 20 ha potenciado el uso de **Signals** como mecanismo de reactividad, **RxJS** sigue siendo una pieza fundamental del ecosistema. Muchos módulos de Angular (HttpClient, Router, Forms) están construidos sobre **Observables**, y en formularios avanzados es habitual necesitar **validaciones asíncronas** o **sincronización de datos en tiempo real**.  

La clave está en la **interoperabilidad entre Signals y Observables**, que Angular facilita con utilidades como `toSignal` y `toObservable`.


### 5.5.1. Validaciones asíncronas con RxJS

Un caso típico es comprobar en el backend si un valor ya existe (ej. nombre de usuario). Esto se implementa con un **AsyncValidator** que devuelve un `Observable<ValidationErrors | null>`.

```ts
import { FormControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { Observable, of, timer } from 'rxjs';
import { switchMap, map, catchError } from 'rxjs/operators';

function usernameAvailableValidator(http: HttpClient): AsyncValidatorFn {
  return (control: FormControl<string>): Observable<ValidationErrors | null> => {
    const value = control.value?.trim();
    if (!value) return of(null);

    return timer(300).pipe( // debounce
      switchMap(() => http.get<{ available: boolean }>(`/api/users/check/${value}`)),
      map(res => (res.available ? null : { usernameTaken: true })),
      catchError(() => of(null))
    );
  };
}
```

👉 Aquí RxJS nos permite **controlar asincronía, aplicar debounce y manejar errores** de forma declarativa.

### 5.5.2. Exponer validaciones como Signals

Podemos convertir el flujo de validación en un **Signal** con `toSignal`, lo que facilita integrarlo en la UI:

```ts
import { toSignal } from '@angular/core/rxjs-interop';
import { computed } from '@angular/core';

const statusSig = toSignal(usernameControl.statusChanges, { initialValue: usernameControl.status });
const valueSig = toSignal(usernameControl.valueChanges, { initialValue: usernameControl.value });

const usernameError = computed(() => {
  statusSig(); // dependencias
  valueSig();

  if (usernameControl.pending) return '⏳ Comprobando...';
  if (usernameControl.hasError('required')) return 'El usuario es obligatorio';
  if (usernameControl.hasError('minlength')) return 'Mínimo 3 caracteres';
  if (usernameControl.hasError('usernameTaken')) return 'El usuario ya existe';
  return '';
});
```

En la plantilla:

```html
<input formControlName="username" />
@if (usernameError()) {
  <p class="error">{{ usernameError() }}</p>
}
```

### 5.5.3. Sincronización con Observables externos

Además de validaciones, los formularios suelen necesitar **sincronizarse con flujos de datos externos**:  
- Actualizar un formulario con datos de un servicio en tiempo real.  
- Guardar automáticamente cambios en el backend.  
- Integrar formularios con stores de estado (NgRx, Akita, etc.).  

Ejemplo: sincronizar un formulario con un `BehaviorSubject` de perfil de usuario:

```ts
import { BehaviorSubject } from 'rxjs';
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

const user$ = new BehaviorSubject({ name: 'Ana', email: 'ana@test.com' });

// Convertimos el Observable en Signal
const userSig = toSignal(user$, { initialValue: { name: '', email: '' } });

// Formulario tipado
form = new FormGroup({
  name: new FormControl(userSig().name, { nonNullable: true }),
  email: new FormControl(userSig().email, { nonNullable: true })
});

// Convertimos el formulario en Observable para sincronizar cambios
const formChanges$ = toObservable(form.valueChanges);

// Guardamos automáticamente en el backend
formChanges$.pipe(
  switchMap(value => http.post('/api/users/update', value))
).subscribe();
```

👉 Aquí vemos cómo `toSignal` y `toObservable` permiten **puentear Signals y Observables**, logrando sincronización bidireccional.

### 5.5.4. Ventajas de la integración RxJS + Signals

- **Compatibilidad total**: puedes seguir usando RxJS en validadores, peticiones HTTP y stores.  
- **Reactividad declarativa**: los Signals permiten reflejar el estado en la UI sin suscripciones manuales.  
- **Sincronización bidireccional**: `toSignal` y `toObservable` facilitan el intercambio entre ambos mundos.  
- **Escalabilidad**: ideal para formularios enterprise que combinan datos locales y remotos.  

## 5.6 Ejemplos prácticos de formularios enterprise con lógica avanzada

En entornos corporativos, los formularios suelen ser **grandes, dinámicos y con reglas de negocio complejas**. Angular 20, con **Typed Forms**, **Signals**, **RxJS** y los **nuevos bloques de control de flujo**, nos permite construir soluciones robustas, escalables y fáciles de mantener.  

A continuación, veremos tres escenarios típicos en proyectos enterprise.

### 5.6.1. Formulario de registro con validaciones condicionales

**Caso de uso:**  
Un formulario de registro donde ciertos campos aparecen o se validan solo si el usuario selecciona determinadas opciones (ej. empresa vs. particular).

```ts
form = new FormGroup({
  userType: new FormControl<'individual' | 'company'>('individual', { nonNullable: true }),
  companyName: new FormControl('', { nonNullable: true }),
  vatNumber: new FormControl('', { nonNullable: true })
});

// Signal derivado para activar/desactivar validaciones
isCompany = computed(() => this.form.controls.userType.value === 'company');

constructor() {
  effect(() => {
    if (this.isCompany()) {
      this.form.controls.companyName.addValidators([Validators.required]);
      this.form.controls.vatNumber.addValidators([Validators.required]);
    } else {
      this.form.controls.companyName.clearValidators();
      this.form.controls.vatNumber.clearValidators();
    }
    this.form.controls.companyName.updateValueAndValidity();
    this.form.controls.vatNumber.updateValueAndValidity();
  });
}
```

En la plantilla:

```html
<select formControlName="userType">
  <option value="individual">Particular</option>
  <option value="company">Empresa</option>
</select>

@if (isCompany()) {
  <input formControlName="companyName" placeholder="Nombre de la empresa" />
  <input formControlName="vatNumber" placeholder="NIF/CIF" />
}
```

👉 Aquí vemos cómo **Signals** permiten activar validaciones dinámicas y mostrar campos condicionales de forma declarativa.

### 5.6.2. Formulario dinámico con arrays de controles

**Caso de uso:**  
Un formulario de pedidos donde el usuario puede añadir o eliminar productos dinámicamente.

```ts
form = new FormGroup({
  customer: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
  items: new FormArray<FormGroup<{
    product: FormControl<string>;
    quantity: FormControl<number>;
  }>>([])
});

// Método para añadir un producto
addItem() {
  this.form.controls.items.push(new FormGroup({
    product: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
    quantity: new FormControl(1, { nonNullable: true, validators: [Validators.min(1)] })
  }));
}
```

En la plantilla:

```html
<input formControlName="customer" placeholder="Cliente" />

@for (item of form.controls.items.controls; track $index) {
  <div>
    <input [formControl]="item.controls.product" placeholder="Producto" />
    <input type="number" [formControl]="item.controls.quantity" />
    <button (click)="form.controls.items.removeAt($index)">Eliminar</button>
  </div>
}

<button (click)="addItem()">Añadir producto</button>
```

👉 Con `@for` y Typed Forms, la gestión de listas dinámicas es más clara y segura.

### 5.6.3. Formulario con sincronización en tiempo real (RxJS + Signals)

**Caso de uso:**  
Un formulario de perfil que se sincroniza automáticamente con el backend cada vez que cambia.

```ts
import { toObservable } from '@angular/core/rxjs-interop';
import { switchMap } from 'rxjs/operators';

form = new FormGroup({
  name: new FormControl('', { nonNullable: true }),
  email: new FormControl('', { nonNullable: true, validators: [Validators.email] })
});

constructor(private http: HttpClient) {
  // Convertimos valueChanges en Observable
  const formChanges$ = toObservable(this.form.valueChanges);

  formChanges$.pipe(
    switchMap(value => this.http.post('/api/profile/update', value))
  ).subscribe();
}
```

👉 Aquí, **RxJS** maneja la asincronía y la comunicación con el servidor, mientras que **Signals** pueden usarse para reflejar estados de carga o éxito en la UI.

## 5.7 Buenas prácticas y estrategias de migración para equipos grandes

La adopción de Angular 20 en proyectos enterprise no es solo una cuestión técnica: implica coordinar equipos grandes, mantener la productividad y garantizar que la transición no afecte a la calidad del software. A continuación, se presentan **buenas prácticas y estrategias de migración** que facilitan este proceso.

### 5.7.1. Principios clave de migración

- **Migración incremental**: no intentes actualizar todo de golpe. Angular ofrece herramientas como `ng update` y migraciones automáticas para avanzar paso a paso.  
- **Compatibilidad progresiva**: Angular 20 mantiene compatibilidad con formularios no tipados y APIs previas, lo que permite migrar gradualmente sin bloquear el desarrollo.  
- **Soporte extendido**: cada versión cuenta con soporte activo y LTS, lo que da margen para planificar la transición sin prisas.  

### 5.7.2. Buenas prácticas para equipos grandes

#### 🔹 Organización del código
- **Standalone Components**: adopta componentes, directivas y pipes independientes para reducir la dependencia de NgModules.  
- **Estructura por funcionalidades**: organiza el código por dominios de negocio (ej. `usuarios/`, `pedidos/`) en lugar de separar por tipo de archivo.  
- **Uso de monorepos con Nx**: facilita la modularización, la compartición de librerías internas y la optimización de builds en CI/CD.  

#### 🔹 Estrategias de validación y formularios
- **Typed Forms primero**: prioriza migrar formularios críticos a tipados para reducir errores en producción.  
- **Signals progresivos**: introduce Signals en validaciones y estados de formularios de forma gradual, manteniendo compatibilidad con Observables.  
- **Catálogo centralizado de errores**: evita duplicación de mensajes y facilita la internacionalización.  

#### 🔹 Calidad y colaboración
- **Automatización de pruebas**: integra pruebas unitarias y E2E (Cypress, Playwright) en la pipeline de CI/CD.  
- **Linting y formateo**: aplica ESLint y Prettier para mantener consistencia en equipos grandes.  
- **Documentación viva**: mantén guías internas de migración y ejemplos de patrones recomendados.  

### 5.7.3. Estrategias de migración en fases

1. **Preparación**  
   - Actualizar Node.js y TypeScript a versiones compatibles.  
   - Limpiar imports obsoletos y APIs deprecated.  

2. **Migración técnica**  
   - Migrar primero componentes aislados a standalone.  
   - Adoptar el nuevo control flow (`@if`, `@for`, `@switch`) en plantillas.  
   - Convertir formularios clave a Typed Forms.  

3. **Optimización**  
   - Introducir Signals en validaciones y sincronización de datos.  
   - Refactorizar servicios a `inject()` en lugar de inyección por constructor.  
   - Implementar lazy loading en rutas para mejorar rendimiento.  

4. **Consolidación**  
   - Centralizar mensajes de error y validaciones.  
   - Revisar arquitectura de estado (Signals + RxJS o NgRx).  
   - Establecer métricas de calidad y rendimiento post-migración.  
