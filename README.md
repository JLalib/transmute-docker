# 🔄 Transmute Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/transmute-app/transmute)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Ftransmute--app%2Ftransmute-blue?logo=docker)](https://github.com/transmute-app/transmute/pkgs/container/transmute)
[![License](https://img.shields.io/badge/License-MIT-green)](https://github.com/transmute-app/transmute/blob/main/LICENSE)

## 📋 Descripción general

**Transmute** es un convertidor de archivos auto-hospedado que te permite procesar documentos, imágenes, video, audio y más directamente en tu propio servidor, garantizando privacidad total y sin límites de tamaño basados en tu almacenamiento disponible. Soporta más de 100 formatos, incluye API REST documentada (OpenAPI), autenticación integrada con soporte OIDC/SSO, y funciona como PWA instalable.

> ⚠️ **Advertencia de seguridad**: Piensa cuidadosamente antes de exponer Transmute a internet público / WAN. Transmute incluye autenticación integrada e aislamiento de datos por usuario, pero está diseñado para redes confiables. Si lo expones más allá de tu LAN, colócalo detrás de un proxy inverso con TLS y limitación de tasa. Los mantenedores no se hacen responsables de problemas de seguridad derivados de tu configuración de despliegue.

## ✨ Características principales

- 🔒 **Privacidad primero**: Los archivos se procesan en tu propio servidor y nunca se envían a terceros
- 🔐 **Soporte OIDC / SSO**: Inicio de sesión y creación de cuentas mediante proveedores OIDC como Authentik
- 📦 **Sin límites de tamaño**: Convierte archivos tan grandes como permita tu almacenamiento
- 🎯 **100+ formatos soportados**: Imágenes, video, audio, documentos, hojas de cálculo, subtítulos y fuentes
- 👥 **Autenticación integrada**: Cuentas de usuario, acceso basado en roles y soporte de API key incluido
- 🐳 **Listo para Docker**: Despliega con un solo comando, sin configuración compleja requerida
- 🔌 **API REST**: Automatiza e integra conversiones de archivos mediante la API OpenAPI documentada
- 🎨 **Múltiples temas**: Siete temas light y dark integrados en la interfaz de usuario
- 📱 **PWA instalable**: Funciona como Progressive Web App para uso offline y móvil

## 📋 Requisitos del sistema

- Docker Engine 20.10+
- Docker Compose v2 (plugin `docker compose`) o Docker Compose standalone 1.29+
- Mínimo 512 MB RAM disponible (recomendado 1 GB+)
- Espacio en disco según archivos a procesar (volumen persistente `transmute_data`)
- Puerto 3313 libre en el host (o el que configures en `APP_URL`)

## 🐳 Instalación

### 1. Crear archivo `docker-compose.yml`

```yaml
version: '3.8'

services:
  transmute:
    image: ghcr.io/transmute-app/transmute:latest
    container_name: transmute
    restart: unless-stopped
    ports:
      - "3313:3313"
    environment:
      # Advertencia: JWT_SECRET debe configurarse en tu archivo .env o entorno
      # ¡Nunca uses valores por defecto en producción!
      - JWT_SECRET=${JWT_SECRET}
      - APP_URL=${APP_URL:-http://localhost:3313}
      - NODE_ENV=${NODE_ENV:-production}
      - PORT=3313
    volumes:
      - transmute_data:/app/data
      # Opcional: monta una hoja de estilos personalizada para salida Markdown/PDF.
      # - ./pdf-custom.css:/app/data/pdf/custom.css:ro
    healthcheck:
      test:
        - "CMD"
        - "wget"
        - "-q"
        - "-O"
        - "/dev/null"
        - "--tries=1"
        - "http://localhost:3313/api/health/ready"
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

volumes:
  transmute_data:
```

### 2. Crear archivo `.env` (obligatorio)

```bash
# Genera un secreto seguro: openssl rand -base64 32
JWT_SECRET=tu_secreto_super_seguro_aqui

# URL pública donde accederás a Transmute (incluye esquema y puerto si no es 80/443)
APP_URL=http://localhost:3313

# Entorno de ejecución
NODE_ENV=production
```

### 3. Iniciar Transmute

```bash
# Levantar en segundo plano
docker compose up -d

# Ver logs en tiempo real (esperar healthcheck OK)
docker compose logs -f transmute
```

## ⚙️ Configuración

1. **JWT_SECRET (obligatorio)**: Clave secreta para firmar tokens JWT. Genera una con `openssl rand -base64 32` y guárdala en `.env`. **Nunca uses valores por defecto en producción**.

2. **APP_URL**: URL completa (esquema + host + puerto) donde será accesible Transmute. Ejemplos:
   - Local: `http://localhost:3313`
   - LAN: `http://192.168.1.50:3313`
   - Dominio con proxy: `https://transmute.midominio.com`

3. **NODE_ENV**: Entorno de ejecución (`production` | `development`). Usa `production` en entornos reales.

4. **PORT**: Puerto interno del contenedor (por defecto 3313). No cambiar a menos que sepas lo que haces.

5. **Volumen `transmute_data`**: Persiste base de datos SQLite, archivos subidos, configuración de usuarios y temas. **No elimines este volumen** salvo que quieras resetear la instancia.

6. **CSS personalizado (opcional)**: Monta `./pdf-custom.css:/app/data/pdf/custom.css:ro` para personalizar la salida Markdown/PDF.

7. **Healthcheck**: Verifica endpoint `/api/health/ready` cada 30s. El contenedor se marca `healthy` solo cuando la API responde correctamente.

## 🚀 Primeros pasos

1. **Clona o crea el directorio del proyecto** y sitúate en él:
   ```bash
   mkdir transmute && cd transmute
   ```

2. **Crea `docker-compose.yml`** con el contenido de la sección [Instalación](#-instalación).

3. **Crea `.env`** con tu `JWT_SECRET` y `APP_URL` (ver [Configuración](#-configuración)).

4. **Levanta la pila**:
   ```bash
   docker compose up -d
   ```

5. **Verifica que el healthcheck pase** (puede tardar 30-60s en el primer arranque):
   ```bash
   docker compose ps
   # STATUS debe mostrar: Up ... (healthy)
   ```

6. **Accede a la interfaz web**:
   - Abre en tu navegador: `http://localhost:3313` (o tu `APP_URL`)
   - Regístrate como primer usuario (se convierte en administrador automáticamente)

7. **Prueba una conversión**: Arrastra un archivo, elige formato de salida y descarga el resultado.

8. **Explora la API**: Visita `http://localhost:3313/api/docs` para la documentación OpenAPI/Swagger interactiva.

## 💡 Casos de uso

- 📄 **Conversión masiva de documentos**: Oficinas que necesitan pasar DOCX → PDF, XLSX → CSV, etc., sin subir datos a la nube
- 🖼️ **Procesamiento de imágenes**: Conversión HEIC/WEBP/AVIF → JPEG/PNG, redimensionado, extracción de metadatos
- 🎬 **Transcodificación de media**: Video/audio a formatos compatibles con dispositivos antiguos o streaming local
- 🔌 **Automatización via API**: Integración en pipelines CI/CD, bots de Telegram/Discord, webhooks de Nextcloud/ownCloud
- 🏠 **Homelab privado**: Sustituto de servicios como CloudConvert, Zamzar o Convertio con control total de datos
- 📱 **PWA móvil**: Instala en Android/iOS para convertir archivos desde el compartir del sistema sin app nativa

## 🔒 Acceso remoto seguro

> **Transmute no incluye TLS ni rate-limiting nativos**. Para exponerlo fuera de tu LAN:

1. **Usa un proxy inverso** (Nginx Proxy Manager, Traefik, Caddy, SWAG) con:
   - Certificados TLS válidos (Let's Encrypt o propios)
   - `X-Forwarded-Proto`, `X-Forwarded-Host` headers
   - Rate limiting (ej. 10 req/s por IP)
   - Bloqueo de IPs sospechosas (fail2ban / crowdsec)

2. **Configura `APP_URL`** con el esquema `https://` y tu dominio público.

3. **Restringe puertos**: Solo expón 80/443 en el firewall; el puerto 3313 queda interno.

4. **Habilita OIDC/SSO** (Authentik, Keycloak, Authelia) para 2FA y gestión centralizada de usuarios.

5. **Audita logs** regularmente: `docker compose logs transmute | grep -i error`

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| Ver estado y healthcheck | `docker compose ps` |
| Ver logs en vivo | `docker compose logs -f transmute` |
| Reiniciar servicio | `docker compose restart transmute` |
| Actualizar imagen | `docker compose pull && docker compose up -d` |
| Backup de datos | `docker run --rm -v transmute_transmute_data:/data -v $(pwd):/backup alpine tar czf /backup/transmute-backup-$(date +%F).tar.gz -C /data .` |
| Restaurar backup | `docker run --rm -v transmute_transmute_data:/data -v $(pwd):/backup alpine tar xzf /backup/transmute-backup-YYYY-MM-DD.tar.gz -C /data` |
| Resetear instancia (¡borra todo!) | `docker compose down -v && docker compose up -d` |
| Acceder a shell del contenedor | `docker compose exec transmute sh` |

**Actualizaciones**: La etiqueta `:latest` sigue la rama principal. Para versiones fijas usa `:v1.2.3` (ver [releases](https://github.com/transmute-app/transmute/releases)).

## 📝 Licencia

Este proyecto de despliegue Docker se distribuye bajo licencia **MIT**. El código de Transmute propiamente dicho tiene su propia licencia en el [repositorio oficial](https://github.com/transmute-app/transmute/blob/main/LICENSE).

---

> 📖 **Guía completa en el blog**: [Cómo instalar y configurar Transmute en Docker | Archivos | Conversión | PWA | Auto-hospedado](https://genbyte.blogspot.com/2026/10/como-instalar-y-configurar-transmute-en.html)