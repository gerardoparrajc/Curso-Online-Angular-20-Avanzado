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

