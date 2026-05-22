# Despliegue AWS con GitHub Actions + CloudFormation

Guía completa para desplegar la aplicación automáticamente en AWS usando CI/CD.

---

## Tabla de Contenidos

1. [Arquitectura](#arquitectura)
2. [Prerequisitos](#prerequisitos)
3. [Configurar GitHub Secrets](#paso-1-configurar-github-secrets)
4. [Desplegar Infraestructura](#paso-2-desplegar-infraestructura)
5. [Desplegar Aplicación](#paso-3-desplegar-aplicación)
6. [Destruir Infraestructura](#paso-4-destruir-infraestructura)
7. [Troubleshooting](#troubleshooting)

---

## Arquitectura

<img width="2816" height="1536" alt="diagrama" src="https://github.com/user-attachments/assets/e5211b5c-231d-406a-adbf-2f008a1198f3" />


**Componentes AWS creados:**

| Recurso | Descripción |
|---------|-------------|
| VPC | Red virtual `10.0.0.0/16` |
| Subnet Pública | `10.0.1.0/24` con IPs públicas |
| Internet Gateway | Acceso a internet |
| Route Table | Enrutamiento al IGW |
| 5 Security Groups | Control de acceso por servicio |
| 5 EC2 (t2.micro) | Una por cada servicio |

---

## Prerequisitos

- Cuenta de **AWS Academy Learner Lab** activa
- Repositorio en **GitHub** (público o privado)
- Los 3 workflows ya están en `.github/workflows/`

---

## Paso 1: Configurar GitHub Secrets

### 1.1 Obtener credenciales del Learner Lab

1. Inicia sesión en **AWS Academy**
2. Abre tu **Learner Lab**
3. Haz clic en **"Start Lab"** (botón verde)
4. Espera a que el indicador esté verde
5. Haz clic en **"AWS Details"**
6. En la sección **"AWS CLI"**, haz clic en **"Show"**
7. Copia los 3 valores:
   - `aws_access_key_id`
   - `aws_secret_access_key`
   - `aws_session_token`

### 1.2 Agregar Secrets en GitHub

1. Ve a tu repositorio en GitHub
2. **Settings** → **Secrets and variables** → **Actions**
3. Crea los siguientes secrets:

| Secret Name | Valor |
|------------|-------|
| `AWS_ACCESS_KEY_ID` | El `aws_access_key_id` del lab |
| `AWS_SECRET_ACCESS_KEY` | El `aws_secret_access_key` del lab |
| `AWS_SESSION_TOKEN` | El `aws_session_token` del lab |

> **IMPORTANTE**: Las credenciales del Learner Lab **expiran cada ~4 horas**. 
> Cada vez que reinicies el lab, debes actualizar estos 3 secrets.

---

## Paso 2: Desplegar Infraestructura

1. Ve a la pestaña **"Actions"** de tu repositorio
2. En el panel izquierdo, selecciona **"Deploy Infrastructure"**
3. Haz clic en **"Run workflow"**
4. Selecciona el tipo de instancia (default: `t2.micro`)
5. Haz clic en **"Run workflow"** (botón verde)

**Tiempo estimado**: ~5 minutos

Al terminar, verás las IPs de las 5 instancias EC2 en los logs del workflow.

---

## Paso 3: Desplegar Aplicación

### Opción A: Automático (push a main)

Simplemente haz `git push` a la rama `main`. El workflow se ejecuta automáticamente.

### Opción B: Manual

1. Ve a **Actions** → **"Deploy Application"**
2. Haz clic en **"Run workflow"**

**Tiempo estimado**: ~15-20 minutos (primera vez), ~5-10 minutos (actualizaciones)

Al terminar, accede a `http://<IP-Frontend>` en tu navegador.

---

## Paso 4: Destruir Infraestructura

**IMPORTANTE**: Siempre destruye la infraestructura al terminar para no gastar créditos.

1. Ve a **Actions** → **"Destroy Infrastructure"**
2. Haz clic en **"Run workflow"**
3. Escribe `destroy` en el campo de confirmación
4. Haz clic en **"Run workflow"**

**Tiempo estimado**: ~5 minutos

---

## Troubleshooting

### Error: "Unable to locate credentials"

**Causa**: Los secrets de AWS no están configurados o expiraron.

**Solución**: 
1. Reinicia el Learner Lab
2. Copia las nuevas credenciales
3. Actualiza los 3 secrets en GitHub

### Error: "Key pair does not exist"

**Causa**: El key pair se eliminó manualmente en AWS.

**Solución**: Ejecuta **"Deploy Infrastructure"** de nuevo.

### Error: Timeout esperando instancias

**Causa**: Las instancias tardan en instalar Docker.

**Solución**: Ejecuta **"Deploy Application"** de nuevo manualmente.

### Error: Docker build falla por memoria (OOM)

**Causa**: t2.micro tiene solo 1GB RAM.

**Solución**: 
1. Destruye la infraestructura actual
2. Redesplega con `t2.small` (2GB RAM) seleccionando la opción en el workflow

### Backend no conecta a MySQL

**Causa**: MySQL puede tardar ~30s en estar listo.

**Solución**: Espera 1-2 minutos y refresca el navegador. Si persiste, revisa los logs:
```bash
# Conectar a la EC2 del backend
ssh -i key.pem ec2-user@<IP_BACKEND>

# Ver logs del contenedor
docker logs backend-ventas
```

### Frontend muestra 502 Bad Gateway

**Causa**: Los backends aún no están listos.

**Solución**: Espera a que los backends arranquen (~1-2 min después del deploy).

---

## Estructura de Archivos

```
.github/
  workflows/
    deploy-infra.yml       ← Crea la infraestructura AWS
    deploy-app.yml         ← Despliega la aplicación
    destroy-infra.yml      ← Destruye la infraestructura

infra/
  cloudformation.yml       ← Template de infraestructura
  README-AWS.md            ← Esta documentación

front_despacho/
  nginx.conf               ← Config para Docker Compose (local)
  nginx-aws.conf.template  ← Config para AWS (IPs dinámicas)
```

---

## Flujo de Trabajo Típico

```
1. Iniciar Learner Lab
         ↓
2. Copiar credenciales → GitHub Secrets
         ↓
3. Ejecutar "Deploy Infrastructure" (una vez)
         ↓
4. Ejecutar "Deploy Application" (o push a main)
         ↓
5. Trabajar con la aplicación
         ↓
6. Ejecutar "Destroy Infrastructure" (al terminar)
         ↓
7. Detener Learner Lab
```
