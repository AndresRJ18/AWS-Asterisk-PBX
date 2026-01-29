# ☁️ Parte 1: Aprovisionamiento de Infraestructura en AWS

Esta sección detalla la configuración de la red y el servidor en Amazon Web Services (AWS) necesarios para soportar tráfico VoIP (Voz sobre IP) y Video en tiempo real sin latencia ni problemas de NAT.

## 1.1. Lanzamiento de la Instancia EC2
El servidor PBX se aloja en una instancia de computación elástica.

* **Servicio:** Amazon EC2
* **Región:** us-east-1 (N. Virginia) o tu región más cercana.
* **AMI (Imagen de SO):** `Ubuntu Server 24.04 LTS (HVM), SSD Volume Type`.
* **Tipo de Instancia:** `t3.micro` o `t2.micro` (Suficiente para <10 llamadas concurrentes).
* **Almacenamiento:** 10 GB gp3 (General Purpose SSD).
* **Key Pair:** Generar par de llaves `.pem` (RSA) para acceso SSH seguro.

> **Nota:** Se recomienda la familia `t3` sobre la `t2` por su mejor rendimiento en ráfagas (burstable performance), ideal para picos de llamadas.

---

## 1.2. Configuración de Seguridad (Firewall)
La configuración del **Security Group** es el paso más crítico. Se deben abrir puertos específicos para permitir la señalización SIP y el flujo de medios (Audio/Video).

**Reglas de Entrada (Inbound Rules):**

| Tipo | Protocolo | Rango de Puertos | Origen (Source) | Propósito |
| :--- | :--- | :--- | :--- | :--- |
| **SSH** | TCP | `22` | `My IP` | Administración remota segura. |
| **Custom UDP** | **UDP** | `5060` | `0.0.0.0/0` | **SIP Signaling:** Establecimiento de llamadas. |
| **Custom UDP** | **UDP** | `10000 - 20000` | `0.0.0.0/0` | **RTP Media:** Transporte de Audio y Video. |

> ⚠️ **Advertencia:** Es un error común configurar el puerto 5060 o el rango RTP como TCP. VoIP requiere **UDP** para la transmisión en tiempo real.

---

## 1.3. Estabilidad de Red (Elastic IP)
Las instancias EC2 cambian de IP pública al detenerse/iniciarse. Para un servidor de telefonía, la IP debe ser estática.

1. Navegar a **Network & Security** > **Elastic IPs**.
2. Seleccionar **Allocate Elastic IP address**.
3. Seleccionar la IP creada > **Actions** > **Associate Elastic IP address**.
4. Vincular a la instancia `Asterisk-PBX`.

> **Impacto Técnico:** El uso de una Elastic IP simplifica la configuración de NAT en Asterisk (`external_media_address`), evitando el problema de "audio en una sola vía" (one-way audio) cuando la IP pública cambia dinámicamente.
# ⚙️ Parte 2: Instalación y Despliegue de Configuración

Una vez aprovisionada la infraestructura, procedemos a instalar Asterisk y desplegar los archivos de configuración almacenados en este repositorio.

## 2.1. Instalación del Motor Asterisk
Conéctate a tu instancia mediante SSH y ejecuta los siguientes comandos para preparar el entorno:

```bash
# 1. Actualizar repositorios y sistema
sudo apt update && sudo apt upgrade -y

# 2. Instalar Asterisk y dependencias base
sudo apt install asterisk -y

# 3. Habilitar el servicio al inicio
sudo systemctl enable asterisk
```

## 2.2. Despliegue de Archivos (Deployment)
En lugar de editar manualmente, reemplazaremos los archivos por defecto de Asterisk con los templates optimizados de la carpeta configs/ de este proyecto.

  ### A.Clonar este repositorio en el servidor (opcional) o subir los archivos: (Si tienes git instalado en el servidor)

```bash
git clone [https://github.com/TU_USUARIO/aws-asterisk-pbx.git](https://github.com/TU_USUARIO/aws-asterisk-pbx.git)
cd aws-asterisk-pbx
```
  ### B.Reemplazar configuraciones: Haremos un backup de los originales y copiaremos los nuestros.
```bash
# Backup de seguridad
sudo mv /etc/asterisk/pjsip.conf /etc/asterisk/pjsip.conf.bak
sudo mv /etc/asterisk/extensions.conf /etc/asterisk/extensions.conf.bak
sudo mv /etc/asterisk/rtp.conf /etc/asterisk/rtp.conf.bak

# Copiar archivos del repositorio al directorio de Asterisk
sudo cp configs/pjsip.conf /etc/asterisk/
sudo cp configs/extensions.conf /etc/asterisk/
sudo cp configs/rtp.conf /etc/asterisk/
```
## 2.3. Configuración de Variables de Entorno (CRUCIAL)
Debes editar el archivo pjsip.conf para que coincida con tu IP Elástica de AWS.

```bash
sudo nano /etc/asterisk/pjsip.conf
```
Cambios requeridos:

* Busca la línea external_media_address y pon tu Elastic IP.
* Busca la línea external_signaling_address y pon tu Elastic IP.
* (Opcional) Cambia las contraseñas de los usuarios 100 y 101.
  
## 2.4. Reiniciar Servicio
```bash
sudo systemctl restart asterisk
```
# 📱 3. Conexión de Clientes (Softphones)

Para probar las videollamadas, se recomienda usar **Zoiper** (Móvil/PC), **Linphone** o **MicroSIP**.

## 3.1 Datos de Conexión

- **Domain / Host:** Tu Elastic IP (`X.X.X.X`)
- **Username:** `100` o `101`
- **Password:** La definida en `pjsip.conf` (Default: `Andres123`)
- **Transport:** `UDP`

---

## 3.2 Habilitar Video (IMPORTANTE)

Por defecto, la mayoría de softphones tienen el video desactivado.

1. Ir a **Settings → Video**
2. Habilitar **Enable Video**
3. En **Video Codecs**, asegurarse de que **H.264** esté seleccionado y con prioridad alta

---

## 3.3 Pruebas

- **Echo Test:** Marcar `600` para probar latencia y retorno de audio
- **Llamada P2P:** Conectar un dispositivo con el usuario `100` y otro con `101` y realizar una videollamada
