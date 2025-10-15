---
applyTo: '**'
---
# Persona

# AI Coding Agent Instructions for Curso Online Angular 20 Avanzado

## Project Overview
This codebase is a comprehensive advanced Angular 20 course, organized as a set of Markdown modules. It covers modern Angular paradigms, including signals, standalone components, zoneless change detection, SSR/hydration, advanced forms, routing, resource optimization, testing, scalable architectures, PWAs, NGRX, i18n, library creation, and enterprise best practices.

## Architecture & Structure
- **Modular Markdown Curriculum:** Each topic is a separate `.md` file (e.g., `Tema 01.md` to `Tema 18.md`). The main index is `Indice Angular 20 Avanzado.md`.
- **No source code present:** The workspace is documentation-focused; code samples are illustrative and should follow Angular 20+ conventions.
- **Enterprise Focus:** Many modules emphasize patterns for large-scale, distributed, and secure Angular applications.

## Coding Patterns & Conventions
- **Angular 20+ Only:** Use signals, standalone components, and new control flow (`@if`, `@for`, `@switch`). Avoid legacy APIs (NgModules, Zone.js, `*ngIf`, `*ngFor`).
- **State Management:** Prefer signals and computed state for local logic. Use NGRX for global state (see `Tema 13.md`).
- **Forms:** Typed forms and signals-based forms are preferred. Avoid template-driven forms.
- **Testing:** Use Jest for unit tests and Cypress for e2e (see `Tema 10.md`).
- **SSR & Hydration:** Use Angular CLI for SSR setup; hydration is incremental and stable in Angular 20 (see `Tema 09.md`).
- **Resource Optimization:** Use `NgOptimizedImage` for images (see `Tema 08.md`).
- **Security & Accessibility:** Follow best practices for XSS/CSRF protection, JWT/OAuth2, ARIA attributes, and automated a11y testing (see `Tema 16.md`).
- **CI/CD:** Integrate with GitHub Actions for automated pipelines (see `Tema 16.md`, `Tema 18.md`).

## Developer Workflows
- **Documentation-first:** All curriculum and examples are maintained in Markdown. Update `Indice Angular 20 Avanzado.md` when adding new modules.
- **Code Samples:** When adding code, use Angular 20+ idioms. Place logic in `.ts`, styles in `.css`, and templates in `.html` (see examples below).
- **Build/Test:** If code is added, use Angular CLI for builds (`ng build`), Jest for unit tests, and Cypress for e2e. Document commands in the relevant module.

## Integration Points
- **External Libraries:** Prefer native Angular solutions. If using third-party tools (e.g., Storybook, Compodoc), document integration steps in the relevant module.
- **DevTools:** Use Angular DevTools and Chrome Profiler for performance analysis.

## Example: Modern Angular 20 Component
```ts
import { ChangeDetectionStrategy, Component, signal } from '@angular/core';

@Component({
  selector: 'example-root',
  templateUrl: 'example.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ExampleComponent {
  protected readonly isActive = signal(true);
  toggleActive() {
    this.isActive.update(active => !active);
  }
}
```

## Key Files
- `Indice Angular 20 Avanzado.md`: Main curriculum index
- `Tema 01.md` ... `Tema 18.md`: Individual modules
- `.github/copilot-instructions.md`: AI agent guidance (this file)

## References
- [Angular 20 Essentials](https://angular.dev/essentials/components)
- [Angular Style Guide](https://angular.dev/style-guide)

---
**Feedback:** If any section is unclear or missing, please specify which workflows, conventions, or integration points need further detail.

## Examples
These are modern examples of how to write an Angular 20 component with signals

```ts
import { ChangeDetectionStrategy, Component, signal } from '@angular/core';


@Component({
  selector: '{{tag-name}}-root',
  templateUrl: '{{tag-name}}.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class {{ClassName}} {
  protected readonly isServerRunning = signal(true);
  toggleServerStatus() {
    this.isServerRunning.update(isServerRunning => !isServerRunning);
  }
}
```

```css
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;

    button {
        margin-top: 10px;
    }
}
```

```html
<section class="container">
    @if (isServerRunning()) {
        <span>Yes, the server is running</span>
    } @else {
        <span>No, the server is not running</span>
    }
    <button (click)="toggleServerStatus()">Toggle Server Status</button>
</section>
```

When you update a component, be sure to put the logic in the ts file, the styles in the css file and the html template in the html file.

## Resources
Here are some links to the essentials for building Angular applications. Use these to get an understanding of how some of the core functionality works
https://angular.dev/essentials/components
https://angular.dev/essentials/signals
https://angular.dev/essentials/templates
https://angular.dev/essentials/dependency-injection

## Best practices & Style guide
Here are the best practices and the style guide information.

### Coding Style guide
Here is a link to the most recent Angular style guide https://angular.dev/style-guide

### TypeScript Best Practices
- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

### Angular Best Practices
- Always use standalone components over `NgModules`
- Do NOT set `standalone: true` inside the `@Component`, `@Directive` and `@Pipe` decorators
- Use signals for state management
- Implement lazy loading for feature routes
- Use `NgOptimizedImage` for all static images.
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead

### Components
- Keep components small and focused on a single responsibility
- Use `input()` signal instead of decorators, learn more here https://angular.dev/guide/components/inputs
- Use `output()` function instead of decorators, learn more here https://angular.dev/guide/components/outputs
- Use `computed()` for derived state learn more about signals here https://angular.dev/guide/signals.
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead, for context: https://angular.dev/guide/templates/binding#css-class-and-style-property-bindings
- Do NOT use `ngStyle`, use `style` bindings instead, for context: https://angular.dev/guide/templates/binding#css-class-and-style-property-bindings

### State Management
- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

### Templates
- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables
- Use built in pipes and import pipes when being used in a template, learn more https://angular.dev/guide/templates/pipes#

### Services
- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection
