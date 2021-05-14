---
title: 'Azure VMware Solution by CloudSimple: configuración de vCenter en la nube privada para vRealize Automation'
description: Describe cómo configurar un servidor de VMware vCenter en la nube privada de CloudSimple como punto de conexión para VMware vRealize Automation.
author: shortpatti
ms.author: v-patsho
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 765e7616c5bea838ba093fb64e9b7e12e651ec69
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108182415"
---
# <a name="set-up-vcenter-on-your-private-cloud-for-vmware-vrealize-automation"></a>Configuración de vCenter en la nube privada para VMware vRealize Automation

Puede configurar un servidor de VMware vCenter en la nube privada de CloudSimple como punto de conexión de VMware vRealize Automation.

## <a name="before-you-begin"></a>Antes de empezar

Complete estas tareas antes de configurar el servidor vCenter Server:

* Configure una [conexión VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre el entorno local y la nube privada.
* [Configure el reenvío de DNS de las solicitudes DNS locales](on-premises-dns-setup.md) a los servidores DNS de la nube privada.
* Envíe una [solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) para crear un usuario administrativo de IaaS de vRealize Automation con el conjunto de permisos que figuran en la siguiente tabla.

| Valor del atributo | Permiso |
------------ | ------------- |  
| Almacén de datos |  Asignar espacio <br> Examinar un almacén de datos |
| Clúster de almacén de datos | Configurar un clúster de almacén de datos |
| Carpeta | Creación de la carpeta <br>Eliminar carpeta |
| Global |  Administrar atributos personalizados<br>Establecer un atributo personalizado |
| Red | Asignar una red |
| Permisos | Modificar permisos |
| Resource | Asignar una máquina virtual a un grupo de recursos<br>Migrar una máquina virtual apagada<br>Migrar una máquina virtual encendida |
| Inventario de máquinas virtuales |  Crear a partir de un elemento existente<br>Crear nuevo<br>Move<br>Remove | 
| Interacción de máquina virtual |  Configurar medio de CD<br>Interacción de la consola<br>Conexión del dispositivo<br>Apagar<br>Encender<br>Reset<br>Suspender<br>Instalación de herramientas | 
| Configuración de máquina virtual |  Agregar disco existente<br>Agregar disco nuevo<br>Agregar o quitar<br>Quitar disco<br>Avanzado<br>Cambiar cantidad de CPU<br>Cambiar recurso<br>Extender disco virtual<br>Seguimiento de cambios de disco<br>Memoria<br>Modificar configuración de dispositivo<br>Cambiar nombre<br>Establecer anotación (versión 5.0 y posterior)<br>Configuración<br>Ubicación del archivo de intercambio |
| Aprovisionamiento |  Personalizar<br>Clonar plantilla<br>Clonar máquina virtual<br>Implementar plantilla<br>Leer especificaciones de personalización |
| Estado de la máquina virtual | Crear instantánea<br>Quitar instantánea<br>Revertir a instantánea |

## <a name="install-vrealize-automation-in-your-on-premises-environment"></a>Instalación de vRealize Automation en el entorno local

1. Inicie sesión en el dispositivo de servidor IaaS de vRealize Automation con la cuenta de usuario administrador de IaaS que CloudSimple creó automáticamente.
2. Implemente un agente de vSphere para el punto de conexión de vRealize Automation.
    1. Vaya a https://*url-vra*:5480/installer, donde *url-vra* es la dirección URL que usa para acceder a la interfaz de usuario de administración de vRealize Automation.
    2. Haga clic en **IaaS Installer** (Instalador de IaaS) para descargar el instalador.<br>
    La convención de nomenclatura del archivo del instalador es setup_ *url-vra*@5480.exe.
    3. Ejecute al programa de instalación. En la pantalla de bienvenida, haga clic en **Siguiente**.
    4. Acepte el contrato de licencia de usuario final y haga clic en **Next** (Siguiente).
    5. Proporcione la información de inicio de sesión, haga clic en **Accept Certificate** (Aceptar certificado) y, después, haga clic en **Next** (Siguiente).
    ![Credenciales de vRA](media/configure-vra-endpoint-login.png)
    6. Seleccione **Custom Install** (Instalación personalizada) y **Proxy Agents** (Agentes proxy) y haga clic en **Next** (Siguiente).
    ![Tipo de instalación de vRA](media/configure-vra-endpoint-install-type.png)
    7. Escriba la información de inicio de sesión del servidor IaaS y haga clic en **Next** (Siguiente). Si usa Active Directory, escriba el nombre de usuario con el formato **dominio\usuario**. Si no, use el formato **user@domain** .
    ![Información de inicio de sesión de vRA](media/configure-vra-endpoint-account.png)
    8. En la configuración de proxy, escriba **vSphere** en **Agent type** (Tipo de agente). Escriba un nombre para el agente.
    9. Escriba el FQDN del servidor IaaS en los campos **Manager Service Host** (Host del servicio de administrador) y **Model Manager Web Service Host** (Host del servicio web del administrador de modelos). Haga clic en **Test** (Probar) para comprobar la conexión de cada uno de los valores de FQDN. Si la prueba no se supera, modifique la configuración de DNS para que se resuelva el nombre de host del servidor IaaS.
    10. Indique un nombre para el punto de conexión del servidor vCenter Server para la nube privada. Anote el nombre para usarlo más adelante en el proceso de configuración.

        ![Proxy de instalación de vRA](media/configure-vra-endpoint-proxy.png)

    11. Haga clic en **Next**.
    12. Haga clic en **Instalar**.

## <a name="configure-the-vsphere-agent"></a>Configuración del agente de vSphere

1. Vaya a https://*url-vra*/vcac e inicie sesión como **ConfigurationAdmin**.
2. Seleccione **Infrastructure** (Infraestructura)  > **Endpoints** (Puntos de conexión)  > **Endpoints** (Puntos de conexión).
3. Seleccione **New** (Nuevo)  > **Virtual** > **vSphere**.
4. Escriba el nombre del punto de conexión de vSphere que especificó en el procedimiento anterior.
5. En **Address** (Dirección), escriba la dirección URL de vCenter Server de la nube privada con el formato https://*fqdn-vcenter*/sdk, donde *fqdn-vcenter* es el nombre del servidor vCenter Server.
6. Escriba las credenciales de usuario administrador de IaaS de vRealize Automation que CloudSimple creó automáticamente.
7. Seleccione **Test Connection** (Probar conexión) para validar la configuración. Si la prueba no se supera, compruebe la dirección URL, la información de la cuenta y el [nombre del punto de conexión](#verify-the-endpoint-name) y vuelva a realizar la prueba.
8. Tras superar la prueba, haga clic en **OK** (Aceptar) para crear el punto de conexión de vSphere.
    ![Acceso a la configuración de punto de conexión de vRA](media/configure-vra-endpoint-vra-edit.png)

### <a name="verify-the-endpoint-name"></a>Comprobación del nombre del punto de conexión

Haga lo siguiente para identificar el nombre de punto de conexión del servidor vCenter Server correcto:

1. Abra un símbolo del sistema en el dispositivo IaaS.
2. Cambie el directorio a c:\Archivos de programa (x86) \VMware\vCAC\Agents\nombre-agente, donde *nombre-agente* es el nombre asignado al punto de conexión del servidor vCenter Server.
3. Ejecute el siguiente comando:

```
..\..\Server\DynamicOps.Vrm.VRMencrypt.exe VRMAgent.exe.config get
```

La salida será similar a la siguiente. El valor del campo `managementEndpointName` es el nombre del punto de conexión.

```
managementEndpointName: cslab1pc3-vc
doDeletes: true
```
