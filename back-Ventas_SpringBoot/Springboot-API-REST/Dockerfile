# ============================================================
# Dockerfile — Backend Ventas (Spring Boot + Maven)
# ============================================================
# Multi-stage build con 2 etapas:
#   1. BUILD: Usa Maven con JDK 17 para compilar el proyecto
#      Spring Boot y generar el archivo .jar ejecutable.
#   2. PRODUCCIÓN: Usa JRE 17 Alpine (solo el runtime de Java,
#      sin compilador ni herramientas de desarrollo). Imagen
#      final ~200MB vs ~800MB con JDK completo.
# ============================================================


# ==================== ETAPA 1: BUILD ====================
# Maven 3.9.9 con Eclipse Temurin JDK 17 sobre Alpine Linux.
# Esta imagen incluye Maven + JDK completo para compilar.
FROM maven:3.9.9-eclipse-temurin-17-alpine AS build

# Directorio de trabajo dentro del contenedor
WORKDIR /app

# -- Optimización de caché de Docker --
# Copia SOLO el archivo pom.xml primero.
# Docker cachea esta capa: si pom.xml no cambia,
# Maven no re-descarga las dependencias en builds futuros.
COPY pom.xml .

# Descarga TODAS las dependencias declaradas en pom.xml
# sin compilar el código. Esto crea una capa cacheada
# con las dependencias (~200MB de librerías .jar).
# -B = modo batch (sin output interactivo).
RUN mvn dependency:go-offline -B

# Ahora copia TODO el código fuente del proyecto.
# Esta capa se invalida cuando cambia cualquier archivo .java,
# pero las dependencias (capa anterior) permanecen cacheadas.
COPY src ./src

# Compila el proyecto y genera el .jar ejecutable.
# -DskipTests: omite la ejecución de tests unitarios
# (los tests se ejecutan en el pipeline de CI/CD, no aquí).
# El .jar resultante queda en /app/target/
RUN mvn clean package -DskipTests -B


# ==================== ETAPA 2: PRODUCCIÓN ====================
# Eclipse Temurin JRE 17 sobre Alpine Linux.
# JRE = Java Runtime Environment. Solo contiene lo necesario
# para EJECUTAR aplicaciones Java. No incluye el compilador
# (javac), Maven, ni herramientas de desarrollo.
# Resultado: imagen final ~200MB en vez de ~800MB.
FROM eclipse-temurin:17-jre-alpine AS production

# Directorio de trabajo para la aplicación
WORKDIR /app

# -- Configuración de usuario no-root (mínimo privilegio) --
# Crea un grupo "appgroup" y un usuario "appuser" del sistema
# (-S) sin directorio home (-H) y sin shell de login.
# Este usuario NO es root, por lo que no puede:
#   - Instalar paquetes
#   - Modificar archivos del sistema
#   - Escalar privilegios
RUN addgroup -S appgroup && \
    adduser -S -G appgroup -H -s /sbin/nologin appuser

# -- Copia del .jar desde la etapa de build --
# COPY --from=build toma archivos de la etapa 1.
# El .jar es un archivo ejecutable que contiene toda la
# aplicación Spring Boot empaquetada (código + dependencias).
# Se renombra a app.jar para simplificar el comando de inicio.
COPY --from=build /app/target/*.jar app.jar

# Asigna la propiedad del .jar al usuario no-root.
RUN chown appuser:appgroup app.jar

# A partir de aquí, todo se ejecuta como appuser (no root).
USER appuser

# Documenta que el contenedor escucha en el puerto 8080.
# Spring Boot usa este puerto por defecto.
EXPOSE 8080

# Comando de inicio del contenedor.
# java -jar app.jar ejecuta la aplicación Spring Boot.
# Spring Boot levanta un servidor Tomcat embebido que
# escucha en el puerto configurado en application.properties.
ENTRYPOINT ["java", "-jar", "app.jar"]
