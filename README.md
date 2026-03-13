#  Tarea 2 – Administración de Sitios Web

> **Tarea No.2 – Curso 03102 | 2026**  
> Implementación de un pipeline CI/CD usando GitHub Actions con despliegue automático en GitHub Pages.

---

##  Descripción del Proyecto

Este repositorio contiene un sitio web estático desarrollado en **HTML5, CSS3 y JavaScript**, que sirve como demostración práctica de los conceptos de **CI/CD (Integración y Despliegue Continuo)** aplicados a la administración de sitios web.

Cada vez que se realiza un `push` a la rama `master`, el pipeline automatiza:
1. **Validación del HTML** con `html5validator`
2. **Despliegue automático** en GitHub Pages
 
---
 
## Estructura del Repositorio
 
```
Tarea-2-DanielGutierrezA-03101/
├── index.html                    # Página principal del sitio
├── css/
│   └── index.css                # Estilos del sitio
├── js/
│   └── index.js                   # Interactividad (scroll, animaciones)
├── .github/
│   └── workflows/
│       └── deploy.yml            # Pipeline de CI/CD con GitHub Actions
└── README.md                     # Este archivo
```
 
---
 
## Configuración del Workflow (paso a paso)
 
### Paso 1 – Crear el repositorio en GitHub
 
1. Inicia sesión en [github.com](https://github.com)
2. Haz clic en **"New repository"**
3. Nombre: `Tarea-2-DanielGutierrezA-03101`
4. Visibilidad: **Public**
5. Haz clic en **"Create repository"**
 
---
 
### Paso 2 – Subir los archivos del sitio
 
```bash
git clone https://github.com/DanielJUNED/Tarea-2-DanielGutierrezA-03101.git
cd Tarea-2-DanielGutierrezA-03101
 
git add .
git commit -m "Commit inicial"
git push origin master
```
 
---
 
### Paso 3 – Configuración del Workflow (`deploy.yml`)
 
El archivo `.github/workflows/deploy.yml` es el corazón del pipeline. Se crea dentro del repositorio con la siguiente ruta y estructura:
 
```
.github/
└── workflows/
    └── deploy.yml
```
 
#### ¿Cuándo se activa?
 
```yaml
on:
  push:
    branches: [master]       # Se dispara al hacer push a master
  pull_request:
    branches: [master]       # También valida Pull Requests antes de mergear
```
 
#### Permisos necesarios para GitHub Pages
 
```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```
 
Estos permisos le dan al workflow acceso de escritura sobre GitHub Pages, lo cual es requerido para el despliegue automático.
 
#### Job 1 – Integración Continua (CI): Validar HTML
 
```yaml
jobs:
  validar-html:
    name: Validar HTML
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # Descarga el código del repo
      - uses: actions/setup-python@v5      # Instala Python
        with:
          python-version: '3.x'
      - run: pip install html5validator    # Instala el validador
      - run: html5validator --root . --also-check-css --log INFO
```
 
Este job corre en una máquina virtual Ubuntu, instala `html5validator` y analiza todos los archivos `.html` y `.css` del repositorio. Si encuentra errores, **el pipeline se detiene y no despliega**.
 
#### Job 2 – Despliegue Continuo (CD): Publicar en GitHub Pages
 
```yaml
  desplegar:
    name: Desplegar en GitHub Pages
    runs-on: ubuntu-latest
    needs: validar-html          # Solo corre si el Job 1 pasó exitosamente
    if: github.ref == 'refs/heads/master'   # Solo en la rama master
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5           # Prepara GitHub Pages
      - uses: actions/upload-pages-artifact@v3     # Empaqueta los archivos
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4              # Publica el sitio
```
 
La clave `needs: validar-html` garantiza que el despliegue **nunca ocurre si la validación falla**. Esto asegura que solo código válido llega a producción.
 
#### Relación entre los dos jobs
 
```
push a master
     │
     ▼
 validar-html ──── error CSS/HTML → pipeline se cancela, sin despliegue
     │ sin errores
     ▼
  desplegar ──────── sitio publicado automáticamente en GitHub Pages
```
 
---
 
### Paso 4 – Activar GitHub Pages
 
1. Repositorio → **Settings** → **Pages**
2. En **"Source"** selecciona: **GitHub Actions**
3. Guarda los cambios
 
---
 
### Paso 5 – Verificar el Pipeline
 
1. Repositorio → pestaña **"Actions"**
2. Ver los dos jobs ejecutándose:
   - **Validar HTML** → revisa errores en el código
   - **Desplegar** → publica en GitHub Pages
3. Sitio disponible en: `https://danieljuned.github.io/Tarea-2-DanielGutierrezA-03101/`
 
---
 
## ¿Qué problema resuelve la automatización?
 
```
Desarrollador → git push → GitHub Actions → Validación → Despliegue
    (1 acción)                              (automático)   (automático)
```
 
**Sin CI/CD:** proceso manual, lento, propenso a errores, sin validación previa.  
**Con CI/CD:** un solo `git push` dispara validación + despliegue de forma automática y reproducible.
 
---
 
## Flujo del Pipeline
 
```
git push
    │
    ▼
GitHub Actions se activa
    │
    ▼
JOB 1: Validar HTML (CI) ──── Falla → detiene el proceso
    │ Pasa
    ▼
JOB 2: Desplegar (CD) ──────── Sitio publicado en GitHub Pages
```
 
---
 
## Tecnologías
 
| Tecnología | Propósito |
|---|---|
| HTML5 / CSS3 / JS | Sitio web estático |
| GitHub | Control de versiones |
| GitHub Actions | Motor CI/CD |
| html5validator | Validación de HTML |
| GitHub Pages | Hosting gratuito |
 
---
 
## Referencias
 
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [html5validator PyPI](https://pypi.org/project/html5validator/)
- [CI/CD Best Practices – Atlassian](https://www.atlassian.com/continuous-delivery)
 
---
 
*Curso 03102 – Administración de Sitios Web | 2026*