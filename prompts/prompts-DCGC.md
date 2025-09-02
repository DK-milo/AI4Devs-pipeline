# Prompts para Pipeline de GitHub Actions - DCGC

Este documento contiene los prompts utilizados para generar cada paso del pipeline de GitHub Actions que automatiza el despliegue del backend en EC2.

## 1. Tests de Backend

**Prompt utilizado:**
```
Crea un job de GitHub Actions para ejecutar tests de backend en un proyecto Node.js con TypeScript que usa:
- Jest como framework de testing
- TypeScript con ts-jest
- Prisma como ORM
- Express como framework web

El job debe:
- Ejecutarse en Ubuntu latest
- Usar Node.js 18
- Instalar dependencias con npm ci
- Ejecutar npm test
- Subir los resultados de coverage como artifact
- Usar cache de npm para optimizar el tiempo de ejecución
```

**Resultado:** Job `test` que ejecuta los tests del backend y sube los resultados de coverage.

## 2. Generación del Build del Backend

**Prompt utilizado:**
```
Crea un job de GitHub Actions para generar el build de un backend Node.js/TypeScript que:
- Depende del job de tests (needs: test)
- Compila TypeScript a JavaScript usando tsc
- Genera el cliente de Prisma
- Crea un artifact con los archivos compilados
- Usa Node.js 18 y cache de npm
- El output debe ir a la carpeta dist/
```

**Resultado:** Job `build` que compila el código TypeScript y genera el cliente de Prisma.

## 3. Despliegue del Backend en EC2

**Prompt utilizado:**
```
Crea un job de GitHub Actions para desplegar un backend Node.js en EC2 que:
- Se ejecute solo en pushes a ramas (no main/master)
- Dependa de los jobs de test y build
- Use las siguientes secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, EC2_INSTANCE, EC2_USER, EC2_PRIVATE_KEY
- Configure credenciales de AWS
- Copie los archivos compilados al servidor EC2
- Instale dependencias de producción
- Use systemctl para gestionar el servicio
- Incluya rollback automático si el despliegue falla
- El servicio debe llamarse 'backend-app'
- Los archivos deben ir a /opt/backend/
```

**Resultado:** Job `deploy` que despliega el backend en EC2 con gestión de servicios y rollback automático.

## Configuración de Secrets Requeridos

Para que el pipeline funcione correctamente, debes configurar los siguientes secrets en tu repositorio de GitHub:

1. **AWS_ACCESS_KEY_ID**: Tu Access Key ID de AWS
2. **AWS_SECRET_ACCESS_KEY**: Tu Secret Access Key de AWS  
3. **EC2_INSTANCE**: La IP pública o DNS de tu instancia EC2
4. **EC2_USER**: El usuario SSH de tu instancia EC2 (ej: ubuntu, ec2-user)
5. **EC2_PRIVATE_KEY**: Tu clave privada SSH para conectar a EC2

**Nota**: La región AWS está hardcodeada como `us-east-2` en el pipeline. Si necesitas cambiar la región, modifica el archivo `.github/workflows/pipeline.yml`.

## Configuración Adicional en EC2

Antes de usar el pipeline, asegúrate de que tu instancia EC2 tenga:

1. **Node.js 18** instalado
2. **systemd service** configurado para el backend
3. **Permisos SSH** configurados correctamente
4. **Puertos** abiertos para la aplicación

### Ejemplo de configuración del servicio systemd:

```bash
# Crear archivo de servicio
sudo nano /etc/systemd/system/backend-app.service

# Contenido del archivo:
[Unit]
Description=Backend Application
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/backend
ExecStart=/usr/bin/node dist/index.js
Restart=always
RestartSec=10
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target

# Habilitar y recargar el servicio
sudo systemctl daemon-reload
sudo systemctl enable backend-app
```

## Flujo del Pipeline

1. **Trigger**: Push a cualquier rama (excepto main/master)
2. **Tests**: Ejecuta tests del backend
3. **Build**: Compila el código TypeScript
4. **Deploy**: Despliega en EC2 solo si los tests y build pasan

El pipeline está diseñado para ser seguro y confiable, con rollback automático en caso de fallo en el despliegue.
