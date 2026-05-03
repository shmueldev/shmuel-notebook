# Documentación de Despliegue en AWS: Data Platform (Airflow + Celery) y Nginx Proxy Manager

Este documento detalla el paso a paso para desplegar la arquitectura distribuida (Master, Workers, RabbitMQ, DB y NPM) en Amazon Web Services (AWS).

## Fase 1: Redes y Seguridad (Networking)

1. **Creación de VPC**
   - Crear una Virtual Private Cloud (VPC) dedicada al proyecto.

2. **Creación de Subredes (Subnets)**
   - **Públicas**: Para Nginx Proxy Manager y NAT Gateway.
   - **Privadas**: Para Airflow Master, Workers, RDS (PostgreSQL) y RabbitMQ.

3. **Creación de Internet Gateway (IGW)**
   - Crear y adjuntar el IGW a la VPC para permitir el acceso desde/hacia el exterior en las subredes públicas.

4. **Creación de NAT Gateway**
   - Desplegar en la subred pública asignando una Elastic IP.
   - Necesario para que las instancias en subredes privadas (Workers, Master) puedan descargar imágenes de Docker o dependencias sin ser expuestas a internet.

5. **Creación de Tablas de Enrutamiento (Route Tables)**
   - **Tabla Pública**: Ruta `0.0.0.0/0` apuntando al Internet Gateway.
   - **Tabla Privada**: Ruta `0.0.0.0/0` apuntando al NAT Gateway.

6. **Creación de Grupos de Seguridad (Security Groups)**
   - **SG-Proxy**: Permitir tráfico HTTP (80), HTTPS (443) y el puerto de admin de NPM (81) desde `0.0.0.0/0`, SSH.
   - **SG-Airflow-Master**: Permitir puerto 8080 desde `0.0.0.0/0`, puerto 5555 desde `SG-Proxy` y SSH.
   - **SG-Airflow-Workers**: Permitir tráfico interno desde el Master.
   - **SG-RabbitMQ**: Permitir puerto 5672 para `SG-Airflow-Master` y `SG-Airflow-Workers` y 15672 a `0.0.0.0/0`.

## Fase 2: Lanzar instancias EC2

7. **Lanzar 3 instancias workers**
   - privadas
   - c7i-flex.large

8. **Lanzar 1 instancia master**
   - privadas
   - m7i-flex.large

9. **Lanzar 1 instancia rabbitmq**
   - privadas
   - t3.micro

10. **Lanzar 1 instancia proxy-manager**   
   - públicas
   - t3.micro
