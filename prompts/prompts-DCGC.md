# Prompts para Pipeline GitHub Actions - AI4Devs Backend

## Información del Proyecto
- **Repositorio**: https://github.com/DK-milo/AI4Devs-pipeline
- **Tecnologías**: Node.js, Express, TypeScript, Prisma ORM, PostgreSQL
- **Arquitectura**: Clean Architecture (application, domain, infrastructure, presentation)

---

## 1. Prompt para Tests de Backend

### Contexto
Necesito crear una configuración de tests para un backend Node.js/Express con TypeScript que usa Prisma como ORM y PostgreSQL como base de datos.

### Prompt Optimizado
```
Crea una configuración completa de tests para un backend Node.js/Express con TypeScript que incluya:

1. **Configuración de entorno de test**:
   - Base de datos PostgreSQL en memoria o contenedor Docker
   - Variables de entorno específicas para testing
   - Setup y teardown de base de datos

2. **Tests unitarios**:
   - Tests para la capa de dominio (modelos y reglas de negocio)
   - Tests para la capa de aplicación (casos de uso)
   - Tests para la capa de infraestructura (repositorios con Prisma)

3. **Tests de integración**:
   - Tests para endpoints de API REST
   - Tests end-to-end que incluyan base de datos
   - Validación de schemas de entrada y salida

4. **Configuración de herramientas**:
   - Jest como framework de testing
   - Supertest para tests de API
   - Coverage reporting
   - Scripts npm para ejecutar tests

5. **Estructura del proyecto**:
   ```
   backend/
   ├── src/
   │   ├── application/
   │   ├── domain/
   │   ├── infrastructure/
   │   └── presentation/
   └── tests/
       ├── unit/
       ├── integration/
       └── helpers/
   ```

Incluye ejemplos de tests para un endpoint de candidatos con operaciones CRUD y manejo de errores.
```

---

## 2. Prompt para Build del Backend

### Contexto
Necesito configurar el proceso de build para un backend TypeScript con Prisma que sea optimizado para producción.

### Prompt Optimizado
```
Crea una configuración de build optimizada para producción de un backend Node.js/Express con TypeScript y Prisma que incluya:

1. **Configuración de TypeScript**:
   - tsconfig.json optimizado para producción
   - Compilación de TypeScript a JavaScript
   - Source maps para debugging
   - Exclusión de archivos de test y desarrollo

2. **Build del proyecto**:
   - Script npm para build completo
   - Generación del cliente Prisma
   - Compilación de assets estáticos si los hay
   - Minificación y optimización de código

3. **Preparación para deployment**:
   - Estructura de directorios para producción
   - Copia de archivos necesarios (package.json, prisma schema)
   - Exclusión de devDependencies
   - Creación de archivo comprimido para deployment

4. **Validaciones pre-build**:
   - Verificación de sintaxis TypeScript
   - Lint del código
   - Verificación de dependencias

5. **Scripts npm necesarios**:
   ```json
   {
     "scripts": {
       "build": "...",
       "build:prod": "...",
       "prebuild": "...",
       "postbuild": "..."
     }
   }
   ```

El resultado debe ser una aplicación lista para deployment en servidor de producción con todas las dependencias y configuraciones necesarias.
```

---

## 3. Prompt para Despliegue en EC2

### Contexto
Necesito automatizar el despliegue de un backend Node.js/Express con base de datos PostgreSQL en una instancia EC2 usando GitHub Actions.

### Prompt Optimizado
```
Crea un proceso automatizado de despliegue a EC2 para un backend Node.js/Express con las siguientes características:

1. **Configuración de EC2**:
   - Conexión SSH segura usando claves privadas
   - Instalación automática de dependencias (Node.js, PM2, PostgreSQL)
   - Configuración de puertos y security groups
   - Setup de usuario y permisos

2. **Proceso de despliegue**:
   - Transferencia segura de archivos build via SCP
   - Backup de versión anterior antes del deploy
   - Extracción y configuración de nueva versión
   - Instalación de dependencias de producción solamente

3. **Gestión de base de datos**:
   - Ejecución automática de migraciones Prisma
   - Manejo de conexiones de base de datos
   - Rollback en caso de fallos de migración

4. **Gestión de procesos**:
   - Uso de PM2 para gestión de procesos
   - Graceful restart sin downtime
   - Monitoreo de salud de la aplicación
   - Logs y debugging

5. **Configuración de ambiente**:
   - Variables de entorno de producción
   - Secrets management para credenciales sensibles
   - SSL/TLS setup si es necesario

6. **Rollback y recovery**:
   - Estrategia de rollback automático en caso de fallo
   - Health checks post-deployment
   - Notificaciones de éxito/fallo

7. **Secrets de GitHub necesarios**:
   - AWS_ACCESS_KEY_ID
   - AWS_SECRET_ACCESS_KEY
   - EC2_HOST
   - EC2_PRIVATE_KEY
   - DATABASE_URL
   - AWS_REGION

Incluye comandos específicos para cada paso y manejo de errores robusto.
```

---

## Estructura de Archivos Requeridos

### Archivos que debes crear en tu repositorio:

1. **`.github/workflows/pipeline.yml`** - El pipeline principal
2. **`backend/jest.config.js`** - Configuración de Jest para tests
3. **`backend/.env.test`** - Variables de entorno para testing
4. **`backend/tests/`** - Directorio con tests unitarios e integración

### Secrets de GitHub a configurar:

1. `AWS_ACCESS_KEY_ID` - ID de clave de acceso AWS
2. `AWS_SECRET_ACCESS_KEY` - Clave secreta AWS
3. `AWS_REGION` - Región AWS (ej: us-east-1)
4. `EC2_HOST` - IP pública o DNS de tu instancia EC2
5. `EC2_USER` - Usuario SSH (normalmente 'ec2-user' o 'ubuntu')
6. `EC2_PRIVATE_KEY` - Clave privada SSH para acceder a EC2
7. `DATABASE_URL` - URL de conexión a PostgreSQL en producción
8. `APP_NAME` - Nombre de la aplicación (opcional, por defecto: ai4devs-backend)

---

## Comandos para Configurar EC2

### Preparación inicial de la instancia EC2:

```bash
# Conectar a EC2
ssh -i your-key.pem ec2-user@your-ec2-ip

# Instalar Node.js
curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Instalar PM2
sudo npm install -g pm2

# Configurar PM2 para auto-start
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user

# Instalar PostgreSQL (si no usas RDS)
sudo yum install -y postgresql postgresql-server
sudo postgresql-setup initdb
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

### Security Group requerido:
- Puerto 22 (SSH)
- Puerto 3010 (Backend API)
- Puerto 5432 (PostgreSQL) - solo si la DB está en la misma instancia

---

## Notas Importantes

1. **Base de datos**: El pipeline asume PostgreSQL. Si usas RDS, ajusta la configuración de DATABASE_URL.

2. **Environment**: El job de deploy usa `environment: production` para requerir aprobación manual si lo configuras.

3. **Health Check**: Incluye verificación de salud post-deployment en el puerto 3010.

4. **Rollback**: En caso de fallo, PM2 mantendrá la versión anterior ejecutándose.

5. **Seguridad**: Todas las credenciales se manejan via GitHub Secrets.

---

## Personalización por Tecnología

Este pipeline está optimizado para el stack específico del repositorio AI4Devs-pipeline:
- **Runtime**: Node.js 18
- **Framework**: Express.js
- **Language**: TypeScript
- **ORM**: Prisma
- **Database**: PostgreSQL
- **Process Manager**: PM2
- **Architecture**: Clean Architecture