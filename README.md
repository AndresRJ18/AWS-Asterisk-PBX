☁️ AWS Cloud PBX with Asterisk 20 (Audio & Video Support)
Implementación de una Central Telefónica IP (PBX) moderna desplegada en AWS EC2, utilizando Asterisk 20 y el stack PJSIP. Este proyecto soporta llamadas de Audio de Alta Definición y Videollamadas (H.264) a través de Internet, resolviendo desafíos complejos de NAT mediante Elastic IPs.

🛠️ Tech Stack
Cloud Provider: AWS (EC2 Instance, Security Groups, Elastic IP).

Core Engine: Asterisk 20 (LTS).

Protocol: SIP sobre UDP (Driver PJSIP).

Media Support:

Audio: uLaw, aLaw, G.722, GSM.

Video: H.264, VP8 (Pass-through).

OS: Ubuntu Server 24.04 LTS.

🚀 Arquitectura
El sistema está diseñado para superar los problemas comunes de VoIP en la nube (One-way audio):

EC2 Instance: Alojamiento del motor de telefonía.

AWS Elastic IP: Garantiza una dirección pública estática para la señalización SIP correcta.

Security Groups: Configuración de firewall granular (UDP 5060 para señalización, UDP 10000-20000 para RTP).

Endpoint Configuration: Soporte para softphones móviles y de escritorio (Zoiper, Linphone, MicroSIP).

⚙️ Características Clave
Multi-Device Support: Cada extensión (ej. 100) soporta hasta 3 dispositivos simultáneos (max_contacts=3) con lógica de rotación automática.

NAT Traversal: Configuración avanzada de external_media_address y force_rport para evitar pérdida de audio en redes 4G/WiFi.

Video Calls: Habilitado el paso de paquetes de video H.264 para conferencias visuales.