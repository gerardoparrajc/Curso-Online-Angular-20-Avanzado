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

