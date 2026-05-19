# Informe - Laboratorio 6.1: Despliegue de Sitio Web Estático con CI/CD

## 1. Descripción del Proyecto

**Casa Gabriel** es una landing page inmobiliaria que muestra una propiedad residencial exclusiva. El sitio está construido con HTML5, CSS3 y JavaScript vanilla, y se despliega automáticamente en Amazon S3 mediante un pipeline de CI/CD configurado en GitHub Actions.

### Estructura del sitio

| Sección | Descripción |
|---|---|
| Hero | Imagen principal con título, estadísticas de la propiedad y llamada a la acción |
| Características | 6 características destacadas de la propiedad |
| Galería | Tour visual de las habitaciones principales |
| Precio | Información de inversión y financiamiento |
| Contacto | Formulario de contacto y datos de la propiedad |

### Archivos del proyecto

```
casa-gabriel/
├── index.html              → Página principal
├── css/
│   └── style.css           → Estilos con paleta cálida
├── js/
│   └── app.js              → Interactividad y animaciones
└── .github/
    └── workflows/
        └── deploy.yml      → Pipeline de despliegue
```

---

## 2. Descripción del Pipeline de CD

### Flujo completo

```
Push a main
     ↓
GitHub Actions — deploy.yml
     ↓
┌─────────────────────────────────────────┐
│  Step 1: Checkout del código            │
│  → Clona el repositorio en el runner   │
└─────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────┐
│  Step 2: Configurar credenciales AWS    │
│  → Usa secrets de GitHub               │
│  → Autentica con IAM                   │
└─────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────┐
│  Step 3: Sincronizar con S3             │
│  → aws s3 sync                         │
│  → Sube archivos nuevos/modificados    │
│  → Elimina archivos borrados (--delete)│
│  → Excluye .git, .github, docs         │
└─────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────┐
│  Step 4: Invalidar caché CloudFront     │
│  → Solo si está configurado             │
│  → Fuerza que los usuarios vean        │
│     la versión más reciente            │
└─────────────────────────────────────────┘
     ↓
Sitio web actualizado en segundos ✅
```

### Decisiones técnicas

**`aws s3 sync` en lugar de `aws s3 cp`:**
`sync` es inteligente — solo sube archivos que cambiaron, no todos cada vez. Esto hace el despliegue más rápido y eficiente.

**Flag `--delete`:**
Elimina del bucket los archivos que ya no existen en el repositorio. Mantiene el bucket sincronizado exactamente con el estado del repositorio.

**Exclusiones:**
Se excluyen archivos que no deben ser públicos como `.git/`, `.github/`, `INFORME.md` y `.gitignore`.

**CloudFront + invalidación:**
CloudFront cachea los archivos en servidores alrededor del mundo para servir el sitio más rápido. Al invalidar `/*` después de cada deploy, forzamos que todos los servidores descarguen la versión más reciente.

**Usuario IAM con permisos mínimos:**
Se creó un usuario IAM exclusivo para GitHub Actions con acceso solo al bucket específico. Nunca se usan credenciales de la cuenta raíz de AWS.

---

## 3. Evidencias

### 3.1 Bucket S3 creado y configurado

> 📸 _Insertar captura del bucket S3 en la consola de AWS_

---

### 3.2 Static Website Hosting habilitado

> 📸 _Insertar captura de Properties → Static website hosting habilitado con index.html_

---

### 3.3 Política de bucket configurada

> 📸 _Insertar captura de Permissions → Bucket policy con PublicReadGetObject_

---

### 3.4 Usuario IAM creado

> 📸 _Insertar captura del usuario github-actions-s3-deploy en IAM_

---

### 3.5 Secretos y variables en GitHub

> 📸 _Insertar captura de Settings → Secrets mostrando AWS_ACCESS_KEY_ID y AWS_SECRET_ACCESS_KEY_

> 📸 _Insertar captura de Settings → Variables mostrando AWS_REGION y AWS_S3_BUCKET_

---

### 3.6 Pipeline ejecutándose en Actions

> 📸 _Insertar captura del workflow corriendo en la pestaña Actions_

---

### 3.7 Pipeline exitoso con todos los steps

> 📸 _Insertar captura del workflow completado con ✅ en cada step_

---

### 3.8 Archivos sincronizados en el bucket

> 📸 _Insertar captura de S3 → Objects mostrando index.html, css/ y js/_

---

### 3.9 Sitio web funcionando en URL de S3

> 📸 _Insertar captura del navegador mostrando Casa Gabriel en la URL de S3_

---

### 3.10 Distribución de CloudFront configurada

> 📸 _Insertar captura de CloudFront mostrando la distribución activa_

---

### 3.11 Sitio web funcionando en URL de CloudFront (HTTPS)

> 📸 _Insertar captura del navegador con HTTPS en la URL de CloudFront_

---

### 3.12 Fallo intencional corregido

> 📸 _Insertar captura del workflow fallando ❌ y luego el fix ✅_

---

## 4. URLs del proyecto

| Entorno | URL |
|---|---|
| S3 (HTTP) | `http://NOMBRE-BUCKET.s3-website-REGION.amazonaws.com` |
| CloudFront (HTTPS) | `https://XXXXXXXXXX.cloudfront.net` |

---

## 5. Conclusiones

El despliegue continuo de sitios estáticos en S3 ofrece ventajas claras:

**Velocidad de entrega:** Cualquier cambio en el sitio llega a producción en segundos, sin procesos manuales de FTP o SSH al servidor.

**Costo casi nulo:** S3 en el Free Tier de AWS cuesta prácticamente cero para sitios con tráfico moderado. No hay servidor que mantener ni pagar.

**Escalabilidad automática:** S3 sirve el sitio sin importar si hay 1 o 1,000,000 visitantes simultáneos. CloudFront distribuye el contenido globalmente para máxima velocidad.

**Seguridad:** Las credenciales AWS nunca están en el código. Están cifradas como secrets en GitHub y se usan solo durante la ejecución del workflow.

**Reproducibilidad:** El estado del sitio en producción siempre refleja exactamente el estado del repositorio en la rama main. No hay configuración manual que pueda desincronizarlos.

**Diferencia con despliegue de API:** A diferencia del Lab 5.2 donde desplegamos una API con Docker, aquí no hay servidor, no hay contenedor y no hay health check. S3 simplemente sirve archivos estáticos. Es más simple, más barato y más confiable para este tipo de contenido.
