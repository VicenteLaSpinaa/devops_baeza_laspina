# ============================================================
# Dockerfile — Frontend React (Vite + Tailwind)
# ============================================================
# Multi-stage build con 2 etapas:
#   1. BUILD: Instala dependencias y compila el proyecto React
#      con Vite. Genera archivos estáticos en /app/dist.
#   2. PRODUCCIÓN: Copia los estáticos a Nginx Alpine (imagen
#      ultra liviana ~40MB). Configura usuario no-root.
# ============================================================


# ==================== ETAPA 1: BUILD ====================
# Usa Node.js 20 sobre Alpine Linux (imagen liviana ~180MB)
# "AS build" le da nombre a esta etapa para referenciarla después
FROM node:20-alpine AS build

# Establece /app como directorio de trabajo dentro del contenedor.
# Todos los comandos siguientes se ejecutan desde /app.
WORKDIR /app

# -- Optimización de caché de Docker --
# Copia SOLO los archivos de dependencias primero.
# Docker cachea esta capa: si package.json no cambia,
# no reinstala las dependencias en builds futuros.
COPY package.json package-lock.json ./

# Instala las dependencias exactas del lockfile.
# --frozen-lockfile: falla si package-lock.json no coincide
# con package.json (previene inconsistencias).
RUN npm ci --frozen-lockfile

# Ahora copia TODO el código fuente del proyecto.
# Esta capa se invalida cada vez que cambia cualquier archivo,
# pero las dependencias (capa anterior) siguen cacheadas.
COPY . .

# Ejecuta el script "build" definido en package.json.
# Vite compila React + Tailwind y genera archivos estáticos
# optimizados (HTML, CSS, JS minificado) en la carpeta /app/dist.
RUN npm run build


# ==================== ETAPA 2: PRODUCCIÓN ====================
# Usa Nginx sobre Alpine Linux (imagen final ~40MB).
# Solo contiene el servidor web y los archivos estáticos.
# Todo lo de la etapa 1 (Node.js, node_modules, código fuente)
# se descarta automáticamente.
FROM nginx:1.27-alpine AS production

# -- Configuración de usuario no-root (mínimo privilegio) --
# Crear un grupo "nginxgroup" y un usuario "nginxuser" sin
# directorio home (-H), sin shell de login (-s /sbin/nologin),
# y sin contraseña (-D). Este usuario NO puede ejecutar
# comandos como root, reduciendo el riesgo de seguridad.
RUN addgroup -S nginxgroup && \
    adduser -S -G nginxgroup -H -s /sbin/nologin nginxuser

# Elimina la configuración por defecto de Nginx
RUN rm /etc/nginx/conf.d/default.conf

# Copia la configuración personalizada de Nginx que incluye
# el reverse proxy hacia los backends
COPY nginx.conf /etc/nginx/conf.d/default.conf

# -- Copia de archivos estáticos desde la etapa de build --
# COPY --from=build indica que el archivo viene de la etapa 1.
# /app/dist es donde Vite dejó los archivos compilados.
# Se copian a /usr/share/nginx/html, que es la carpeta raíz
# por defecto de Nginx para servir contenido.
COPY --from=build /app/dist /usr/share/nginx/html

# Asigna la propiedad de los archivos al usuario no-root.
# Nginx necesita leer estos archivos, y el usuario nginxuser
# necesita permisos de lectura sobre ellos.
RUN chown -R nginxuser:nginxgroup /usr/share/nginx/html && \
    chown -R nginxuser:nginxgroup /var/cache/nginx && \
    chown -R nginxuser:nginxgroup /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginxuser:nginxgroup /var/run/nginx.pid

# A partir de aquí, todos los comandos se ejecutan como
# nginxuser en lugar de root.
USER nginxuser

# Documenta que el contenedor escucha en el puerto 80.
# Esto NO abre el puerto, solo es metadata para docker-compose.
EXPOSE 80

# Comando que se ejecuta al iniciar el contenedor.
# "daemon off" hace que Nginx se ejecute en primer plano
# (foreground), lo cual es obligatorio en Docker porque
# si el proceso principal termina, el contenedor se detiene.
CMD ["nginx", "-g", "daemon off;"]
