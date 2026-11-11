# Docker + Azure Container Registry + App Service

Este proyecto utiliza **Create React App**, no Vite. `npm run build` produce la carpeta `build/`.
La imagen Docker compila con Node.js y sirve el contenido estatico con Nginx en el **puerto 80**.

## 1. Probar localmente

Desde la carpeta que contiene el `Dockerfile`:

```bash
docker build -t reactbasicapp:v1 .
docker run --rm -p 8080:80 --name reactbasicapp reactbasicapp:v1
```

Abrir http://localhost:8080 . La verificacion de salud es http://localhost:8080/health .
En una Mac con Apple Silicon, para producir una imagen AMD64 localmente:

```bash
docker buildx build --platform linux/amd64 -t reactbasicapp:v1 --load .
```

## 2. Crear Azure Container Registry y publicar imagen

Necesitas Azure CLI y una suscripcion seleccionada (`az login`; `az account show`).
Sustituir el nombre `miacrreactunico123` por uno **globalmente unico**, solo letras y numeros.

```bash
az group create --name rg-react-docker --location eastus
az acr create --resource-group rg-react-docker --name miacrreactunico123 --sku Basic
az acr build --registry miacrreactunico123 --image reactbasicapp:v1 .
```

`az acr build` compila en Azure y evita inconvenientes de arquitectura ARM64/AMD64.
La imagen resultante sera `miacrreactunico123.azurecr.io/reactbasicapp:v1`.

## 3. Desplegar en Azure App Service (contenedor Linux)

En Azure Portal:

1. Crear **Web App** con sistema operativo **Linux** y tipo de publicacion **Container**.
2. Elegir **Azure Container Registry**, el registro y la imagen `reactbasicapp`, etiqueta `v1`.
3. Configurar acceso de la Web App al ACR (preferentemente **identidad administrada** con rol **AcrPull** sobre el registro; comprobar que la configuracion de despliegue use esa identidad).
4. En las variables de entorno / configuracion de la Web App establecer `WEBSITES_PORT=80`.
5. Reiniciar la Web App y comprobar la URL que proporciona Azure.

La Web App requiere un plan compatible con contenedores Linux.

## Importante sobre Azure Static Web Apps

Azure Static Web Apps **no consume directamente** imagenes de ACR. El repositorio ya tiene workflows de GitHub Actions para Static Web Apps; se conservan, pero **no fueron cambiados**. Para publicar alli se usa el build estatico de CRA (`build/`) mediante un pipeline de Static Web Apps, no la imagen Docker.

## Archivos agregados

- `Dockerfile`: compilacion multi-etapa y Nginx.
- `.dockerignore`: limita archivos enviados al build.
- `nginx.conf`: rutas SPA, cache de assets y endpoint `/health`.

No se cambiaron `src/`, `public/`, `package.json` ni `package-lock.json`.
