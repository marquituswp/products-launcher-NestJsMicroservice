### English

# E-Commerce Microservices Platform (Monorepo)

This repository serves as the centralized build and orchestration layer for all microservices in the E-Commerce platform. It contains:

- **Dockerfiles & Build Config**: Definitions to build container images for each service.
- **Git Submodules**: References to individual microservice repositories (Product, Order, Payment, Auth, and API Gateway).
- **Docker Compose & Helm Charts**: Local development setup via Docker Compose and production deployment manifests for Kubernetes (Helm).
- **CI/CD Pipelines**: GitHub Actions workflows that build images, run tests, publish to Container Registry, and update deployments.
- **Google Cloud Integration**: Scripts and configurations for deploying to Google Cloud Platform.
- **NATS**: Message broker for inter-service communication.
- **Stripe**: Payment processing integration.
- **Prisma**: ORM for database interactions.

### Spanish

# Plataforma de Microservicios E-Commerce (Monorepo)

Este repositorio sirve como la capa de construcción y orquestación centralizada para todos los microservicios en la plataforma E-Commerce. Contiene:

- **Dockerfiles & Configuración de Construcción**: Definiciones para construir imágenes de contenedor para cada servicio.
- **Git Submodules**: Referencias a repositorios de microservicios individuales (Producto, Pedido, Pago, Autenticación y API Gateway).
- **Docker Compose & Helm Charts**: Configuración de desarrollo local a través de Docker Compose y manifiestos de despliegue para producción en Kubernetes (Helm).
- **Pipelines CI/CD**: Flujos de trabajo de GitHub Actions que construyen imágenes, ejecutan pruebas, publican en el Registro de Contenedores y actualizan despliegues.
- **Integración con Google Cloud**: Scripts y configuraciones para desplegar en Google Cloud Platform.
- **NATS**: Broker de mensajes para la comunicación entre servicios.
- **Stripe**: Integración de procesamiento de pagos.

## Dev

1. Clonar el repositorio
2. Crear un .env basado en el .env.template
3. Ejecutar el comando `git submodule update --init --recursive` para reconstruir los sub-módulos
4. Ejecturar el comando `docker-compose up --build` para crear y correr los contenedores de docker

## PROD

1. Clonar el repositorio
2. Crear un .env basado en el .env.template
3. Ejecutar el comando

```
docker compose -f docker-compose.prod.yml build
```

4. Correr el comando

```
docker compose -f docker-compose.prod.yml up
```

### Pasos para crear los Git Submodules

1. Crear un nuevo repositorio en GitHub
2. Clonar el repositorio en la máquina local
3. Añadir el submodule, donde `repository_url` es la url del repositorio y `directory_name` es el nombre de la carpeta donde quieres que se guarde el sub-módulo (no debe de existir en el proyecto)

```
git submodule add <repository_url> <directory_name>
```

4. Añadir los cambios al repositorio (git add, git commit, git push)
   Ej:

```
git add .
git commit -m "Add submodule"
git push
```

5. Inicializar y actualizar Sub-módulos, cuando alguien clona el repositorio por primera vez, debe de ejecutar el siguiente comando para inicializar y actualizar los sub-módulos

```
git submodule update --init --recursive
```

6. Para actualizar las referencias de los sub-módulos

```
git submodule update --remote
```

## Importante

Si se trabaja en el repositorio que tiene los sub-módulos, **primero actualizar y hacer push** en el sub-módulo y **después** en el repositorio principal.

Si se hace al revés, se perderán las referencias de los sub-módulos en el repositorio principal y tendremos que resolver conflictos.

## STRIPE

Para poder probar el sistema de pagos, es necesario crear una cuenta en Stripe y añadir las credenciales en el archivo `.env`.
Para ello, seguir los siguientes pasos:

1. Crear una cuenta en Stripe: [Stripe](https://dashboard.stripe.com/register)
2. Ir a la sección de "Developers" y luego a "API keys". Ahí encontrarás las claves de prueba y producción.
3. Copiar las claves y pegarlas en el archivo `.env` de tu proyecto. Las claves que necesitas son:

   ```
   STRIPE_SECRET_KEY=sk_test_4eCk2j2v2j2v2j2v2j2v2j2v2
   STRIPE_PUBLIC_KEY=pk_test_4eCk2j2v2j2v2j2v2j2v2j2v2
   ```

4. Ir a la sección de "Webhooks" y crear un nuevo webhook. La URL del webhook debe ser `http://localhost:3000/api/v1/payments/webhook` para el entorno de desarrollo.
5. Seleccionar los eventos que quieres recibir. Para este proyecto, selecciona los siguientes eventos:
   - `charge.succeeded`
6. Copiar la clave de firma del webhook y pegarla en el archivo `.env` de tu proyecto. La clave que necesitas es:

   ```
   STRIPE_WEBHOOK_SECRET=whsec_4eCk2j2v2j2v2j2v2j2v2j2v2
   ```

7. Guardar los cambios en el archivo `.env` y reiniciar el servidor de desarrollo.

Para escuchar el webhook de Stripe, es necesario tener el servidor corriendo y el webhook configurado en Stripe. Para ello, puedes usar una herramienta como [hookdeck](https://hookdeck.com/) para exponer tu servidor local a internet.

Luego, ejecutar el siguiente comando para iniciar hookdeck:

```bash
hookdeck listen 3003 stripe-to-localhost
```

Esto creará un túnel a tu servidor local y podrás recibir los eventos de Stripe en tu servidor de desarrollo.

## Notas

- **FUNCIONAMIENTO CORRECTO DE WATCH MODE EN WINDOWS**:

Para que el watch mode funcione correctamente en windows, es necesario añadir la siguiente línea en el `docker-compose.yml`:

```yaml
environment:
  - CHOKIDAR_USEPOLLING=true
```

Además, hay que añadir lo siguiente en el `tsconfig.json`:

```json
   "watchOptions": {
    "watchFile": "fixedPollingInterval"
}
```

- **MICROSERVICES MESSAGES PATTERNS**:

````
foo.* matches foo.bar, foo.baz, etc. but not foo.bar.baz
foo.*.bar matches foo.baz.bar, foo.qux.bar, etc. but not foo.bar or foo.bar.baz
foo.> matches foo.bar, foo.bar.baz, etc.
```
````

## Google Cloud

1. Crear un proyecto en Google Cloud
2. Crear un container registry en Google Cloud
3. Crear un bucket en Google Cloud Storage
4. Instalación de Google Cloud SDK
5. Autenticación de Google Cloud SDK

## Subir imagenes Docker a Google Cloud

1. Construir las imagenenes Docker

```bash
docker build -t gcr.io/<PROJECT_ID>/<IMAGE_NAME> . || docker compose -f docker-compose.prod.yml build
```

2. Subir la imagen Docker a Google Cloud

```bash
docker image push gcr.io/<PROJECT_ID>/<IMAGE_NAME>
```
