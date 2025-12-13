# Infraestructura y CI/CD para la API de Catálogo EV4 Transformacion Digital

Este repositorio contiene la configuración de la infraestructura y el pipeline de Integración y Despliegue Continuo (CI/CD) para la **API de Catálogo en Laravel**.

El objetivo principal es automatizar el análisis de calidad del código y el despliegue de la aplicación en un entorno de AWS EC2 mediante el uso de GitHub Actions y Ansible.

## Componentes Principales

-   **.github/workflows/deploy.yml**: Archivo principal que define el pipeline de CI/CD. Orquesta el análisis de código con SonarQube y el despliegue con Ansible.
-   **ansible/playbook.yml**: Playbook de Ansible que contiene el conjunto de tareas para configurar el servidor remoto y desplegar la última versión de la API.
-   **ansible/inventory.ini**: Archivo de inventario de Ansible, generado dinámicamente por el workflow para apuntar al servidor de producción.

## Flujo de Trabajo CI/CD (`deploy.yml`)

El pipeline se activa automáticamente con cada `push` a la rama `main` y consta de dos fases secuenciales:

### 1. Job de Análisis de Código (`sonarqube`)

Esta fase actúa como un control de calidad automatizado para el código de la API.

-   **Acción**: Clona el repositorio de la aplicación (`Api_Laravel_Ev4`).
-   **Análisis**: Ejecuta un escaneo con SonarQube para analizar el código en busca de bugs, vulnerabilidades y malas prácticas ("code smells").
-   **Validación**: Compara los resultados con un "Quality Gate" predefinido. Si el código no cumple con los estándares de calidad, el job falla y el pipeline se detiene, impidiendo el despliegue.

### 2. Job de Despliegue (`deploy`)

Esta fase se encarga de poner la aplicación en producción, pero solo se ejecuta si el análisis de SonarQube fue exitoso.

-   **Preparación**: Instala Ansible y configura las credenciales SSH para acceder de forma segura a la instancia EC2.
-   **Ejecución**: Lanza el `playbook.yml` de Ansible, que realiza las siguientes tareas en el servidor:
    1.  Navega al directorio de la aplicación.
    2.  Actualiza el código fuente desde el repositorio (`git pull`).
    3.  Instala las dependencias de PHP (`composer install`).
    4.  Ejecuta las migraciones de la base de datos (`php artisan migrate`).
    5.  Optimiza la aplicación limpiando la caché.
-   **Verificación**: Al finalizar, realiza una prueba de conexión (`curl`) para asegurar que la API está en línea y funcionando correctamente.

## Configuración Requerida

Para que el pipeline funcione, es necesario configurar los siguientes "Secrets" en los ajustes del repositorio de GitHub:

-   `SERVER_IP`: La dirección IP pública de la instancia EC2.
-   `SSH_PRIVATE_KEY`: La clave SSH privada para acceder a la instancia EC2.
-   `SONAR_TOKEN`: El token de autenticación para enviar los resultados del análisis a SonarQube/SonarCloud.

## Tecnologías Utilizadas

-   **GitHub Actions**: Para la orquestación del pipeline de CI/CD.
-   **Ansible**: Para la automatización de la configuración y el despliegue en el servidor.
-   **SonarQube**: Para el análisis estático y el aseguramiento de la calidad del código.
-   **AWS EC2**: Como plataforma de hosting para la aplicación.
.