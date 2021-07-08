---
title: 'Inicio rápido: WildFly en CentOS'
description: Implementación de aplicaciones Java en WildFly en una máquina virtual de CentOS
author: Theresa-Nguyen
ms.author: bicnguy
ms.date: 10/23/2020
ms.assetid: 7aa21ef8-9cfb-43e0-bfda-3f10a2a2f3ef
ms.topic: quickstart
ms.service: virtual-machines
ms.subservice: redhat
ms.custom:
- mode-api
ms.collection: linux
ms.openlocfilehash: 7c80634e78a8c37551a57fc8597831898646ef8e
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111900942"
---
# <a name="quickstart-wildfly-on-centos-8"></a>Inicio rápido: WildFly en CentOS 8

En esta guía de inicio rápido se muestra cómo implementar el nodo independiente de WildFly de la máquina virtual de CentOS 8. Es perfecto para el desarrollo y las pruebas de aplicaciones Java empresariales en Azure. La suscripción de servidor de aplicaciones no es necesaria para implementar este inicio rápido.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. Si no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o [crear una cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

## <a name="use-case"></a>Caso de uso

WildFly es perfecto para el desarrollo y las pruebas de aplicaciones Java empresariales en Azure. Las listas de tecnologías disponibles en los perfiles de configuración del servidor de WildFly 18 están disponibles en la [guía de introducción de WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#getting-started-with-wildfly).

Puede usar WildFly en modo independiente o de clúster por su caso de uso. Puede garantizar la alta disponibilidad de las aplicaciones críticas de Yakarta EE por WildFly en un clúster de nodos. Realice un pequeño número de cambios en la configuración de la aplicación y, a continuación, implemente la aplicación en el clúster. Para más información al respecto, consulte la [guía de alta disponibilidad de WildFly](https://docs.wildfly.org/18/High_Availability_Guide.html).

## <a name="configuration-choice"></a>Opciones de configuración

WildFly se puede iniciar en modo de **servidor independiente**: una instancia de servidor independiente es un proceso independiente, como una instancia del servidor de aplicaciones JBoss (AS) 3, 4, 5 o 6. Las instancias independientes se pueden iniciar mediante los scripts de inicio standalone.sh o standalone.bat. En el caso de más de una instancia independiente, es responsabilidad del usuario coordinar la administración de varios servidores en todos los servidores.

También puede iniciar la instancia de WildFly con una configuración alternativa mediante el uso de archivos de configuración disponibles en la carpeta de configuración.

A continuación se muestran los archivos de configuración del servidor independiente:

- standalone.xml (valor predeterminado): esta configuración es el archivo predeterminado que se usa para iniciar la instancia de WildFly. Contiene la configuración certificada de un perfil web de Yakarta con las tecnologías necesarias.
   
- standalone-ha.xml: configuración certificada del perfil 8 web de Yakarta EE con alta disponibilidad (destinada a aplicaciones web).
   
- standalone-full.xml: configuración certificada de la plataforma 8 de Yakarta EE, incluidas todas las tecnologías necesarias para hospedar aplicaciones de Yakarta EE.

- standalone-full-ha.xml: configuración certificada de la plataforma 8 de Jakarta EE con alta disponibilidad para hospedar aplicaciones de Yakarta EE.

Para iniciar el servidor de WildFly independiente con otra configuración proporcionada, use el argumento --server-config con el archivo de configuración del servidor.

Por ejemplo, para usar la plataforma 8 de Yakarta EE con capacidades de agrupación en clústeres, use el siguiente comando:

```
./standalone.sh --server-config=standalone-full-ha.xml
```

Para más información acerca de las configuraciones, consulte la [guía de introducción de WildFly](https://docs.wildfly.org/18/Getting_Started_Guide.html#wildfly-10-configurations).

## <a name="licensing-support-and-subscription-notes"></a>Notas sobre licencias, soporte técnico y suscripciones

La imagen de Azure CentOS 8 es una imagen de máquina virtual de pago por uso y no requiere que el usuario obtenga una licencia. La primera vez que se inicia la máquina virtual, la licencia del sistema operativo de la máquina virtual se activará automáticamente, y se aplicará una tarifa por hora. Esto se aplica a las tarifas por hora de las máquinas virtuales Linux de Microsoft. Consulte [Precios de máquinas virtuales Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#linux) para más información. La descarga y el uso de WildFly son gratuitos, y no requiere una suscripción o una licencia de Red Hat.

## <a name="how-to-consume"></a>Procedimiento para el consumo

Puede implementar la plantilla de las tres maneras siguientes:

- Uso de PowerShell: implemente la plantilla mediante la ejecución de los siguientes comandos: (Consulte [Azure PowerShell](/powershell/azure/) para obtener información sobre la instalación y configuración de Azure PowerShell).

    ```
    New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/wildfly/wildfly-standalone-centos8/azuredeploy.json
    ```
    
- Uso de la CLI de Azure: implemente la plantilla mediante la ejecución de los siguientes comandos: (Consulte la [línea de comandos multiplataforma de Azure](/cli/azure/install-azure-cli) para más información sobre la instalación y configuración de la interfaz de la línea de comandos multiplataforma de Azure).

    ```
    az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new Resource Group for your deployment
    ```

    ```
    az deployment group create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/wildfly/wildfly-standalone-centos8/azuredeploy.json
    ```

- Uso de Azure Portal: implemente la plantilla haciendo clic <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fapplication-workloads%2Fwildfly%2Fwildfly-standalone-centos8%2Fazuredeploy.json" target="_blank">aquí</a> e inicie sesión en Azure Portal.

## <a name="arm-template"></a>Plantilla ARM

<a href="https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/wildfly/wildfly-standalone-centos8" target="_blank"> WildFly 18 en CentOS 8 (máquina virtual independiente)</a>: se trata de una plantilla de inicio rápido que crea un nodo independiente de WildFly 18.0.1.Final en una máquina virtual de CentOS 8 en el grupo de recursos que incluye una dirección IP privada para la máquina virtual, red virtual y cuenta de almacenamiento de diagnósticos. También implementa una aplicación Java de ejemplo denominada JBoss-EAP en Azure en WildFly.

## <a name="resource-links"></a>Vínculos de recursos

* Más información sobre [WildFly 18](https://docs.wildfly.org/18/)
* Obtenga más información sobre [Distribuciones de Linux aprobadas en Azure](../../linux/endorsed-distros.md).
* [Documentación para desarrolladores de Azure para Java](https://github.com/JasonFreeberg/jboss-on-app-service)

## <a name="next-steps"></a>Pasos siguientes

Para el entorno de producción, consulte las plantillas de Resource Manager de inicio rápido de JBoss EAP de Red Hat de Azure:

Máquina virtual RHEL independiente con aplicación de ejemplo:

*  <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-standalone-rhel" target="_blank"> JBoss EAP en RHEL (máquina virtual independiente)</a>

Máquinas virtuales RHEL en clúster con aplicación de ejemplo:

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-multivm-rhel" target="_blank"> JBoss EAP en RHEL (máquinas virtuales en clúster)</a>

Conjunto de escalado de máquinas virtuales RHEL en clúster con aplicación de ejemplo:

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/jboss-eap-clustered-vmss-rhel" target="_blank"> JBoss EAP en RHEL (conjunto de escalado de máquinas virtuales en clúster)</a>
