# 🔄 Transmute Docker

[![GitHub](https://img.shields.io/badge/GitHub-transmute--app%2Ftransmute-181717?logo=github)](https://github.com/transmute-app/transmute)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Ftransmute--app%2Ftransmute-2496ED?logo=docker)](https://github.com/transmute-app/transmute/pkgs/container/transmute)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/transmute-app/transmute/blob/main/LICENSE)

## 📋 Descripción general

**Transmute** es un convertidor de archivos auto-hospedado que permite procesar documentos, imágenes, video, audio y más directamente en tu propio servidor, garantizando privacidad total y sin límites de tamaño basados en tu almacenamiento disponible. Soporta más de 100 formatos, incluye API REST documentada, autenticación integrada con soporte OIDC/SSO y funciona como PWA.

## ✨ Características principales

- 🔒 **Privacidad primero**: Los archivos se procesan en tu propio servidor y nunca se envían a terceros
- 🔐 **Soporte OIDC / SSO**: Inicio de sesión y creación de cuentas mediante proveedores OIDC como Authentik
- 📦 **Sin límites de tamaño**: Convierte archivos tan grandes como permita tu almacenamiento
- 🎯 **100+ formatos soportados**: Imágenes, video, audio, documentos, hojas de cálculo, subtítulos y fuentes
- 👥 **Autenticación integrada**: Cuentas de usuario, acceso basado en roles y soporte de API key incluido
- 🐳 **Listo para Docker**: Despliega con un solo comando, sin configuración compleja requerida
- 🔌 **API REST**: Automatiza e integra conversiones de archivos mediante la API OpenAPI documentada
- 🎨 **Múltiples temas**: Siete temas light y dark integrados en la interfaz de usuario

## 📋 Requisitos del sistema

- Docker Engine 20.10+
- Docker Compose v2 (plugin `docker compose`) o docker-compose standalone
- Mínimo 512 MB RAM disponible (recomendado 1 GB+)
- Espacio en disco según archivos a procesar
- Puerto 3313 libre (o el que configures en `APP_URL`)

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
# Genera un secreto seguro con: openssl rand -base64 32
JWT_SECRET=tu_secreto_super_seguro_aqui

# URL pública donde accederás a Transmute (incluye esquema y puerto si no es 80/443)
APP_URL=http://localhost:3313

# Entorno de ejecución
NODE_ENV=production
```

### 3. Iniciar Transmute

```bash
# Levantar el contenedor en segundo plano
docker compose up -d

# Ver logs en tiempo real para confirmar arranque correcto
docker compose logs -f transmute
```

## ⚙️ Configuración

1. **JWT_SECRET (obligatorio)**: Clave secreta para firmar tokens JWT. Genera una con `openssl rand -base64 32` y guárdala en `.env`. **Nunca uses el valor por defecto en producción**.

2. **APP_URL**: URL completa (esquema + host + puerto) donde será accesible Transmute. Ejemplos:
   - Local: `http://localhost:3313`
   - LAN: `http://192.168.1.50:3313`
   - Dominio con proxy inverso: `https://transmute.midominio.com`

3. **NODE_ENV**: Entorno de ejecución (`production` | `development`). Usa `production` siempre.

4. **PORT**: Puerto interno del contenedor (por defecto `3313`). No cambiar a menos que sepas lo que haces.

5. **Volumen `transmute_data`**: Persiste base de datos SQLite, archivos subidos, configuración y cachés. Haz backups periódicos de este volumen.

6. **CSS personalizado (opcional)**: Descomenta la línea en `volumes` y crea `pdf-custom.css` para personalizar la salida Markdown/PDF.

## 🚀 Primeros pasos

1. **Clona o crea el directorio del proyecto**:
   ```bash
   mkdir transmute && cd transmute
   ```

2. **Crea el `docker-compose.yml`** con el contenido de la sección [Instalación](#-instalación).

3. **Crea el archivo `.env`** con tus valores reales (ver [Configuración](#-configuración)).

4. **Inicia la pila**:
   ```bash
   docker compose up -d
   ```

5. **Verifica que el healthcheck pase** (puede tardar ~40s en el primer arranque):
   ```bash
   docker compose ps
   # STATUS debe mostrar "healthy"
   ```

6. **Accede a la interfaz web**:
   - Abre en tu navegador: `http://localhost:3313` (o tu `APP_URL`)
   - Regístrate como primer usuario (se convierte en administrador automáticamente)

7. **Prueba una conversión**: Arrastra un archivo, elige formato de salida y descarga el resultado.

## 💡 Casos de uso

- 📄 Conversión masiva de documentos ofimáticos (docx ↔ pdf, xlsx ↔ csv, etc.)
- 🖼️ Procesamiento por lotes de imágenes (redimensionar, cambiar formato, optimizar)
- 🎬 Transcodificación de video/audio a formatos compatibles con tus dispositivos
- 🔌 Automatización vía API REST en pipelines CI/CD o scripts propios
- 🏠 Alternativa privada a servicios cloud (CloudConvert, Zamzar, etc.) sin límites de archivo
- 📱 Uso como PWA instalable en móvil/tablet para conversiones sobre la marcha

## 🔒 Acceso remoto seguro

> ⚠️ **Advertencia oficial**: Piensa cuidadosamente antes de exponer Transmute a internet público / WAN. Transmute incluye autenticación integrada e aislamiento de datos por usuario, pero está diseñado para redes confiables. Si lo expones más allá de tu LAN, colócalo detrás de un proxy inverso con TLS y limitación de tasa. Los mantenedores no se hacen responsables de problemas de seguridad derivados de tu configuración de despliegue.

**Recomendaciones para exposición externa**:
- Usa un proxy inverso (Nginx Proxy Manager, Traefik, Caddy) con **TLS válido** (Let's Encrypt)
- Habilita **rate limiting** en el proxy (ej. 10 req/s por IP)
- Restringe acceso por **IP/CIDR** si es posible (VPN, Tailscale, Cloudflare Access)
- Configura `APP_URL` con `https://` y dominio real
- Mantén `JWT_SECRET` único, largo y fuera de control de versiones

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| Ver logs en vivo | `docker compose logs -f transmute` |
| Ver estado y healthcheck | `docker compose ps` |
| Reiniciar servicio | `docker compose restart transmute` |
| Actualizar a última imagen | `docker compose pull && docker compose up -d` |
| Backup del volumen (datos) | `docker run --rm -v transmute_transmute_data:/data -v $(pwd):/backup alpine tar czf /backup/transmute-backup-$(date +%F).tar.gz -C /data .` |
| Restaurar backup | `docker run --rm -v transmute_transmute_data:/data -v $(pwd):/backup alpine tar xzf /backup/transmute-backup-YYYY-MM-DD.tar.gz -C /data` |
| Eliminar pila (conserva volumen) | `docker compose down` |
| Eliminar pila y volumen (**pérdida de datos**) | `docker compose down -v` |

## 📝 Licencia

Este proyecto de despliegue Docker se distribuye bajo licencia **MIT**. El código fuente de Transmute es propiedad de sus autores y está licenciado bajo su propia licencia (consulta [repositorio oficial](https://github.com/transmute-app/transmute)).

---

> 📖 **Guía completa y referencia**: [Cómo instalar y configurar Transmute en Docker](https://genbyte.blogspot.com/2026/10/como-instalar-y-configurar-transmute-en_090116431.html) — Genbyte