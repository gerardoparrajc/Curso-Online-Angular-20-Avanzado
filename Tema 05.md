# 5. Formularios avanzados en Angular 20

## 5.1 Formularios fuertemente tipados (Typed Forms) y sus ventajas

Los **Formularios fuertemente tipados** (Typed Forms) son una de las mejoras más significativas introducidas en Angular desde la versión 14 y perfeccionadas en Angular 20. Esta funcionalidad proporciona **seguridad de tipos en tiempo de compilación** para todos los aspectos de los formularios reactivos, eliminando una de las principales fuentes de errores en aplicaciones Angular complejas.

### ¿Qué son los Typed Forms?

Los Typed Forms extienden la funcionalidad de los formularios reactivos tradicionales añadiendo **tipado estático completo** a través de TypeScript. Esto significa que el compilador conoce exactamente la estructura, tipos de datos y validaciones de cada campo del formulario en tiempo de compilación.

**Antes (Formularios tradicionales):**
```ts
// Sin tipado: cualquier error se detecta solo en runtime
const form = this.fb.group({
  nombre: [''],
  edad: [0],
  email: ['']
});

// Propenso a errores - no hay validación de tipos
const nombre = form.get('nombre')?.value; // tipo: any
const edadIncorrecta = form.get('edad')?.value; // tipo: any
```

**Ahora (Typed Forms en Angular 20):**
```ts
// Con tipado completo: errores detectados en tiempo de compilación
interface Usuario {
  nombre: string;
  edad: number;
  email: string;
}

const form = this.fb.group<Usuario>({
  nombre: ['', Validators.required],
  edad: [0, [Validators.required, Validators.min(18)]],
  email: ['', [Validators.required, Validators.email]]
});

// Completamente tipado y seguro
const nombre: string = form.controls.nombre.value; // Tipo conocido: string
const edad: number = form.controls.edad.value;     // Tipo conocido: number
```

### Ventajas principales de los Typed Forms

#### 1. **Seguridad de tipos en tiempo de compilación**

La ventaja más importante es que TypeScript puede validar que estás accediendo a campos que existen y con los tipos correctos.

```ts
interface ProductoFormValue {
  nombre: string;
  precio: number;
  categoria: string;
  disponible: boolean;
}

const productoForm = this.fb.group<ProductoFormValue>({
  nombre: ['', Validators.required],
  precio: [0, [Validators.required, Validators.min(0)]],
  categoria: ['', Validators.required],
  disponible: [true]
});

// Autocompletado y verificación de tipos automática
productoForm.controls.nombre.setValue('Laptop Gaming'); // ✓ Correcto
productoForm.controls.precio.setValue(999.99);          // ✓ Correcto
productoForm.controls.precio.setValue('caro');          // ✗ Error de compilación!
```

#### 2. **Autocompletado inteligente en el IDE**

Tu editor conoce exactamente qué propiedades están disponibles y te ayuda con autocompletado preciso.

```ts
// Al escribir "form.controls." el IDE te muestra exactamente:
// - nombre (string)
// - precio (number) 
// - categoria (string)
// - disponible (boolean)
```

#### 3. **Refactoring seguro**

Cuando cambias la estructura de un formulario, TypeScript te ayuda a encontrar todos los lugares que necesitan actualizarse.

```ts
interface UsuarioForm {
  // Cambio: 'nombre' -> 'nombreCompleto'
  nombreCompleto: string; // Era: nombre: string
  edad: number;
  email: string;
}

// Al cambiar la interfaz, TypeScript marcará TODOS los lugares 
// donde se usa 'nombre' y necesitas cambiarlos a 'nombreCompleto'
```

#### 4. **Validación de estructura en tiempo de compilación**

El compilador verifica que la estructura del formulario coincida exactamente con la interfaz definida.

```ts
interface ContactoForm {
  nombre: string;
  telefono: string;
  mensaje: string;
}

// Error de compilación si falta algún campo o hay campos extra
const contactoForm = this.fb.group<ContactoForm>({
  nombre: [''],
  telefono: [''],
  // mensaje: ['']  // ERROR: falta el campo 'mensaje'
  extra: ['']    // ERROR: campo 'extra' no existe en la interfaz
});
```

### Implementación práctica paso a paso

#### Paso 1: Definir la interfaz del formulario

```ts
interface RegistroUsuarioForm {
  datosPersonales: {
    nombre: string;
    apellidos: string;
    fechaNacimiento: Date;
  };
  contacto: {
    email: string;
    telefono: string;
  };
  preferencias: {
    newsletter: boolean;
    idioma: string;
  };
}
```

#### Paso 2: Crear el formulario tipado

```ts
import { Component, inject } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-registro',
  template: `...` // veremos la template después
})
export class RegistroComponent {
  private fb = inject(FormBuilder);

  registroForm = this.fb.group<RegistroUsuarioForm>({
    datosPersonales: this.fb.group({
      nombre: ['', [Validators.required, Validators.minLength(2)]],
      apellidos: ['', [Validators.required, Validators.minLength(2)]],
      fechaNacimiento: [new Date(), Validators.required]
    }),
    contacto: this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      telefono: ['', [Validators.required, Validators.pattern(/^\d{9}$/)]]
    }),
    preferencias: this.fb.group({
      newsletter: [false],
      idioma: ['es', Validators.required]
    })
  });
}
```

#### Paso 3: Acceso tipado a los controles

```ts
export class RegistroComponent {
  // Acceso tipado completo
  get nombreControl() {
    return this.registroForm.controls.datosPersonales.controls.nombre;
  }

  get emailControl() {
    return this.registroForm.controls.contacto.controls.email;
  }

  onSubmit() {
    if (this.registroForm.valid) {
      // El valor está completamente tipado
      const formValue: RegistroUsuarioForm = this.registroForm.value;
      
      // Acceso seguro a propiedades anidadas
      console.log('Nombre:', formValue.datosPersonales.nombre);
      console.log('Email:', formValue.contacto.email);
      console.log('Newsletter:', formValue.preferencias.newsletter);
    }
  }

  // Métodos de utilidad con tipos seguros
  resetearDatosPersonales() {
    this.registroForm.controls.datosPersonales.reset();
  }

  actualizarIdioma(nuevoIdioma: string) {
    this.registroForm.controls.preferencias.controls.idioma.setValue(nuevoIdioma);
  }
}
```

### Template con binding tipado

```html
<form [formGroup]="registroForm" (ngSubmit)="onSubmit()">
  <!-- Datos Personales -->
  <fieldset formGroupName="datosPersonales">
    <legend>Datos Personales</legend>
    
    <label>
      Nombre:
      <input type="text" formControlName="nombre">
      @if (nombreControl.invalid && nombreControl.touched) {
        <span class="error">El nombre es requerido (mín. 2 caracteres)</span>
      }
    </label>

    <label>
      Apellidos:
      <input type="text" formControlName="apellidos">
    </label>

    <label>
      Fecha de Nacimiento:
      <input type="date" formControlName="fechaNacimiento">
    </label>
  </fieldset>

  <!-- Contacto -->
  <fieldset formGroupName="contacto">
    <legend>Información de Contacto</legend>
    
    <label>
      Email:
      <input type="email" formControlName="email">
      @if (emailControl.invalid && emailControl.touched) {
        <span class="error">Email requerido y válido</span>
      }
    </label>

    <label>
      Teléfono:
      <input type="tel" formControlName="telefono">
    </label>
  </fieldset>

  <!-- Preferencias -->
  <fieldset formGroupName="preferencias">
    <legend>Preferencias</legend>
    
    <label>
      <input type="checkbox" formControlName="newsletter">
      Suscribirse al newsletter
    </label>

    <label>
      Idioma:
      <select formControlName="idioma">
        <option value="es">Español</option>
        <option value="en">English</option>
        <option value="fr">Français</option>
      </select>
    </label>
  </fieldset>

  <button type="submit" [disabled]="registroForm.invalid">
    Registrarse
  </button>
</form>
```

### Ventajas adicionales en aplicaciones complejas

#### 1. **Mantenimiento a largo plazo**

En equipos grandes o proyectos de larga duración, los Typed Forms proporcionan una documentación viva del código:

```ts
// La interfaz documenta exactamente qué espera el formulario
interface ConfiguracionAvanzada {
  servidor: {
    host: string;
    puerto: number;
    ssl: boolean;
  };
  autenticacion: {
    tipo: 'oauth' | 'basic' | 'token';
    credenciales: string;
  };
  configuracion: {
    timeout: number;
    reintentos: number;
    debug: boolean;
  };
}
```

#### 2. **Integración con APIs tipadas**

Los Typed Forms se integran perfectamente con servicios HTTP tipados:

```ts
interface UsuarioAPI {
  id?: number;
  nombre: string;
  email: string;
  activo: boolean;
}

// El servicio y el formulario comparten el mismo tipo
class UsuarioService {
  actualizar(usuario: UsuarioAPI): Observable<UsuarioAPI> {
    return this.http.put<UsuarioAPI>(`/api/usuarios/${usuario.id}`, usuario);
  }
}

// En el componente
onSubmit() {
  if (this.usuarioForm.valid) {
    const usuario: UsuarioAPI = this.usuarioForm.value;
    this.usuarioService.actualizar(usuario).subscribe(/* ... */);
  }
}
```

#### 3. **Validaciones personalizadas tipadas**

```ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Validador personalizado con tipos
function validarEdadMinima(edadMinima: number): ValidatorFn {
  return (control: AbstractControl<number>): ValidationErrors | null => {
    const edad = control.value;
    if (edad && edad < edadMinima) {
      return { edadMinima: { actual: edad, requerida: edadMinima } };
    }
    return null;
  };
}

// Uso en el formulario
const usuarioForm = this.fb.group<UsuarioForm>({
  nombre: ['', Validators.required],
  edad: [0, [Validators.required, validarEdadMinima(18)]],
  email: ['', [Validators.required, Validators.email]]
});
```

### Migración de formularios existentes

Si tienes formularios legacy, la migración es gradual y directa:

```ts
// ANTES: Formulario sin tipos
const formularioLegacy = this.fb.group({
  campo1: [''],
  campo2: [0],
  campo3: [false]
});

// DESPUÉS: Agregar tipado gradualmente
interface MiFormulario {
  campo1: string;
  campo2: number;
  campo3: boolean;
}

const formularioTipado = this.fb.group<MiFormulario>({
  campo1: [''],
  campo2: [0],
  campo3: [false]
});
```

Los Typed Forms en Angular 20 representan un salto cualitativo en la robustez y mantenibilidad de las aplicaciones. Proporcionan todos los beneficios de TypeScript aplicados específicamente al dominio de los formularios, reduciendo errores, mejorando la experiencia de desarrollo y facilitando el mantenimiento a largo plazo.

## 5.2 Creación de formularios híbridos (Typed + Signals)

Los **formularios híbridos** combinan lo mejor de dos mundos: la **seguridad de tipos** de los Typed Forms y la **reactividad granular** de los Signals. Esta aproximación permite crear formularios que no solo son type-safe, sino que también ofrecen una experiencia de usuario superior gracias a la capacidad de reaccionar automáticamente a cambios de estado de manera eficiente.

### ¿Por qué combinar Typed Forms con Signals?

Aunque los Typed Forms proporcionan seguridad de tipos, tradicionalmente han dependido de RxJS observables para la reactividad. Los Signals en Angular 20 ofrecen varias ventajas importantes:

#### Ventajas de la combinación:

1. **Reactividad granular**: Los Signals actualizan solo los elementos específicos que dependen del valor que cambió
2. **Mejor rendimiento**: Evita re-renderizados innecesarios comparado con observables
3. **Sintaxis más simple**: Menos código boilerplate que con RxJS
4. **Integración nativa**: Los Signals están integrados directamente en el sistema de Change Detection de Angular 20
5. **Debugging más fácil**: Trazabilidad clara de dependencias y cambios

### Arquitectura de un formulario híbrido

Un formulario híbrido típico tiene esta estructura:

```ts
interface UsuarioFormModel {
  nombre: string;
  email: string;
  edad: number;
  activo: boolean;
}

@Component({
  selector: 'app-usuario-form',
  template: `...`
})
export class UsuarioFormComponent {
  private fb = inject(FormBuilder);
  
  // 1. Formulario tipado tradicional
  usuarioForm = this.fb.group<UsuarioFormModel>({
    nombre: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    edad: [18, [Validators.required, Validators.min(18)]],
    activo: [true]
  });

  // 2. Signals derivados del formulario
  formValue = signal(this.usuarioForm.value);
  isFormValid = signal(this.usuarioForm.valid);
  formErrors = signal(this.getFormErrors());

  // 3. Signals computados para UI logic
  puedeEnviar = computed(() => 
    this.isFormValid() && !this.enviandoDatos()
  );
  
  mensajeEstado = computed(() => {
    if (!this.isFormValid()) return 'Complete todos los campos requeridos';
    if (this.enviandoDatos()) return 'Enviando datos...';
    return 'Formulario listo para enviar';
  });

  enviandoDatos = signal(false);
}
```

### Implementación paso a paso

#### Paso 1: Configuración base del componente

```ts
import { Component, Signal, computed, effect, inject, signal } from '@angular/core';
import { FormBuilder, FormGroup, Validators, AbstractControl } from '@angular/forms';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

interface ProductoFormModel {
  nombre: string;
  precio: number;
  categoria: string;
  descripcion: string;
  disponible: boolean;
  fechaLanzamiento: Date;
}

interface FormValidationState {
  isValid: boolean;
  errors: Record<string, string[]>;
  touchedFields: string[];
}

@Component({
  selector: 'app-producto-hibrido',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  template: `...` // veremos el template después
})
export class ProductoHibridoComponent {
  private fb = inject(FormBuilder);
  
  // Formulario tipado base
  productoForm = this.fb.group<ProductoFormModel>({
    nombre: ['', [Validators.required, Validators.minLength(3)]],
    precio: [0, [Validators.required, Validators.min(0.01)]],
    categoria: ['', Validators.required],
    descripcion: [''],
    disponible: [true],
    fechaLanzamiento: [new Date(), Validators.required]
  });
}
```

#### Paso 2: Creación de Signals reactivos

```ts
export class ProductoHibridoComponent {
  // ... código anterior
  
  // Signals para el estado del formulario
  formData = signal<Partial<ProductoFormModel>>({});
  validationState = signal<FormValidationState>({
    isValid: false,
    errors: {},
    touchedFields: []
  });
  
  // Signals para UI state
  isSubmitting = signal(false);
  submitSuccess = signal(false);
  categorias = signal(['Electrónicos', 'Ropa', 'Hogar', 'Deportes']);
  
  // Signals computados para lógica de UI
  precioFormateado = computed(() => {
    const precio = this.formData().precio;
    return precio ? `$${precio.toFixed(2)}` : '$0.00';
  });
  
  puedeGuardar = computed(() => 
    this.validationState().isValid && 
    !this.isSubmitting() && 
    this.hasChanges()
  );
  
  mensajeValidacion = computed(() => {
    const state = this.validationState();
    if (state.isValid) return 'Formulario válido ✓';
    
    const errorCount = Object.keys(state.errors).length;
    return `${errorCount} error(es) encontrado(s)`;
  });
  
  private hasChanges = computed(() => {
    const currentData = this.formData();
    const originalData = this.datosOriginales();
    return JSON.stringify(currentData) !== JSON.stringify(originalData);
  });
  
  datosOriginales = signal<ProductoFormModel | null>(null);
}
```

#### Paso 3: Sincronización bidireccional

```ts
export class ProductoHibridoComponent {
  // ... código anterior
  
  constructor() {
    // Sincronizar cambios del formulario hacia los signals
    this.productoForm.valueChanges
      .pipe(takeUntilDestroyed())
      .subscribe(value => {
        this.formData.set(value as Partial<ProductoFormModel>);
        this.updateValidationState();
      });
    
    // Sincronizar cambios de validación
    this.productoForm.statusChanges
      .pipe(takeUntilDestroyed())
      .subscribe(() => {
        this.updateValidationState();
      });
    
    // Effect para reaccionar a cambios en los datos
    effect(() => {
      const data = this.formData();
      console.log('Datos del formulario actualizados:', data);
      
      // Auto-save cuando hay cambios válidos
      if (this.puedeGuardar() && this.hasChanges()) {
        this.autoSave();
      }
    });
    
    // Effect para logging de cambios de validación
    effect(() => {
      const state = this.validationState();
      if (!state.isValid) {
        console.warn('Errores de validación:', state.errors);
      }
    });
  }
  
  private updateValidationState(): void {
    const errors: Record<string, string[]> = {};
    const touchedFields: string[] = [];
    
    Object.keys(this.productoForm.controls).forEach(key => {
      const control = this.productoForm.get(key);
      if (control) {
        if (control.touched) {
          touchedFields.push(key);
        }
        
        if (control.invalid && control.touched) {
          errors[key] = this.getControlErrors(control);
        }
      }
    });
    
    this.validationState.set({
      isValid: this.productoForm.valid,
      errors,
      touchedFields
    });
  }
  
  private getControlErrors(control: AbstractControl): string[] {
    const errors: string[] = [];
    
    if (control.errors) {
      if (control.errors['required']) errors.push('Campo requerido');
      if (control.errors['minlength']) {
        errors.push(`Mínimo ${control.errors['minlength'].requiredLength} caracteres`);
      }
      if (control.errors['min']) {
        errors.push(`Valor mínimo: ${control.errors['min'].min}`);
      }
      if (control.errors['email']) errors.push('Email inválido');
    }
    
    return errors;
  }
}
```

#### Paso 4: Métodos de interacción

```ts
export class ProductoHibridoComponent {
  // ... código anterior
  
  // Métodos públicos que modifican signals y formulario
  async onSubmit(): Promise<void> {
    if (!this.puedeGuardar()) return;
    
    this.isSubmitting.set(true);
    
    try {
      const formData = this.formData() as ProductoFormModel;
      await this.guardarProducto(formData);
      
      this.submitSuccess.set(true);
      this.datosOriginales.set(formData);
      
      // Reset success message after 3 seconds
      setTimeout(() => this.submitSuccess.set(false), 3000);
      
    } catch (error) {
      console.error('Error al guardar:', error);
    } finally {
      this.isSubmitting.set(false);
    }
  }
  
  resetForm(): void {
    this.productoForm.reset();
    this.isSubmitting.set(false);
    this.submitSuccess.set(false);
    this.datosOriginales.set(null);
  }
  
  cargarDatos(producto: ProductoFormModel): void {
    this.productoForm.patchValue(producto);
    this.datosOriginales.set(producto);
  }
  
  // Signal-driven conditional actions
  agregarCategoria(nuevaCategoria: string): void {
    const categoriasActuales = this.categorias();
    if (!categoriasActuales.includes(nuevaCategoria)) {
      this.categorias.set([...categoriasActuales, nuevaCategoria]);
      
      // Auto-seleccionar la nueva categoría
      this.productoForm.controls.categoria.setValue(nuevaCategoria);
    }
  }
  
  private autoSave(): void {
    // Implementar auto-guardado
    console.log('Auto-guardando datos...', this.formData());
  }
  
  private async guardarProducto(data: ProductoFormModel): Promise<void> {
    // Simular llamada API
    return new Promise(resolve => setTimeout(resolve, 2000));
  }
}
```

### Template reactivo con Signals

```html
<form [formGroup]="productoForm" (ngSubmit)="onSubmit()" class="producto-form">
  <h2>Formulario de Producto Híbrido</h2>
  
  <!-- Indicador de estado general -->
  <div class="form-status" [class.valid]="validationState().isValid">
    {{ mensajeValidacion() }}
    @if (submitSuccess()) {
      <span class="success">¡Guardado exitosamente!</span>
    }
  </div>
  
  <!-- Campo nombre con validación reactiva -->
  <div class="form-field">
    <label for="nombre">Nombre del Producto</label>
    <input 
      id="nombre"
      type="text" 
      formControlName="nombre"
      [class.error]="validationState().errors['nombre']">
    
    @if (validationState().errors['nombre']) {
      <div class="error-messages">
        @for (error of validationState().errors['nombre']; track error) {
          <span class="error">{{ error }}</span>
        }
      </div>
    }
  </div>
  
  <!-- Campo precio con formato reactivo -->
  <div class="form-field">
    <label for="precio">Precio</label>
    <input 
      id="precio"
      type="number" 
      step="0.01"
      formControlName="precio"
      [class.error]="validationState().errors['precio']">
    
    <div class="precio-preview">
      Precio formateado: {{ precioFormateado() }}
    </div>
    
    @if (validationState().errors['precio']) {
      <div class="error-messages">
        @for (error of validationState().errors['precio']; track error) {
          <span class="error">{{ error }}</span>
        }
      </div>
    }
  </div>
  
  <!-- Select de categoría con signal reactivo -->
  <div class="form-field">
    <label for="categoria">Categoría</label>
    <select id="categoria" formControlName="categoria">
      <option value="">Seleccione una categoría</option>
      @for (categoria of categorias(); track categoria) {
        <option [value]="categoria">{{ categoria }}</option>
      }
    </select>
    
    <button 
      type="button" 
      (click)="agregarCategoria('Nueva Categoría')"
      class="btn-secondary">
      Agregar Categoría
    </button>
  </div>
  
  <!-- Campo descripción -->
  <div class="form-field">
    <label for="descripcion">Descripción</label>
    <textarea 
      id="descripcion"
      formControlName="descripcion"
      rows="4">
    </textarea>
  </div>
  
  <!-- Checkbox disponible -->
  <div class="form-field checkbox-field">
    <label>
      <input 
        type="checkbox" 
        formControlName="disponible">
      Producto disponible
    </label>
  </div>
  
  <!-- Campo fecha -->
  <div class="form-field">
    <label for="fechaLanzamiento">Fecha de Lanzamiento</label>
    <input 
      id="fechaLanzamiento"
      type="date" 
      formControlName="fechaLanzamiento">
  </div>
  
  <!-- Botones de acción con estados reactivos -->
  <div class="form-actions">
    <button 
      type="submit"
      [disabled]="!puedeGuardar()"
      [class.loading]="isSubmitting()">
      
      @if (isSubmitting()) {
        Guardando...
      } @else {
        Guardar Producto
      }
    </button>
    
    <button 
      type="button"
      (click)="resetForm()"
      [disabled]="isSubmitting()">
      Limpiar Formulario
    </button>
  </div>
  
  <!-- Debug panel (solo en desarrollo) -->
  <details class="debug-panel">
    <summary>Estado del Formulario (Debug)</summary>
    <pre>{{ formData() | json }}</pre>
    <pre>{{ validationState() | json }}</pre>
  </details>
</form>
```

### Ventajas de los formularios híbridos

1. **Rendimiento superior**: Los Signals actualizan solo las partes necesarias de la UI
2. **Código más limpio**: Menos subscripciones manuales y gestión de observables
3. **Mejor debugging**: Trazabilidad clara de qué signal provocó qué cambio
4. **Reactividad granular**: Actualizaciones específicas sin re-renderizar todo el formulario
5. **Integración natural**: Los Signals funcionan perfectamente con el Change Detection de Angular 20

Los formularios híbridos representan la evolución natural de los formularios en Angular 20, combinando la robustez de los tipos con la eficiencia de los Signals para crear experiencias de usuario fluidas y código mantenible.

## 5.3 Validaciones síncronas y asíncronas aplicadas con Signals


Las **validaciones** son un aspecto fundamental en cualquier formulario avanzado. Angular 20 permite combinar la potencia de los **Typed Forms** con la reactividad de los **Signals** para implementar validaciones tanto síncronas como asíncronas de forma eficiente, declarativa y reactiva.

### ¿Por qué usar Signals para validaciones?

1. **Reactividad instantánea**: Los errores y estados de validación se actualizan automáticamente en la UI sin necesidad de suscripciones manuales.
2. **Menos boilerplate**: Se reduce la necesidad de gestionar manualmente el estado de los errores.
3. **Composición sencilla**: Es fácil combinar validaciones síncronas y asíncronas en un mismo flujo reactivo.
4. **Feedback inmediato**: El usuario recibe retroalimentación en tiempo real, mejorando la experiencia de usuario.

---

### Ejemplo completo: Validaciones síncronas y asíncronas con Signals

Supongamos un formulario de registro de usuario con los siguientes requisitos:
- Validación síncrona: el nombre es obligatorio y debe tener al menos 3 caracteres.
- Validación asíncrona: el email debe ser único (verificado contra una API).

```ts
import { Component, computed, effect, inject, signal } from '@angular/core';
import { FormBuilder, Validators, AsyncValidatorFn, AbstractControl, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { delay, map } from 'rxjs/operators';

// Simulación de servicio de validación asíncrona
function emailUnicoValidator(): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    const email = control.value;
    // Simula una llamada HTTP
    return of(email !== 'ya@existe.com').pipe(
      delay(1000),
      map(isUnique => (isUnique ? null : { emailNoUnico: true }))
    );
  };
}

interface RegistroForm {
  nombre: string;
  email: string;
}

@Component({
  selector: 'app-registro-signals',
  template: `...` // Veremos el template después
})
export class RegistroSignalsComponent {
  private fb = inject(FormBuilder);

  registroForm = this.fb.group<RegistroForm>({
    nombre: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
  }, { asyncValidators: [emailUnicoValidator()] });

  // Signals sincronizados con los valores y estados de los controles
  nombreValue = signal<string>('');
  emailValue = signal<string>('');
  nombreErrors = signal<string[]>([]);
  emailErrors = signal<string[]>([]);
  emailChecking = signal(false);

  constructor() {
    // Sincronizar signals con los valores del formulario
    this.registroForm.controls.nombre.valueChanges.subscribe(value => this.nombreValue.set(value ?? ''));
    this.registroForm.controls.email.valueChanges.subscribe(value => this.emailValue.set(value ?? ''));

    // Effect para validación síncrona del nombre
    effect(() => {
      const control = this.registroForm.controls.nombre;
      this.nombreValue(); // Dependencia explícita para reactividad
      const errors = [];
      if (control.errors) {
        if (control.errors['required']) errors.push('El nombre es obligatorio');
        if (control.errors['minlength']) errors.push('Mínimo 3 caracteres');
      }
      this.nombreErrors.set(errors);
    });

    // Effect para validación asíncrona del email
    effect(() => {
      const control = this.registroForm.controls.email;
      this.emailValue(); // Dependencia explícita para reactividad
      const errors = [];
      if (control.errors) {
        if (control.errors['required']) errors.push('El email es obligatorio');
        if (control.errors['email']) errors.push('Formato de email inválido');
        if (control.errors['emailNoUnico']) errors.push('El email ya está registrado');
      }
      this.emailErrors.set(errors);
      this.emailChecking.set(control.pending);
    });
  }
}
```

### Template reactivo con feedback inmediato

```html
<form [formGroup]="registroForm" (ngSubmit)="onSubmit()">
  <div>
    <label>Nombre:</label>
    <input type="text" formControlName="nombre">
    <span *ngFor="let error of nombreErrors()" class="error">{{ error }}</span>
  </div>
  <div>
    <label>Email:</label>
    <input type="email" formControlName="email">
    <span *ngFor="let error of emailErrors()" class="error">{{ error }}</span>
    <span *ngIf="emailChecking()">Verificando email...</span>
  </div>
  <button type="submit" [disabled]="!registroForm.valid || emailChecking()">Registrar</button>
</form>
```

---

### Patrones avanzados de validación con Signals

#### 1. Validación de campos dependientes

```ts
@Component({
  selector: 'app-password-signals',
  template: `...`
})
export class PasswordSignalsComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    password: ['', [Validators.required, Validators.minLength(8)]],
    confirmPassword: ['', Validators.required]
  });

  // Signals derivados de los valores de los controles
  password = signal<string>('');
  confirmPassword = signal<string>('');
  passwordMatchError = signal<string | null>(null);

  constructor() {
    // Sincronizar los signals con los valores del formulario
    this.form.controls.password.valueChanges.subscribe(value => this.password.set(value ?? ''));
    this.form.controls.confirmPassword.valueChanges.subscribe(value => this.confirmPassword.set(value ?? ''));

    effect(() => {
      const pass = this.password();
      const confirm = this.confirmPassword();
      this.passwordMatchError.set(
        pass && confirm && pass !== confirm ? 'Las contraseñas no coinciden' : null
      );
    });
  }
}
```

#### 2. Validación asíncrona con debounce y cancelación

```ts
import { debounceTime, switchMap } from 'rxjs/operators';

@Component({
  selector: 'app-usuario-signals',
  template: `...`
})
export class UsuarioSignalsComponent {
  private fb = inject(FormBuilder);
  usuarioForm = this.fb.group({
    username: ['', Validators.required]
  });
  usernameValue = signal<string>('');
  usernameAvailable = signal<boolean | null>(null);
  checking = signal(false);

  constructor() {
    // Sincronizar el signal con el valor del control
    this.usuarioForm.controls.username.valueChanges.subscribe(value => this.usernameValue.set(value ?? ''));

    effect(() => {
      const username = this.usernameValue();
      if (username && username.length > 2) {
        this.checking.set(true);
        // Simula llamada API con debounce
        of(username).pipe(
          debounceTime(500),
          switchMap(name => this.verificarUsername(name))
        ).subscribe(isAvailable => {
          this.usernameAvailable.set(isAvailable);
          this.checking.set(false);
        });
      } else {
        this.usernameAvailable.set(null);
      }
    });
  }

  private verificarUsername(username: string): Observable<boolean> {
    // Simula API
    return of(username !== 'taken').pipe(delay(800));
  }
}
```

---

### Buenas prácticas para validación con Signals

- Usa **Signals** para exponer el estado de validación y errores en la UI de forma reactiva.
- Combina validaciones síncronas y asíncronas para una experiencia de usuario óptima.
- Utiliza **effects** para actualizar errores y estados en tiempo real.
- Implementa debounce y cancelación en validaciones asíncronas para evitar llamadas innecesarias.
- Mantén la lógica de validación separada y reutilizable.



## 5.4 Personalización de mensajes de error reactivos y dinámicos con Signals

Una de las grandes ventajas de Angular 20 es que, gracias a los **signals**, la personalización de mensajes de error se vuelve completamente reactiva y dinámica. Los signals permiten que los mensajes de error respondan automáticamente a cambios en el idioma, el contexto de la aplicación o los datos del formulario, sin necesidad de lógica adicional ni suscripciones manuales.

### ¿Por qué usar signals para personalizar mensajes de error?

- **Reactividad total**: Cambia el idioma, el contexto o la lógica de mensajes y los errores se actualizan automáticamente en la UI.
- **Centralización**: Toda la lógica de mensajes puede estar en signals o funciones computadas, facilitando el mantenimiento.
- **Experiencia de usuario superior**: El feedback es inmediato y siempre contextualizado.

---


### Ejemplo: Mapeo dinámico de errores con Signals

Un ejemplo sencillo de personalización de mensajes de error usando signals:

```ts
import { Component, inject, signal } from '@angular/core';
import { FormBuilder, Validators, AbstractControl } from '@angular/forms';

@Component({
  selector: 'app-registro-errores',
  template: `...` // Veremos el template después
})
export class RegistroErroresComponent {
  private fb = inject(FormBuilder);

  registroForm = this.fb.group({
    nombre: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
  });

  // Diccionario simple de mensajes
  mensajesError: Record<string, string> = {
    required: 'Este campo es obligatorio',
    minlength: 'Debe tener al menos 3 caracteres',
    email: 'Introduce un email válido',
  };

  // Signals para los mensajes de error
  nombreError = signal('');
  emailError = signal('');

  constructor() {
    this.registroForm.controls.nombre.valueChanges.subscribe(() => this.setNombreError());
    this.registroForm.controls.email.valueChanges.subscribe(() => this.setEmailError());
    this.setNombreError();
    this.setEmailError();
  }

  private setNombreError() {
    this.nombreError.set(this.getPrimerError(this.registroForm.controls.nombre));
  }
  private setEmailError() {
    this.emailError.set(this.getPrimerError(this.registroForm.controls.email));
  }

  private getPrimerError(control: AbstractControl): string {
    if (!control || !control.errors) return '';
    const key = Object.keys(control.errors)[0];
    return this.mensajesError[key] || 'Error desconocido';
  }
}
```


### Template con mensajes de error personalizados y reactivos

```html
<form [formGroup]="registroForm">
  <div>
    <label>Nombre:</label>
    <input type="text" formControlName="nombre">
    @if (nombreError()) {
      <span class="error">{{ nombreError() }}</span>
    }
  </div>
  <div>
    <label>Email:</label>
    <input type="email" formControlName="email">
    @if (emailError()) {
      <span class="error">{{ emailError() }}</span>
    }
  </div>
  <button type="submit" [disabled]="registroForm.invalid">Registrar</button>
</form>
```

---



### Patrones avanzados de personalización con signals

#### 1. Mensajes de error multilenguaje reactivos

Define un signal para el idioma y un signal computado para los mensajes:

```ts
idioma = signal<'es' | 'en'>('es');

mensajesError = computed(() =>
  idioma() === 'es'
    ? { required: 'Campo obligatorio', email: 'Email inválido', minlength: 'Debe tener al menos 3 caracteres' }
    : { required: 'Required field', email: 'Invalid email', minlength: 'Must be at least 3 characters' }
);

getPrimerError(control: AbstractControl): string {
  if (!control || !control.errors) return '';
  const key = Object.keys(control.errors)[0];
  return this.mensajesError()[key] || 'Error';
}
```

Al cambiar el valor de `idioma`, todos los mensajes de error se actualizan automáticamente en la UI.

#### 2. Mensajes contextuales según el flujo (reactivo)

Puedes usar un signal para el contexto y un signal computado para los mensajes:

```ts
contexto = signal<'registro' | 'edicion'>('registro');

mensajesError = computed(() => ({
  required: contexto() === 'registro' ? 'Debes rellenar este campo' : 'Campo requerido',
  email: 'Introduce un email válido',
  minlength: 'Debe tener al menos 3 caracteres',
}));

getPrimerError(control: AbstractControl): string {
  if (!control || !control.errors) return '';
  const key = Object.keys(control.errors)[0];
  return this.mensajesError()[key] || 'Error';
}
```

#### 3. Mensajes enriquecidos con datos dinámicos (reactivo)

Puedes usar un signal computado para generar mensajes personalizados según el valor del control:

```ts
mensajesError = computed(() => ({
  required: 'Este campo es obligatorio',
  minlength: (control: AbstractControl) => {
    const e = control.errors?.['minlength'];
    if (!e) return '';
    const restantes = e.requiredLength - (control.value?.length || 0);
    return `Te faltan ${restantes} caracteres`;
  },
  email: 'Introduce un email válido',
}));

getPrimerError(control: AbstractControl): string {
  if (!control || !control.errors) return '';
  const key = Object.keys(control.errors)[0];
  const msg = this.mensajesError()[key];
  return typeof msg === 'function' ? msg(control) : msg || 'Error';
}
```

Así, cualquier cambio en idioma, contexto o datos del formulario se refleja automáticamente en los mensajes de error gracias a los signals.

---


- Centraliza la lógica de mensajes de error en funciones reutilizables.
- Usa signals para exponer los mensajes de error de forma reactiva.
- Personaliza los mensajes según idioma, contexto y datos dinámicos.
- Mejora la experiencia de usuario con feedback claro y contextualizado.

La personalización reactiva de mensajes de error en Angular 20 permite construir formularios mucho más amigables, accesibles y profesionales.

## 5.5 Integración con RxJS: validaciones y sincronización con Observables

Aunque Angular 20 promueve el uso de Signals para la reactividad, RxJS sigue siendo fundamental para manejar flujos de datos asíncronos complejos, validaciones remotas, streams de eventos y sincronización con APIs externas. La integración entre Signals y Observables permite aprovechar lo mejor de ambos mundos.

### ¿Cuándo usar RxJS junto a Signals en formularios?

- Cuando necesitas consumir datos de APIs o servicios que exponen Observables.
- Para validaciones asíncronas avanzadas (por ejemplo, debounce, cancelación, composición de streams).
- Para sincronizar el estado del formulario con fuentes externas en tiempo real.

---

### Ejemplo: Validación asíncrona con debounce y cancelación usando RxJS y Signals

Supongamos un formulario de registro donde el nombre de usuario debe ser único y la validación debe hacerse con debounce y cancelación de peticiones previas.

```ts
import { Component, inject, signal, effect } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';
import { of, Subject } from 'rxjs';
import { debounceTime, switchMap, takeUntil, delay } from 'rxjs/operators';

@Component({
  selector: 'app-usuario-rxjs',
  template: `...` // Veremos el template después
})
export class UsuarioRxjsComponent {
  private fb = inject(FormBuilder);
  usuarioForm = this.fb.group({
    username: ['', Validators.required]
  });

  usernameValue = signal('');
  usernameAvailable = signal<boolean | null>(null);
  checking = signal(false);

  // Subject para cancelar peticiones previas
  private destroy$ = new Subject<void>();

  constructor() {
    // Sincronizar el signal con el valor del control
    this.usuarioForm.controls.username.valueChanges.subscribe(value => this.usernameValue.set(value ?? ''));

    effect(() => {
      const username = this.usernameValue();
      this.destroy$.next(); // Cancela peticiones previas
      if (username && username.length > 2) {
        this.checking.set(true);
        of(username).pipe(
          debounceTime(500),
          switchMap(name => this.verificarUsername(name)),
          takeUntil(this.destroy$)
        ).subscribe(isAvailable => {
          this.usernameAvailable.set(isAvailable);
          this.checking.set(false);
        });
      } else {
        this.usernameAvailable.set(null);
      }
    });
  }

  private verificarUsername(username: string) {
    // Simula llamada API
    return of(username !== 'taken').pipe(delay(800));
  }
}
```

### Template reactivo con feedback de RxJS y Signals

```html
<form [formGroup]="usuarioForm">
  <label>Usuario:</label>
  <input type="text" formControlName="username">
  @if (checking()) {
    <span class="info">Comprobando disponibilidad...</span>
  }
  @if (usernameAvailable() === false) {
    <span class="error">El usuario ya está en uso</span>
  }
  @if (usernameAvailable() === true) {
    <span class="success">¡Usuario disponible!</span>
  }
  <button type="submit" [disabled]="!usuarioForm.valid || checking()">Registrar</button>
</form>
```

---

### Sincronización de formularios con streams externos

Puedes sincronizar el valor del formulario con un Observable externo (por ejemplo, datos en tiempo real de un WebSocket):

```ts
import { Observable } from 'rxjs';

@Component({
  selector: 'app-sincronizacion-stream',
  template: `...`
})
export class SincronizacionStreamComponent {
  private fb = inject(FormBuilder);
  perfilForm = this.fb.group({
    nombre: [''],
    email: ['']
  });

  // Simula un stream externo (por ejemplo, WebSocket)
  datosPerfil$: Observable<{ nombre: string; email: string }>; // Inyectado o creado

  constructor() {
    // Suscribirse y actualizar el formulario automáticamente
    this.datosPerfil$.subscribe(datos => {
      this.perfilForm.patchValue(datos);
    });
  }
}
```

---

### Conversión entre Signals y Observables

Angular proporciona utilidades para convertir entre signals y observables:

- **fromObservable**: Convierte un Observable en un signal reactivo.
- **toObservable**: Convierte un signal en un Observable para integrarlo con RxJS.

```ts
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

// Observable a signal
const datosSignal = toSignal(datosPerfil$);

// Signal a observable
const username$ = toObservable(usernameValue);
```

