# 🛠️ Layerling en Docker - CAD sencillo para impresión 3D autohospedado

[![GitHub](https://img.shields.io/badge/GitHub-layerling--docker-blue?logo=github)](https://github.com/henmedia/layerling)
[![Docker](https://img.shields.io/badge/Docker-henmedia%2Flayerling-blue?logo=docker)](https://hub.docker.com/r/henmedia/layerling)
[![License](https://img.shields.io/badge/License-AGPL--3.0-orange)](https://github.com/henmedia/layerling/blob/main/LICENSE)

## 📋 Descripción general

**Layerling** es un editor de CAD para impresión 3D que funciona completamente en el navegador, sin necesidad de cuenta ni registro. Los diseños se guardan localmente en el navegador de cada usuario. Es un fork de **SketchForge-3D** (de Formsmith746) con licencia **AGPL-3.0**.

Esta imagen Docker oficial permite ejecutar tu propia instancia en un NAS, Raspberry Pi o servidor casero, accesible desde cualquier dispositivo de tu red local. Ideal para talleres compartidos, makerspaces o uso personal.

## ✨ Características principales

- **Sin cuenta ni registro**: Abre la página y empieza a diseñar; los diseños viven en el navegador
- **Formas primitivas**: Cajas, cilindros, esferas, conos, pirámides, cuñas, texto, tubos, toros y más
- **Sólidos y agujeros**: Convierte formas en herramientas de corte y agrúpalas para obtener la geometría final
- **Chaflanes y redondeos**: Aplica chaflán o redondeo a cualquier borde de un sólido, y quítalo después si cambias de idea
- **Geometría exacta**: Mantiene formas CAD exactas (motor OCCT vía occt-wasm), no solo mallas
- **Exportación múltiple**: STL y OBJ para el laminador, STEP para pasar a un CAD completo
- **Proyectos `.lyl`**: Guarda el proyecto completo con historial, bocetos y grupos, y ábrelo en otro sitio
- **Modo servidor compartido**: Con una carpeta compartida, los proyectos se guardan en el servidor y toda la red puede abrirlos
- **Multi-arquitectura**: Imagen oficial para `amd64` y `arm64` (Raspberry Pi, NAS ARM)
- **Servidor MCP opcional**: Permite que un cliente de IA controle los editores abiertos (requiere token seguro)

## 📋 Requisitos del sistema

- **Docker** y **Docker Compose** (en sistemas antiguos el comando puede ser `docker-compose`)
- **Arquitectura**: `amd64` o `arm64` (compatible con Raspberry Pi, NAS ARM, servidores x86)
- **Puerto 3000** del contenedor (mapeable a cualquier puerto del host)
- **Opcional**: Una carpeta en el host para la zona de proyectos compartidos (requiere permisos de usuario 999)

## 🐳 Instalación

### Paso 1: Crear `docker-compose.yml`

```yaml
# Basado en el compose de docker/ del repositorio oficial
services:
  layerling:
    image: henmedia/layerling:latest
    container_name: layerling
    ports:
      - "3000:3000"
    environment:
      PORT: "3000"
      HOSTNAME: "0.0.0.0"
      # Desactiva la telemetría de Next.js (1 = desactivada)
      NEXT_TELEMETRY_DISABLED: "1"
      # Color por defecto para quien entra por primera vez: system, light, dark o graphite
      # LAYERLING_DEFAULT_THEME: "graphite"
      # Carpeta compartida de proyectos (ver sección de configuración avanzada)
      # LAYERLING_SHARED_PROJECTS_DIR: "/shared-projects"
    volumes:
      # Vistas previas de la pantalla de inicio: sobreviven a las actualizaciones
      - layerling-thumbnails:/app/.codex/project-thumbnails
      # - ./shared-projects:/shared-projects
    restart: unless-stopped

volumes:
  layerling-thumbnails:
```

> **Nota sobre volúmenes**: El volumen `layerling-thumbnails` es el que recomienda el propio README para las vistas previas, así que aquí se usa un volumen Docker en lugar de una carpeta. El resto de la configuración no usa volúmenes.

### Paso 2: Iniciar Layerling

```bash
docker compose up -d
# Comprueba que el contenedor arranca
docker compose logs -f layerling
```

### Paso 3: Comprobar la versión

```bash
# Muestra la versión de la imagen que está corriendo
docker inspect layerling | grep image.version
```

## ⚙️ Configuración

1. **Variables de entorno principales** (en `docker-compose.yml`):
   - `PORT`: Puerto interno (por defecto `3000`)
   - `HOSTNAME`: Interfaz de escucha (`0.0.0.0` para todas)
   - `NEXT_TELEMETRY_DISABLED`: `"1"` para desactivar telemetría de Next.js
   - `LAYERLING_DEFAULT_THEME`: Tema por defecto (`system`, `light`, `dark`, `graphite`)

2. **Carpeta compartida de proyectos** (opcional):
   - Descomenta `LAYERLING_SHARED_PROJECTS_DIR: "/shared-projects"` en `environment`
   - Descomenta `- ./shared-projects:/shared-projects` en `volumes`
   - El contenedor corre como usuario **999**; ese usuario debe poder escribir en la carpeta:
     ```bash
     sudo chown 999:999 ./shared-projects
     ```
   - Si prefieres que el contenedor corra como tu propio usuario, añade `user: "1000:1000"` con tus números (`id -u` e `id -g`)

3. **Acceso desde proxy inverso** (dominio real):
   - Añade la dirección a `LAYERLING_ALLOWED_ORIGINS` (separadas por comas)
   - Sin esto, el servidor rechaza guardar en la carpeta compartida:
     ```yaml
     LAYERLING_ALLOWED_ORIGINS: "https://layerling.tudominio.com"
     ```

4. **Servidor MCP para asistentes de IA** (opcional, desactivado por defecto):
   - Requiere dos variables y token de al menos 16 caracteres:
     ```yaml
     LAYERLING_MCP_REMOTE: "true"
     LAYERLING_MCP_TOKEN: "un-secreto-largo-y-aleatorio"
     # Solo si accedes con un dominio real detrás de proxy:
     LAYERLING_MCP_ALLOWED_HOSTS: "layerling.tudominio.com"
     ```
   - **Seguridad**: Guarda el token en un fichero `.env` y nunca en el compose ni en un blog. Usa solo en red de confianza y HTTPS si sales a Internet.

5. **Verificar configuración en ejecución**:
   ```bash
   docker exec layerling printenv LAYERLING_SHARED_PROJECTS_DIR
   ```

## 🚀 Primeros pasos

1. Abre la dirección en el navegador:
   - Desde el propio servidor: `http://localhost:3000`
   - Desde otro dispositivo de tu red: `http://IP_DEL_SERVIDOR:3000`
2. No hace falta cuenta. Pulsa en **Nuevo diseño** y arrastra una forma sobre la mesa
3. Convierte una segunda forma en **agujero**, agrúpalas y ajusta las medidas desde el panel lateral
4. Exporta a **STL** para tu laminador desde el menú de exportación (también OBJ y STEP)

## 💡 Casos de uso

- **Piezas a medida**: Soportes, tapas o separadores que tienen que encajar en algo concreto
- **Taller compartido**: Una sola instancia en el NAS para toda la casa o el makerspace
- **Importar lo que ya tienes**: Cargar un STL y construir alrededor
- **Asistente de IA**: Con el servidor MCP activado, un cliente de IA puede trabajar en el editor mientras tú miras cada paso

## 🔒 Acceso remoto seguro

Para exponer Layerling a Internet de forma segura:

1. Usa un **proxy inverso** (Nginx Proxy Manager, Traefik, Caddy) con **HTTPS** y certificado válido (Let's Encrypt)
2. Configura `LAYERLING_ALLOWED_ORIGINS` con tu dominio público
3. Si usas el servidor MCP, **obligatorio** HTTPS y `LAYERLING_MCP_ALLOWED_HOSTS` con tu dominio
4. Considera añadir autenticación adicional en el proxy (Authelia, Authentik, Basic Auth)
5. Limita el acceso a tu red VPN (Tailscale, WireGuard) en lugar de exponer directamente

## 🛠️ Gestión y mantenimiento

### Ver logs
```bash
docker compose logs -f layerling
```

### Actualizar a la última versión
```bash
docker compose pull
docker compose up -d
```
> **Importante**: Tras cambiar variables de entorno, usa `docker compose up -d`. El comando `docker compose restart` conserva la configuración anterior.

### Detener Layerling
```bash
docker compose down
```

### Stack técnico
- **Aplicación**: Web con Next.js sobre Node.js (la imagen usa Node 24 según el Dockerfile)
- **Geometría**: Motor CAD basado en OCCT (occt-wasm), que mantiene formas exactas
- **Almacenamiento**: Diseños en el navegador; proyectos compartidos en carpeta del servidor
- **Imagen Docker**: Oficial para `amd64` y `arm64`, puerto 3000
- **Licencia**: GNU AGPL-3.0 only

## 📝 Licencia

Este proyecto utiliza la imagen oficial de **Layerling** licenciada bajo **GNU AGPL-3.0 only**.
- Repositorio oficial: [henmedia/layerling](https://github.com/henmedia/layerling)
- Licencia: [AGPL-3.0](https://github.com/henmedia/layerling/blob/main/LICENSE)

---

> 📖 **Artículo original**: [Cómo instalar Layerling en Docker - CAD sencillo para impresión 3D autohospedado](https://genbyte.blogspot.com/2026/10/como-instalar-layerling-en-docker-cad.html)  
> 🎥 **Vídeo tutorial**: [Canal GENBYTE en YouTube](https://www.youtube.com/@genbyte)  
> ☕ **Apoya el canal**: [Ko-fi](https://ko-fi.com/genbyte) | [Newsletter](https://genbyte.blogspot.com/) | [Telegram](https://t.me/genbyte) | [Discord](https://discord.gg/genbyte)