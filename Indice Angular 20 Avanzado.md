# Temario del curso Angular 20 Avanzado

## [1. Repaso de las novedades y fundamentos avanzados incorporados en Angular 20](Tema%2001.md)

- Standalone Components y el fin progresivo de NgModules  
- Nueva sintaxis de control flow (`@if`, `@for`, `@switch`)  
- Reactividad moderna con Signals (`signal`, `computed`, `effect`)  
- Integración de `linkedSignal`, `toSignal` y `resource`  
- Zoneless Angular: detección de cambios sin Zone.js  
- Formularios con Signals (estado de preview en Angular 20)  
- SSR con Hydration incremental estable  
- Angular CLI: recarga en caliente y diagnósticos mejorados  
- DevTools integrados y profiling avanzado en Angular 20  
- Deprecaciones clave desde Angular 17 y su impacto en proyectos actuales  

---

## [2. Angular Signals en profundidad y su interoperabilidad con RxJS](Tema%2002.md)

- Creación y uso de Signals en Angular 20  
- Signals computados (`computed`) y buenas prácticas de cálculo derivado  
- Gestión de efectos reactivos con `effect()`  
- Uso avanzado de `linkedSignal` y `resource` para orquestar flujos de datos  
- Interoperabilidad entre Signals y Observables (`toSignal`, `toObservable`)  
- Patrones de migración desde `Subject` y `BehaviorSubject` hacia Signals  
- Integración de Angular Signals con el cliente HTTP (`httpResource()`)  
- Uso de operadores RxJS aplicados a Signals y viceversa  
- Implementación de patrones arquitectónicos con Signals (state management)  
- Anti-patrones comunes y cómo evitarlos en aplicaciones reales  

---

## [3. Change Detection avanzado en Angular 20](Tema%2003.md)

- Conceptos fundamentales del Change Detection en Angular  
- Zoneless Angular: funcionamiento interno sin Zone.js  
- Estrategias `OnPush` en coexistencia con Signals  
- Métodos clásicos (`markForCheck`, `detach`, `detectChanges`) en modo legacy  
- Uso de `effect()` y `computed()` como mecanismo moderno de gestión de cambios  
- Estrategias para minimizar recomputaciones innecesarias  
- Casos prácticos en los que deshabilitar la propagación del árbol es útil  
- Identificación de operaciones costosas y optimización del renderizado  
- Integración con Angular DevTools para profiling y debugging de CD  
- Buenas prácticas de monitorización y diagnóstico en aplicaciones enterprise  

---

## [4. Directivas y control flow moderno](Tema%2004.md)

- Introducción al nuevo control flow con `@if`, `@for` y `@switch`  
- Diferencias y ventajas respecto a `*ngIf` y `*ngFor` tradicionales    
- Directivas de atributos reactivas integradas con Signals  
- Host Directives: reutilización y encapsulación de lógica transversal  
- Manipulación del DOM de forma reactiva y segura en Angular 20  
- Integración de eventos con Signals en directivas personalizadas  
- Compatibilidad con proyectos legacy: coexistencia con `*ngIf` y `*ngFor`  
- Casos prácticos de directivas avanzadas en entornos enterprise

---

## [5. Formularios avanzados en Angular 20](Tema%2005.md)

- Formularios fuertemente tipados (Typed Forms) y sus ventajas  
- Creación de formularios híbridos (Typed + Signals)  
- Validaciones síncronas y asíncronas aplicadas con Signals  
- Personalización de mensajes de error reactivos y dinámicos  
- Integración con RxJS: validaciones y sincronización con Observables  
- Ejemplos prácticos de formularios enterprise con lógica avanzada  
- Buenas prácticas y estrategias de migración para equipos grandes  

---

## 6. [Inyección de dependencias y Standalone APIs](Tema%2006.md)

- Introducción a los Standalone APIs y la inyección moderna en Angular 20  
- Providers funcionales vs providers clásicos: diferencias y casos de uso  
- Jerarquía avanzada de inyección y modificadores (`@Optional`, `@Self`, `@SkipSelf`)  
- Uso de `InjectionToken` para inyecciones condicionales y seguras  
- Array Providers y View Providers en escenarios complejos  
- Functional Providers y su integración con Standalone Components  
- Casos prácticos de DI en aplicaciones enterprise distribuidas  
- Estrategias de troubleshooting en jerarquías de inyección profundas  
- Diferencias clave entre DI clásico (NgModules) y DI funcional moderno  
- Buenas prácticas para mantener escalabilidad y testabilidad con DI  

---

## 7. [Routing avanzado en Angular 20](Tema%2007.md)

- Configuración moderna de rutas con `provideRouter` y Standalone Components  
- Lazy loading optimizado con `loadChildren` y rutas dinámicas  
- Estrategias de precarga inteligente y bajo demanda con Signals  
- Precarga condicional en entornos de red lenta o móvil  
- Functional Guards (`canActivate`, `canMatch`) con `inject()`  
- Functional Resolvers para la carga previa de datos  
- Creación y uso de Functional Interceptors en el flujo HTTP  
- Rutas hijas y segmentación modular en aplicaciones grandes  
- Comparativa entre módulos y standalone en el lazy loading  
- Herramientas de análisis y debugging de rendimiento en el enrutamiento  

---

## 8. [Optimización de recursos e imágenes](Tema%2008.md)

- Introducción al nuevo `NgOptimizedImage` en Angular 20  
- Configuración básica y avanzada de `NgOptimizedImage`  
- Comparación con técnicas manuales de optimización de imágenes  
- Carga diferida (lazy loading) y diferimiento condicional de recursos  
- Uso de directivas para retrasar imágenes no visibles en pantalla  
- Mejores prácticas de imágenes responsive con Angular  
- Compresión y uso de formatos modernos (WebP, AVIF)  
- Estrategias de imágenes en SSR y PWAs  
- Integración de `NgOptimizedImage` en proyectos enterprise grandes  
- Ejemplos de migración desde librerías de terceros hacia la solución nativa  

---

## 9. [SSR, Prerender e Hydration en Angular 20](Tema%2009.md)

- Fundamentos de Server Side Rendering en Angular  
- Configuración inicial de SSR con Angular CLI y Standalone Components  
- Prerender híbrido: cuándo conviene y cómo implementarlo  
- Hydration incremental estable en Angular 20  
- Comparación entre hydration parcial e incremental  
- Estrategias para mejorar métricas clave: FCP, TTI y LCP  
- SSR con APIs dinámicas y consideraciones de SEO avanzado  
- Uso de Angular DevTools y Chrome Profiler para SSR  
- Casos de uso en aplicaciones de alto tráfico y escalabilidad global  
- Buenas prácticas de mantenimiento y despliegue de SSR/Hydration  

---

## 10. [Testing moderno en Angular 20](Tema%2010.md)

- Introducción a las estrategias de testing modernas en Angular  
- Unit testing con Jest: configuración y ventajas frente a Karma/Jasmine  
- Harnesses del Angular CDK para pruebas aisladas de componentes  
- Mocking avanzado con `TestBed.inject()` y Signals  
- Testing de formularios basados en Signals y validaciones dinámicas  
- Estrategias de testing en aplicaciones con SSR e Hydration  
- End-to-End testing (e2e) con Cypress: casos de uso y configuración  
- Integración de Jest + Cypress en pipelines de CI/CD  
- Buenas prácticas para asegurar cobertura y fiabilidad en proyectos enterprise  

---

## 11. [Arquitecturas escalables con Angular](Tema%2011.md)

- Introducción a Angular Workspaces para proyectos multi-app  
- Uso de librerías compartidas entre diferentes aplicaciones Angular  
- Creación de Schematics personalizados para automatizar generación de código  
- Estrategias de monorepos y multirepos en entornos enterprise  
- Introducción a Microfrontends y cuándo aplicarlos  
- Estrategias de seguridad, despliegue y monitorización en arquitecturas distribuidas  

---

## 12. [PWAs y APIs web avanzadas](Tema%2012.md)

- Introducción y fundamentos de las Progressive Web Apps (PWAs) en Angular  
- Configuración de Service Workers y estrategias de caché para offline-first  
- Implementación de Web Workers para tareas pesadas y paralelización  
- Registro y comunicación con Web Workers en aplicaciones Angular  
- Estrategias de compatibilidad en navegadores modernos y móviles  
- Uso de Push Notifications API en Angular (registro y envío de notificaciones)  
- Integración de PWAs con Angular CLI: instalación y pruebas en dispositivos  
- Configuración de Web Manifest y mejora de experiencia de usuario  
- Implementación de lógica avanzada en notificaciones push (acciones y respuestas)  
- Buenas prácticas en la construcción de PWAs seguras, accesibles y rápidas  

---

## 13. [Estado global con NGRX en Angular 20](Tema%2013.md)

- Introducción a NGRX y su relación con el patrón Redux  
- Ventajas y desventajas del uso de NGRX frente a Signals o servicios locales  
- Creación y configuración inicial del Store en un proyecto Angular  
- Definición y uso de Actions para describir eventos de la aplicación  
- Implementación de Reducers para gestionar el estado global  
- Uso de Selectors para obtener datos del Store en componentes Angular  
- Implementación de Effects para manejar efectos secundarios y peticiones HTTP  
- Estrategias para combinar NGRX con Signals en proyectos modernos  
- Debugging y monitorización del Store con NgRx DevTools  
- Buenas prácticas en proyectos enterprise: modularización, testing y mantenimiento  

---

## 14. [Internacionalización (i18n) en Angular 20](Tema%2014.md)

- Configuración inicial de internacionalización (i18n) en Angular  
- Generación de archivos de traducción con Angular CLI  
- Uso de etiquetas y marcadores para traducir en plantillas y componentes  
- Cambio dinámico de idioma en una aplicación Angular  
- Estrategias para gestionar múltiples idiomas en proyectos enterprise  
- Implementación de selección de idioma mediante formularios y Signals  
- Integración de i18n con SSR para aplicaciones multilingües optimizadas para SEO  
- Automatización de traducciones con servicios externos (Google Translate API, DeepL)  
- Testing de aplicaciones con soporte multi-idioma  
- Buenas prácticas y recomendaciones para proyectos globales  

---

## 15. [Creación de librerías y documentación avanzada en Angular 20](Tema%2015.md)

- Creación de librerías con Angular CLI y estructura recomendada  
- Configuración de un Angular Workspace para librerías reutilizables  
- Definición de APIs públicas y privadas en librerías Angular  
- Empaquetado y optimización para publicación  
- Publicación de librerías en NPM y registros privados (GitHub Packages, Azure Artifacts)  
- Estrategias de versionado semántico y control de dependencias en librerías  
- Documentación de librerías y proyectos con Compodoc  
- Integración de Storybook para documentar componentes UI  
- Automatización de documentación con IA (Copilot, Cursor, WindsurfAI, ChatGPT)  
- Buenas prácticas para librerías internas (enterprise) vs open source  

---

## 16. [Buenas prácticas y mantenimiento en Angular 20](Tema%2016.md)

- Uso de herramientas de IA (Copilot, Cursor) para generación de código y documentación  
- Seguridad en Angular: protección contra XSS, CSRF y vulnerabilidades comunes  
- Mejores prácticas de autenticación y autorización (JWT, OAuth2)  
- Accesibilidad (a11y): uso de atributos ARIA y pruebas automatizadas con axe-core  
- Estrategias de refactorización segura en aplicaciones grandes  
- Versionado semántico y gestión de dependencias con NPM y Renovate  
- Integración de CI/CD con GitHub Actions y pipelines automatizados  
- Monitorización continua y logging en aplicaciones Angular productivas  
- Checklist de buenas prácticas para proyectos enterprise en Angular 20  

---

## 17. [Migración de Angular 17 a Angular 20](Tema%2017.md)

- Recordatorio de las diferencias clave entre Angular 17 y Angular 20  
- Migración de sintaxis de control flow: de `*ngIf`/`*ngFor` a `@if`/`@for`  
- Sustitución progresiva de NgModules por Standalone Components  
- Reemplazo de guards, resolvers e interceptors basados en clases por APIs funcionales  
- Adaptación de formularios clásicos a Typed Forms y formularios con Signals  
- Uso progresivo de Zoneless Change Detection en lugar de Zone.js  
- Migración hacia `NgOptimizedImage` desde librerías de terceros  
- Ajustes en proyectos con SSR: introducción del Hydration incremental  
- Sustitución de test runners obsoletos por Jest y Cypress  
- Herramientas de apoyo para la migración: Angular CLI, Angular Update Guide y checklists  

---

## 18. Proyecto final

- Selección de un caso de uso avanzado (e.g. aplicación SaaS o e-commerce modular)  
- Diseño e implementación con Standalone Components como base arquitectónica  
- Uso combinado de Signals, RxJS y APIs funcionales en la lógica de negocio  
- Implementación de SSR con Hydration incremental para mejorar rendimiento y SEO  
- Optimización de imágenes y recursos con `NgOptimizedImage`  
- Configuración de un flujo de CI/CD completo con GitHub Actions  
- Integración de testing moderno: Jest (unit) y Cypress (e2e)  
- Análisis de rendimiento con Angular DevTools y Chrome Profiler  
- Documentación asistida con IA (Copilot, Cursor)  
- Estrategias de seguridad, accesibilidad y despliegue en entornos enterprise  