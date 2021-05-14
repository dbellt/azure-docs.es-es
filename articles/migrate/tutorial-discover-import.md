---
title: Evaluación de los servidores locales mediante un archivo CSV importado con Azure Migrate Assessment Server
description: Describe cómo detectar servidores locales para la migración a Azure mediante un archivo CSV importado en Azure Migrate Server Assessment
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.openlocfilehash: c142cae3e96d800488b67da613181d1a91ba5b5b
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107713325"
---
# <a name="tutorial-assess-servers-using-an-imported-csv-file"></a>Tutorial: Evaluación de los servidores mediante un archivo CSV importado

Como parte del recorrido de la migración a Azure, puede detectar el inventario y las cargas de trabajo locales. 

En este tutorial se muestra cómo evaluar máquinas locales con la herramienta Azure Migrate Server Assessment, mediante un archivo de valores separados por comas (CSV) importado. 

Si usa un archivo CSV, no es necesario configurar el dispositivo Azure Migrate para detectar y evaluar los servidores. Puede controlar los datos que comparte en el archivo. Muchos son opcionales. Este método es útil si:

- Desea crear una evaluación inicial rápida antes de implementar el dispositivo.
- No se puede implementar el dispositivo de Azure Migrate en la organización.
- No se pueden compartir credenciales que permitan el acceso a servidores locales.
- Las restricciones de seguridad impiden la recopilación y el envío de datos recopilados por el dispositivo a Azure.

> [!NOTE]
> No se puede migrar servidores importados mediante un archivo CSV.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Configuración de una cuenta de Azure
> * Configuración de un proyecto de Azure Migrate
> * Preparación de un archivo CSV
> * Importe el archivo.
> * Evaluar servidores

> [!NOTE]
> Los tutoriales muestran la manera más rápida de probar un escenario y utilizar las opciones predeterminadas siempre que sea posible. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

- Puede agregar hasta 20 000 servidores en un solo archivo CSV y en un proyecto de Azure Migrate. 
- Los nombres de sistema operativo especificados en el archivo CSV deben contener y coincidir con [nombres admitidos](#supported-operating-system-names).


## <a name="prepare-an-azure-user-account"></a>Preparación de una cuenta de usuario de Azure

Para crear un proyecto de Azure Migrate, necesita una cuenta con:
- Permisos de nivel de colaborador o propietario en una suscripción de Azure.
- Permisos para registrar aplicaciones de Azure Active Directory.

Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción. Si no es el propietario, trabaje con él para asignar los permisos, como se indica a continuación:

1. En Azure Portal, busque "suscripciones" y, en **Servicios**, seleccione **Suscripciones**.

    ![Cuadro para buscar la suscripción de Azure](./media/tutorial-discover-import/search-subscription.png)

2. En la página **Suscripciones**, seleccione aquella en la que desee crear un proyecto de Azure Migrate. 
3. En la suscripción, seleccione **Access control (IAM)**  > **Comprobar acceso**.
4. En **Comprobar acceso**, busque la cuenta de usuario correspondiente.
5. En **Agregar una asignación de roles**, seleccione **Agregar**.

    ![Búsqueda de una cuenta de usuario para comprobar el acceso y asignar un rol](./media/tutorial-discover-import/azure-account-access.png)

6. En **Agregar asignación de roles**, seleccione el rol Colaborador o Propietario, y seleccione la cuenta (azmigrateuser en nuestro ejemplo). Después, seleccione **Guardar**.

    ![Se abre la página Agregar asignación de roles para asignar un rol a la cuenta.](./media/tutorial-discover-import/assign-role.png)

7. En el portal, busque los usuarios y, en **Servicios**, seleccione **Usuarios**.
8. En **Configuración de usuario**, compruebe que los usuarios de Azure AD puedan registrar aplicaciones (establecido en **Sí** de forma predeterminada).

    ![Comprobación en la configuración de usuario de que los usuarios puedan registrar aplicaciones de Active Directory](./media/tutorial-discover-import/register-apps.png)



## <a name="set-up-a-project"></a>Configuración de un proyecto

Configure un nuevo proyecto de Azure Migrate si no tiene uno.

1. En Azure Portal > **Todos los servicios**, busque **Azure Migrate**.
2. En **Servicios**, seleccione **Azure Migrate**.
3. En **Información general**, seleccione **Crear proyecto**.
5. En **Crear proyecto**, seleccione su suscripción y grupo de recursos de Azure. Cree un grupo de recursos si no tiene ninguno.
6. En **Detalles del proyecto**, especifique el nombre del proyecto y la región geográfica en la que quiere crearlo. Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

   ![Cuadros de nombre de proyecto y región](./media/tutorial-discover-import/new-project.png)  
    > [!Note]
    > Use la sección de configuración **avanzada** para crear un proyecto de Azure Migrate con conectividad de punto de conexión privado. [Más información](how-to-use-azure-migrate-with-private-endpoints.md#create-a-project-with-private-endpoint-connectivity)

7. Seleccione **Crear**.
8. Espere unos minutos para que se implemente el proyecto de Azure Migrate.

La herramienta **Azure Migrate: Server Assessment** se agrega de forma predeterminada al nuevo proyecto.

![Página que muestra la herramienta Server Assessment agregada de forma predeterminada](./media/tutorial-discover-import/added-tool.png)

## <a name="prepare-the-csv"></a>Preparación del archivo CSV

Descargue la plantilla de CSV y agréguele información de servidor.

### <a name="download-the-template"></a>Descarga de la plantilla

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, seleccione **Detectar**.
2. En **Detectar máquinas**, seleccione **Importar mediante CSV**.
3. Seleccione **Descargar** para descargar la plantilla CSV. O bien, puede [descargarla directamente](https://go.microsoft.com/fwlink/?linkid=2109031).

    ![Descarga de la plantilla CSV](./media/tutorial-discover-import/download-template.png)

### <a name="add-server-information"></a>Incorporación de información del servidor

Recopile los datos del servidor y agréguelos al archivo CSV.

- Para recopilar los datos, puede exportarlos desde herramientas que se usan para la administración de servidores locales, como VMware vSphere o la base de datos de administración de configuración (CMDB).
- Para revisar los datos de ejemplo, descargue el [archivo de ejemplo](https://go.microsoft.com/fwlink/?linkid=2108405).

En la tabla siguiente se resumen los campos de archivo que se van a rellenar:

**Nombre del campo** | **Obligatorio** | **Detalles**
--- | --- | ---
**Nombre del servidor** | Sí | Es recomendable especificar el nombre de dominio completo (FQDN).
**Dirección IP** | No | Dirección del servidor.
**Núcleos** | Sí | Número de núcleos de procesador asignados al servidor.
**Memoria** | Sí | RAM total (en MB) asignada al servidor.
**OS name** (Nombre de SO) | Sí | Sistema operativo del servidor. <br/> La valoración reconoce los nombres del sistema operativo que coinciden con los nombres de [esta](#supported-operating-system-names) lista, o que los contienen.
**Versión del sistema operativo** | No | Versión del sistema operativo del servidor.
**Arquitectura del sistema operativo** | No | Arquitectura del sistema operativo del servidor <br/> Los valores válidos son: x64, x86, amd64, 32 bits o 64 bits
**Número de discos** | No | No es necesario si se proporcionan detalles del disco individual.
**Disk 1 size** (Tamaño de disco 1)  | No | Tamaño máximo del disco (en GB).<br/>Puede agregar detalles de más discos mediante la [adición de columnas](#add-multiple-disks) en la plantilla. Puede agregar hasta ocho discos.
**Disk 1 read ops** (Operaciones de lectura de disco 1) | No | Operaciones de lectura de disco por segundo.
**Disk 1 write ops** (Operaciones de escritura de disco 1) | No | Operaciones de escritura de discos por segundo.
**Disk 1 read throughput** (Rendimiento de lectura de disco 1) | No | Datos leídos del disco por segundo (en MB por segundo).
**Disk 1 write throughput** (Rendimiento de escritura de disco 1) | No | Datos escritos en el disco por segundo (en MB por segundo).
**CPU utilization percentage** (Porcentaje de uso de la CPU) | No | Porcentaje de CPU en uso.
**Memory utilization percentage** (Porcentaje de uso de la memoria) | No | Porcentaje de RAM en uso.
**Total disks read ops** (Total de operaciones de lectura de discos) | No | Operaciones de lectura de disco por segundo.
**Total disks write ops** (Total de operaciones de escritura de discos) | No | Operaciones de escritura de discos por segundo.
**Total disks read throughput** (Total de rendimiento de lectura de discos) | No | Datos leídos del disco (en MB por segundo).
**Total disks write throughput** (Total de rendimiento de escritura de discos) | No | Datos escritos en el disco (en MB por segundo).
**Network in throughput** (Rendimiento de entrada de red) | No | Datos recibidos por el servidor (en MB por segundo).
**Network out throughput** (Rendimiento de salida de red) | No | Datos transmitidos por el servidor (en MB por segundo).
**Firmware type** (Tipo de firmware) | No | Firmware del servidor. Los valores pueden ser "BIOS" o "UEFI".
**MAC address** (Dirección MAC)| No | Dirección MAC de servidor.


### <a name="add-operating-systems"></a>Incorporación de sistemas operativos

La evaluación reconoce nombres específicos del sistema operativo. Cualquier nombre que especifique debe coincidir con una de las cadenas de la [lista de nombres admitidos](#supported-operating-system-names).

### <a name="add-multiple-disks"></a>Agregar varios discos

La plantilla proporciona los campos predeterminados para el primer disco. Puede agregar columnas similares para un máximo de ocho discos.

Por ejemplo, para especificar todos los campos de un segundo disco, agregue estas columnas:

- Tamaño de disco 2
- Operaciones de lectura de disco 2
- Operaciones de escritura de disco 2
- Rendimiento de lectura de disco 2
- Rendimiento de escritura de disco 2


## <a name="import-the-server-information"></a>Importación de la información de servidor

Después de agregar información a la plantilla de CSV, importe el archivo CSV en Server Assessment.

1. En Azure Migrate, en **Detectar máquinas**, vaya a la plantilla rellena.
2. Seleccione **Import** (Importar).
3. Se muestra el estado de la importación.
    - Si aparecen advertencias en el estado, puede corregirlas o continuar.
    - Para mejorar la precisión de la evaluación, mejore la información del servidor como se sugiere en las advertencias.
    - Para ver y corregir advertencias, seleccione **Download warning details .CSV** (Descargar .CSV de detalles de advertencias). Esta operación descarga el archivo CSV con las advertencias. Revíselas y corrija los problemas según sea necesario.
    - Si aparecen errores de estado (que el estado de la importación sea **Failed**), debe corregirlos para poder continuar con la importación:
        1. Descargue el archivo CSV, que ahora incluye los detalles del error.
        1. Revise y solucione los errores según sea necesario. 
        1. Vuelva a cargar el archivo modificado.
4. Si el estado de la importación es **Completed**, esto significa que la información del servidor se ha importado. Actualice si el proceso de importación no parece estar completo.

## <a name="update-server-information"></a>Actualización de la información del servidor

Para actualizar la información de un servidor, vuelva a importar los datos para el servidor con el mismo valor para **Nombre de servidor**. El campo **Nombre de servidor** no se puede modificar. La eliminación de servidores no se admite actualmente.

## <a name="verify-servers-in-the-portal"></a>Comprobación de los servidores en el portal

Para comprobar que los servidores aparecen en Azure Portal tras la detección:

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Assessment**, seleccione el icono que muestra el número de **Servidores detectados**.
3. Seleccione la pestaña **Import based** (Basado en la importación).



## <a name="supported-operating-system-names"></a>Nombres de los sistemas operativos compatibles

Los nombres de sistema operativo que se proporcionan en el CSV deben contener y coincidir. Si no es así, no podrá evaluarlos. 

**A - H** | **I - R** | **S-T** | **U-Z**
--- | --- | --- | ---
Asianux 3<br/>Asianux 4<br/>Asianux 5<br/>CentOS<br/>CentOS 4/5<br/>CoreOS Linux<br/>Debian GNU/Linux 4<br/>Debian GNU/Linux 5<br/>Debian GNU/Linux 6<br/>Debian GNU/Linux 7<br/>Debian GNU/Linux 8<br/>FreeBSD | IBM OS/2<br/>macOS X 10<br/>MS-DOS<br/>Novell NetWare 5<br/>Novell NetWare 6<br/>Oracle Linux<br/>Oracle Linux 4/5<br/>Oracle Solaris 10<br/>Oracle Solaris 11<br/>Red Hat Enterprise Linux 2<br/>Red Hat Enterprise Linux 3<br/>Red Hat Enterprise Linux 4<br/>Red Hat Enterprise Linux 5<br/>Red Hat Enterprise Linux 6<br/>Red Hat Enterprise Linux 7<br/>Red Hat Fedora | SCO OpenServer 5<br/>SCO OpenServer 6<br/>SCO UnixWare 7<br/> Serenity Systems eComStation 1<br/>Serenity Systems eComStation <br/>Sun Microsystems Solaris 8<br/>Sun Microsystems Solaris 9<br/><br/>SUSE Linux Enterprise 10<br/>SUSE Linux Enterprise 11<br/>SUSE Linux Enterprise 12<br/>SUSE Linux Enterprise 8/9<br/>SUSE Linux Enterprise 11<br/>SUSE openSUSE | Ubuntu Linux<br/>VMware ESXi 4<br/>VMware ESXi 5<br/>VMware ESXi 6<br/>Windows 10<br/>Windows 2000<br/>Windows 3<br/>Windows 7<br/>Windows 8<br/>Windows 95<br/>Windows 98<br/>Windows NT<br/>Windows Server (R) 2008<br/>Windows Server 2003<br/>Windows Server 2008<br/>Windows Server 2008 R2<br/>Windows Server 2012<br/>Windows Server 2012 R2<br/>Windows Server 2016<br/>Windows Server 2019<br/>Windows Server Threshold<br/>Windows Vista<br/>Windows Web Server 2008 R2<br/>Windows XP Professional

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Ha creado un proyecto de Azure Migrate 
> * Servidores detectados mediante un archivo CSV importado. Ahora, ejecute una evaluación de la [migración de máquinas virtuales de VMware a máquinas virtuales de Azure](./tutorial-assess-vmware-azure-vm.md).