## Objetivo del Playbook

El objetivo de este playbook de Ansible es automatizar la implementación de múltiples máquinas virtuales (VMs) en un entorno de Hyper-V utilizando Windows Server 2022 Core Data Center. Este proceso incluye la configuración de VMs con características específicas como memoria dinámica, discos de diferenciación, configuración de dominio activo, infraestructura de clave pública (PKI) con certificados de autoridad raíz y subordinada, y emisión de certificados para máquinas del dominio.

## Uso del Playbook

### Requisitos Previos

- Asegúrate de tener un entorno de Ansible configurado y funcionando correctamente con acceso a los hosts de Hyper-V.
- Descarga la imagen ISO de Windows Server 2022 Core Data Center y guárdala en una ruta accesible para el playbook.

### Configuración de Variables

Antes de ejecutar el playbook, configura las variables en el archivo YAML según sea necesario:

```yaml
domain_name: "example.local"
domain_admin_user: "Admin"
domain_admin_password: "Password123!"
ca_root_validity_years: 50
sub_ca_validity_years: 10
windows_server_iso_url: "https://path.to/Windows_Server_2022.iso"
parent_disk_path: "C:\\path\\to\\parent_disk.vhdx"
all_vms:
  - name: "RootCA"
    role: "root_ca"
    disk_path: "C:\\path\\to\\rootca_diff_disk.vhdx"
  - name: "SubCA"
    role: "sub_ca"
    disk_path: "C:\\path\\to\\subca_diff_disk.vhdx"
  - name: "DomainControllerVM"
    role: "domain_controller"
    disk_path: "C:\\path\\to\\dc_diff_disk.vhdx"
  - name: "WebServerVM"
    role: "webserver"
    disk_path: "C:\\path\\to\\webserver_diff_disk.vhdx"
  - name: "DatabaseVM"
    role: "database"
    disk_path: "C:\\path\\to\\database_diff_disk.vhdx"
  - name: "AppServerVM"
    role: "appserver"
    disk_path: "C:\\path\\to\\appserver_diff_disk.vhdx"

### Ejecución del Playbook

Ejecuta el playbook desde la línea de comandos de Ansible:

```bash
ansible-playbook playbook.yml
```

El playbook procederá a descargar la ISO de Windows Server 2022, crear una VM base, configurar los discos de diferenciación y luego te solicitará que elijas qué VMs específicas deseas crear en tu entorno.

### Descripción Detallada de las Tareas del Playbook

1. **Descarga de ISO de Windows Server 2022:**
   - Descarga la imagen ISO de Windows Server 2022 desde la URL especificada y la guarda en la ubicación local.

2. **Creación de VM Base para Windows Server 2022:**
   - Crea una máquina virtual base en Hyper-V con los recursos especificados (CPU, memoria, tamaño de disco) utilizando la imagen ISO descargada.

3. **Inicio de VM Base y Preparación:**
   - Inicia la VM base para prepararla para la configuración inicial y espera a que esté lista para continuar.

4. **Creación de Disco Padre desde VM Base:**
   - Detiene la VM base, exporta el disco virtual de la máquina base como un disco padre que servirá de base para los discos de diferenciación de las otras VMs.

5. **Interacción para Selección de VMs:**
   - Pausa el playbook y solicita al usuario que seleccione las VMs específicas que desea crear en el entorno.

6. **Filtrado y Configuración de VMs:**
   - Filtra las VMs seleccionadas según la entrada del usuario y configura las características de cada una (CPU, memoria, discos de diferenciación).

7. **Inicio de VMs Seleccionadas:**
   - Inicia todas las VMs seleccionadas para que estén disponibles en el entorno de Hyper-V.

8. **Configuración de Autoridad de Certificación Raíz (Root CA):**
   - Instala y configura ADCS para establecer una autoridad de certificación raíz, genera el certificado y exporta la clave pública para su uso posterior en la infraestructura de PKI.

9. **Configuración de Controlador de Dominio (Domain Controller):**
   - Instala los servicios de directorio activo (AD DS), promociona la VM a un controlador de dominio, configurando los parámetros necesarios como el nombre de dominio, contraseñas de administrador y contraseña de modo seguro.

10. **Espera de Preparación del Controlador de Dominio:**
    - Espera a que el controlador de dominio esté completamente configurado y listo para aceptar conexiones.

11. **Configuración de Autoridad de Certificación Subordinada (Sub CA):**
    - Instala ADCS en la VM designada como Sub CA, une la VM al dominio, solicita y emite un certificado de Sub CA desde la Autoridad de Certificación Raíz, e importa el certificado para su uso.

12. **Emisión de Certificados para Máquinas del Dominio:**
    - Emite certificados para las máquinas del dominio como servidores web, bases de datos y aplicaciones utilizando la infraestructura de PKI establecida.

## Consideraciones Adicionales

- **Seguridad y Acceso:** Asegúrate de que las credenciales de administrador utilizadas en las tareas de WinRM estén correctamente configuradas y sean seguras.
- **Validación de Certificados:** Puedes ajustar la configuración de validación de certificados en las conexiones WinRM según las políticas de seguridad de tu entorno.
- **Monitoreo y Resolución de Problemas:** Supervisa el progreso del playbook y resuelve cualquier problema que pueda surgir durante la implementación de las VMs y la configuración del dominio.

Este playbook proporciona una manera eficiente y reproducible de implementar un entorno de Windows Server 2022 en Hyper-V con configuraciones específicas de infraestructura de clave pública y dominio activo, ideal para entornos empresariales que requieren alta disponibilidad y seguridad en sus sistemas de TI.
