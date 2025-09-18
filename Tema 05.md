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

