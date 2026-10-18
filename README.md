# Microservicio Backend de Productos

Microservicio RESTful construido con **Express**, **TypeScript** y **MySQL** para la gestión de productos, inventario, categorías e imágenes de la plataforma **TATSoft**. Autenticación con JWT por roles, almacenamiento de imágenes en **Azure Blob Storage**, integración con el microservicio de preventa y documentación interactiva con **Swagger**.

[![Node.js](https://img.shields.io/badge/Node.js-18.x-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.4-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Express](https://img.shields.io/badge/Express-4.19-000000?logo=express&logoColor=white)](https://expressjs.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)

> [!WARNING]
> **Despliegue temporalmente en pausa**
>
> La implementación del despliegue en Azure (Azure Web App + GitHub Actions) está **completamente configurada y funcional**, pero el entorno en producción se encuentra **detenido momentáneamente** porque el **pago de los créditos de los servicios de Azure está en pausa**. Como consecuencia, el backend publicado no está disponible en este momento. El código y los workflows permanecen intactos, por lo que la publicación se puede reactivar tan pronto como se restablezca el pago de los créditos.

## Tabla de Contenidos

- [Características](#características)
- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Endpoints Disponibles](#endpoints-disponibles)
- [Documentación de la API](#documentación-de-la-api)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Despliegue](#despliegue)
- [Seguridad](#seguridad)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)
- [Scripts Disponibles](#scripts-disponibles)

## Características

- API RESTful para productos, categorías, inventario y reportes.
- Autenticación con **JWT** y autorización por roles (`ADMINISTRADOR`, `COLABORADOR`).
- Validación de datos con **express-validator**.
- Almacenamiento de imágenes en la nube con **Azure Blob Storage** y carga de archivos con **multer**.
- Arquitectura por capas (rutas, controladores, servicios, repositorios y DTOs).
- Integración con el microservicio de **preventa** y el servicio de **usuarios**.
- Reportes de bajo stock y trazabilidad del historial de inventario.
- Documentación interactiva con **Swagger UI**.
- Tipado estático con **TypeScript** y pool de conexiones a **MySQL**.

## Arquitectura

El proyecto sigue una arquitectura en capas que separa las responsabilidades de cada componente:

- **Routes**: definen los endpoints y encadenan middlewares de autenticación y validación.
- **Controllers**: manejan la petición/respuesta HTTP.
- **Services**: contienen la lógica de negocio.
- **Repositories**: encapsulan el acceso a MySQL y a Azure Blob Storage.
- **Dto**: definen la forma de los datos de entrada y salida.
- **Middleware**: verificación de token, roles y validación de esquemas.
- **Config**: conexión centralizada a MySQL y al cliente de Azure Blob Storage.

## Requisitos Previos

- Node.js 18 o superior
- npm
- MySQL 8 o superior
- Git
- (Opcional) Cuenta de Azure Storage

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/JhoamSebastianMunoz/tatsoft-products-service.git
cd tatsoft-products-service
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Crear el archivo `.env`

```env
# Base de datos MySQL
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=usuario
DB_PASSWORD=contraseña
DB_DATABASE=microservice_product
DB_SSL=false

# Servidor
PORT=10102

# Autenticación
JWT_SECRET=tu_clave_secreta_jwt

# Azure Blob Storage
AZURE_STORAGE_CONNECTION_STRING=tu_connection_string
AZURE_STORAGE_CONTAINER_NAME=nombre_contenedor
AZURE_STORAGE_ACCOUNT_NAME=nombre_cuenta
AZURE_STORAGE_ACCOUNT_KEY=clave_cuenta

# Microservicios externos
USUARIOS_SERVICE_URL=http://localhost:10101/user/
```

### 4. Inicializar la base de datos

```bash
mysql -h localhost -u usuario -p < microservice_product.sql
```

## Configuración

| Variable                            | Descripción                            | Obligatoria |
| ----------------------------------- | -------------------------------------- | ----------- |
| `DB_HOST`                           | Host de MySQL                          | Sí          |
| `DB_PORT`                           | Puerto de MySQL (por defecto `3306`)   | No          |
| `DB_USERNAME`                       | Usuario de MySQL                       | Sí          |
| `DB_PASSWORD`                       | Contraseña de MySQL                    | Sí          |
| `DB_DATABASE`                       | Nombre de la base de datos             | Sí          |
| `DB_SSL`                            | Habilitar SSL en la conexión (`true`)  | No          |
| `PORT`                              | Puerto del servidor (por defecto `10102`) | No       |
| `JWT_SECRET`                        | Clave para firmar/verificar tokens JWT | Sí          |
| `AZURE_STORAGE_CONNECTION_STRING`   | Cadena de conexión de Azure Storage    | Sí          |
| `AZURE_STORAGE_CONTAINER_NAME`      | Contenedor de blobs para imágenes      | Sí          |
| `AZURE_STORAGE_ACCOUNT_NAME`        | Nombre de la cuenta de almacenamiento  | Sí          |
| `AZURE_STORAGE_ACCOUNT_KEY`         | Clave de la cuenta de almacenamiento   | Sí          |
| `USUARIOS_SERVICE_URL`              | URL base del microservicio de usuarios | Sí          |

## Uso

### Desarrollo

Compila en modo observador en una terminal y levanta el servidor en otra:

```bash
npx tsc -w
```

```bash
npm run build
npm start
```

La aplicación queda disponible en `http://localhost:10102`. La raíz responde con `Servidor funcionando correctamente` cuando el servicio está activo.

### Producción

```bash
npm run build
npm start
```

## Endpoints Disponibles

Todas las rutas protegidas requieren el encabezado `Authorization: Bearer <token>`.

### Productos

| Método | Endpoint                        | Descripción              | Auth              |
| ------ | ------------------------------- | ------------------------ | ----------------- |
| POST   | `/register-product`             | Registrar producto       | `ADMINISTRADOR`   |
| GET    | `/get-products`                 | Listar productos         | Autenticado       |
| GET    | `/get-product/:id_producto`     | Obtener producto por ID  | Autenticado       |
| PUT    | `/update-product/:id_producto`  | Actualizar producto      | `ADMINISTRADOR`   |
| DELETE | `/delete-product/:id_producto`  | Eliminar producto        | `ADMINISTRADOR`   |

### Inventario y Stock

| Método | Endpoint                            | Descripción                       | Auth            |
| ------ | ----------------------------------- | --------------------------------- | --------------- |
| POST   | `/register-stock/:id_producto`      | Registrar ingreso de stock        | `ADMINISTRADOR` |
| GET    | `/get-hitoricalStock`               | Historial de movimientos de stock | `ADMINISTRADOR` |
| GET    | `/get-detailsStock/:id_registro`    | Detalle de un registro de stock   | No              |

### Categorías

| Método | Endpoint                          | Descripción                | Auth                          |
| ------ | --------------------------------- | -------------------------- | ----------------------------- |
| POST   | `/register-category`              | Registrar categoría        | `ADMINISTRADOR`               |
| GET    | `/getAll-category`                | Listar categorías          | `ADMINISTRADOR`, `COLABORADOR`|
| GET    | `/getById-category/:id_categoria` | Obtener categoría por ID   | `ADMINISTRADOR`               |
| PUT    | `/update-category/:id_categoria`  | Actualizar categoría       | `ADMINISTRADOR`               |
| DELETE | `/delete_category/:id_categoria`  | Eliminar categoría         | `ADMINISTRADOR`               |

### Imágenes (Azure Blob Storage)

| Método | Endpoint                 | Descripción               | Auth |
| ------ | ------------------------ | ------------------------- | ---- |
| POST   | `/upload-image`          | Subir imagen (multipart)  | No   |
| GET    | `/get-image/:fileName`   | Obtener imagen por nombre | No   |
| DELETE | `/delete-image`          | Eliminar imagen           | No   |

### Microservicio de Preventa

| Método | Endpoint                                                  | Descripción                          | Auth |
| ------ | --------------------------------------------------------- | ------------------------------------ | ---- |
| GET    | `/api/products?ids=1,2,3`                                 | Consultar productos por ID           | No   |
| PUT    | `/api/products/actualizar-cantidad/:id_producto/:cantidad` | Descontar cantidad tras una venta    | No   |

### Reportes

| Método | Endpoint                   | Descripción                        | Auth |
| ------ | -------------------------- | ---------------------------------- | ---- |
| GET    | `/estadisticasStockBajo`   | Productos con stock por debajo del mínimo | No |

## Documentación de la API

Swagger UI está disponible una vez el servidor esté en ejecución:

```
http://localhost:10102/api-docs
```

Las especificaciones se definen en [`swagger.yaml`](./swagger.yaml).

## Estructura del Proyecto

```
tatsoft-products-service/
├── app.ts                      # Punto de entrada y montaje de rutas
├── config/                     # Conexión a MySQL y Azure Blob Storage
├── controllers/                # Manejo de peticiones HTTP por dominio
├── Dto/                        # Objetos de transferencia de datos
├── Helpers/                    # Utilidades (hash y generación de tokens)
├── middleware/                 # Autenticación, roles y validadores
├── repositories/               # Acceso a datos (MySQL y Azure)
├── routes/                     # Definición de endpoints
├── services/                   # Lógica de negocio
├── swagger.yaml                # Especificación OpenAPI
├── microservice_product.sql    # Esquema e inicialización de la base de datos
├── package.json
├── tsconfig.json
└── README.md
```

## Despliegue

El despliegue está automatizado con **GitHub Actions** en [`.github/workflows/master_backendproducts.yml`](./.github/workflows/master_backendproducts.yml):

1. Cada push a la rama `master` (o ejecución manual) dispara el workflow.
2. Se instala Node.js, se ejecutan `npm install` y `npm run build`.
3. El artefacto se empaqueta y se despliega en la **Azure Web App** `backendproducts` mediante `azure/webapps-deploy`.

> [!NOTE]
> La automatización de despliegue funciona correctamente. Actualmente la ejecución está en pausa porque el pago de los créditos de Azure está detenido, por lo que el sitio en producción no responde. Se recomienda probar el servicio en local mientras el despliegue permanezca suspendido.

## Seguridad

- Autenticación con **JWT** verificada en cada endpoint protegido (`verifyToken`).
- Autorización basada en roles mediante `checkRoleAndPermission`.
- Validación y sanitización de datos con **express-validator**.
- CORS restringido a orígenes permitidos (frontends local y desplegado).
- Credenciales y secretos gestionados por variables de entorno.
- Acceso a imágenes a través de Azure Blob Storage.

## Tecnologías Utilizadas

- Express.js · TypeScript · MySQL2
- jsonwebtoken · bcryptjs · express-validator
- @azure/storage-blob · multer · uuid
- cors · dotenv · axios
- swagger-ui-express · yamljs
