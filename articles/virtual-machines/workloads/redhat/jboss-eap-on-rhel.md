---
title: 'Inicio rápido: Implementación de JBoss Enterprise Application Platform (EAP) con Red Hat Enterprise Linux (RHEL) en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure'
description: Cómo implementar aplicaciones empresariales Java con Red Hat JBoss EAP en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure con RHEL.
author: theresa-nguyen
ms.author: bicnguy
ms.date: 10/30/2020
ms.assetid: 8a4df7bf-be49-4198-800e-db381cda98f5
ms.topic: quickstart
ms.service: virtual-machines
ms.subservice: redhat
ms.custom:
- mode-api
ms.collection: linux
ms.openlocfilehash: 18cf093adad858f50b2b1fa1c97e38821bd1c949
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538674"
---
# <a name="deploy-enterprise-java-applications-to-azure-with-jboss-eap-on-red-hat-enterprise-linux"></a>Implementación de aplicaciones empresariales Java en Azure con JBoss EAP en Red Hat Enterprise Linux

En las plantillas de inicio rápido de Azure de este artículo se muestra cómo implementar [JBoss Enterprise Application Platform (EAP)](https://www.redhat.com/en/technologies/jboss-middleware/application-platform) con [Red Hat Enterprise Linux (RHEL)](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) en máquinas virtuales de Azure y conjuntos de escalado de máquinas virtuales de Azure. Usará una aplicación Java de ejemplo para validar la implementación. 

JBoss EAP es una plataforma de servidor de aplicaciones de código abierto. Ofrece seguridad, escalabilidad y rendimiento de nivel empresarial para las aplicaciones Java. RHEL es una plataforma de sistema operativo de código abierto. Permite el escalado de aplicaciones existentes y la implementación de nuevas tecnologías en todos los entornos. 

JBoss EAP y RHEL incluyen todo lo necesario para compilar, ejecutar, implementar y administrar aplicaciones empresariales Java en cualquier entorno. Se trata de una solución de código abierto tanto para entornos locales y virtuales como para nubes privadas, públicas o híbridas.

## <a name="prerequisites"></a>Requisitos previos 

* Una cuenta de Azure con una suscripción activa. Para obtener una suscripción de Azure, active sus [créditos de Azure para suscriptores de Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o [cree una cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

* Instalación JBoss EAP. Debe tener una cuenta de Red Hat con derecho de Red Hat Subscription Management (RHSM) para JBoss EAP. Este derecho le permitirá descargar la versión de JBoss EAP probada y certificada por Red Hat.  

  Si no tiene el derecho de EAP, obtenga una [suscripción de evaluación de JBoss EAP](https://access.redhat.com/products/red-hat-jboss-enterprise-application-platform/evaluation) antes de empezar. Para crear una nueva suscripción de Red Hat, vaya al [portal de clientes de Red Hat](https://access.redhat.com/) y configure una cuenta.

* La[CLI de Azure](/cli/azure/overview).

* Opciones de RHEL. Elija el modelo de pago por uso (PAYG) o traiga su propia suscripción (BYOS). Con el modelo BYOS, debe activar su imagen Gold de RHEL de [Red Hat Cloud Access](https://access.redhat.com/) antes de implementar la plantilla de inicio rápido.

## <a name="java-ee-and-jakarta-ee-application-migration"></a>Migración de aplicaciones de Java EE y Jakarta EE

### <a name="migrate-to-jboss-eap"></a>Migración a JBoss EAP
JBoss EAP 7.2 y 7.3 son implementaciones certificadas de las especificaciones Java Enterprise Edition (Java EE) 8 y Jakarta EE 8. JBoss EAP proporciona opciones preconfiguradas para características como clústeres de alta disponibilidad (HA), mensajería y almacenamiento en caché distribuido. También permite a los usuarios escribir, implementar y ejecutar aplicaciones con las distintas API y servicios que proporciona JBoss EAP.  

Para más información sobre JBoss EAP, consulte [Introducción a JBoss EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html-single/introduction_to_jboss_eap/index) o [Introducción a JBoss EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/introduction_to_jboss_eap/index).

 #### <a name="applications-on-jboss-eap"></a>Aplicaciones en JBoss EAP

* **Aplicaciones de servicios web**. Los servicios web proporcionan una manera estándar de interoperar a las aplicaciones de software. Cada aplicación puede ejecutarse en plataformas y marcos diferentes. Estos servicios web facilitan la comunicación de subsistemas internos y heterogéneos. 

  Para más información, consulte [Desarrollo de aplicaciones de servicios web en EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/developing_web_services_applications/index) o [Desarrollo de aplicaciones de servicios web en EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/developing_web_services_applications/index).

* **Aplicaciones Enterprise Java Beans (EJB)** . EJB 3.2 es una API para el desarrollo de aplicaciones Java EE y Jakarta EE distribuidas, transaccionales, seguras y portátiles. EJB usa componentes del lado servidor denominados beans empresariales para implementar la lógica de negocios de una aplicación de forma desacoplada a fin de facilitar la reutilización. 

  Para más información, visite [Desarrollo de aplicaciones EJB en EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/developing_ejb_applications/index) o [Desarrollo de aplicaciones EJB en EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/developing_ejb_applications/index).

* **Aplicaciones Hibernate**. Los desarrolladores y administradores pueden desarrollar e implementar aplicaciones Java Persistence API (JPA) e Hibernate con JBoss EAP. Hibernate Core es un marco de asignación relacional de objetos para el lenguaje Java. Permite asignar un modelo de dominio orientado a objetos a una base de datos relacional, de manera que las aplicaciones pueden evitar la interacción directa con la base de datos. 

  El administrador de entidades de Hibernate implementa las interfaces de programación y las reglas del ciclo de vida tal y como se define en la [especificación de JPA 2.1](https://www.jcp.org/en/jsr/overview). Junto con las anotaciones de Hibernate, este contenedor implementa una solución JPA completa (e independiente) sobre la instancia madura de Hibernate Core. 
  
  Para más información sobre Hibernate, visite [JPA en EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/development_guide/java_persistence_api) o [Jakarta Persistence en EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/development_guide/java_persistence_api).

#### <a name="red-hat-migration-toolkit-for-applications"></a>Red Hat Migration Toolkit for Applications
El [kit de herramientas de migración para aplicaciones (MTA) de Red Hat](https://developers.redhat.com/products/mta/overview) es una herramienta de migración para servidores de aplicaciones Java. Use esta herramienta para migrar desde otro servidor de aplicaciones a JBoss EAP. Funciona con complementos de IDE para [Eclipse IDE](https://www.eclipse.org/ide/), [Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview) y [Visual Studio Code](https://code.visualstudio.com/docs/languages/java) para Java. 

MTA es una herramienta gratuita y de código abierto que hace lo siguiente:
* Automatizar el análisis de aplicaciones.
* Admitir la estimación de esfuerzo.
* Acelerar la migración de código.
* Admitir la contenedorización.
* Integrarse con Azure Workload Builder.

### <a name="migrate-jboss-eap-from-on-premises-to-azure"></a>Migración de JBoss EAP desde un entorno local a Azure
La oferta de Azure Marketplace de JBoss EAP en RHEL se instalará y aprovisionará en máquinas virtuales de Azure en menos de 20 minutos. Puede acceder a estas ofertas desde [Azure Marketplace](https://azuremarketplace.microsoft.com/).

Esta oferta de Azure Marketplace incluye varias combinaciones de versiones de EAP y RHEL para satisfacer sus requisitos. JBoss EAP siempre utiliza el modelo BYOS, pero para el sistema operativo RHEL puede elegir entre BYOS o PAYG. La oferta de Azure Marketplace incluye opciones de planes para JBoss EAP en RHEL, como máquinas virtuales independientes o en clúster:

* JBoss EAP 7.2 en máquina virtual de RHEL 7.7 (PAYG)
* JBoss EAP 7.2 en máquina virtual de RHEL 8.0 (PAYG)
* JBoss EAP 7.3 en máquina virtual de RHEL 8.0 (PAYG)
* JBoss EAP 7.2 en máquina virtual de RHEL 7.7 (BYOS)
* JBoss EAP 7.2 en máquina virtual de RHEL 8.0 (BYOS)
* JBoss EAP 7.3 en máquina virtual de RHEL 8.0 (BYOS)

Junto con las ofertas de Azure Marketplace, puede usar plantillas de inicio rápido disponibles para iniciar su recorrido de migración a Azure. Estos inicios rápidos incluyen plantillas y scripts pregenerados de Azure Resource Manager (ARM) para implementar JBoss EAP en RHEL en diversas configuraciones y combinaciones de versiones. Tendrá lo siguiente:

* Un equilibrador de carga.
* Una dirección IP privada para el equilibrio de carga y las máquinas virtuales.
* Una red virtual con una única subred.
* La configuración de máquinas virtuales (en clúster o independientes).
* Una aplicación Java de ejemplo.

La arquitectura de la solución para estas plantillas incluye:

* JBoss EAP en una máquina virtual de RHEL independiente.
* JBoss EAP en clúster en varias máquinas virtuales de RHEL.
* JBoss EAP en clúster mediante Azure Virtual Machine Scale Sets.

#### <a name="linux-workload-migration-for-jboss-eap"></a>Migración de la carga de trabajo de Linux para JBoss EAP
Azure Workload Builder simplifica el proceso de prueba de concepto, evaluación y migración para aplicaciones Java locales en Azure. Workload Builder se integra con la herramienta de detección de Azure Migrate para identificar servidores de JBoss EAP. A continuación, generará dinámicamente un cuaderno de estrategias de Ansible para la implementación de servidores de JBoss EAP. Utiliza la herramienta MTA de Red Hat para migrar servidores desde otros servidores de aplicaciones a JBoss EAP. 

Entre los pasos para simplificar la migración se incluyen los siguientes:
1. **Evaluación**. Evaluación de los clústeres de JBoss EAP mediante una máquina virtual o un conjunto de escalado de máquinas virtuales de Azure.
1. **Valoración**. Examen de las aplicaciones y la infraestructura.
1. **Configuración de la infraestructura**. Creación de un grupo de cargas de trabajo.
1. **Implementación y pruebas**. Implementación, migración y pruebe de la carga de trabajo.
1. **Configuración posterior a la implementación**. Integración con los datos, la supervisión, la seguridad, la copia de seguridad, etc.

## <a name="server-configuration-choice"></a>Opciones de configuración del servidor

Para la implementación de la máquina virtual de RHEL, puede elegir PAYG o BYOS. Las imágenes de [Azure Marketplace](https://azuremarketplace.microsoft.com) utilizan de forma predeterminada PAYG. Implemente una máquina virtual de RHEL de tipo BYOS si tiene su propia imagen de sistema operativo RHEL. Asegúrese de que su cuenta de RHSM tiene derecho de BYOS mediante Cloud Access antes de implementar la máquina virtual o el conjunto de escalado de máquinas virtuales.

JBoss EAP tiene una potente funcionalidad de administración, además de proporcionar funciones y API a las aplicaciones. Esta funcionalidad de administración puede variar en función del modo de funcionamiento usado para iniciar JBoss EAP. Es compatible con RHEL y Windows Server. JBoss EAP ofrece un modo de funcionamiento de servidor independiente para administrar instancias discretas. También ofrece un modo de funcionamiento de dominio administrado para administrar grupos de instancias desde un único punto de control. 

> [!NOTE]
> Los dominios administrados por JBoss EAP no son compatibles con Microsoft Azure porque los servicios de infraestructura de Azure administran la característica de alta disponibilidad. 

La variable de entorno `EAP_HOME` indica la ruta de acceso a la instalación de JBoss EAP. Use el siguiente comando para iniciar el servicio EAP JBoss en modo independiente:

```
$EAP_HOME/bin/standalone.sh
```
    
Este script de inicio usa el archivo EAP_HOME/bin/Standalone.conf para establecer algunas preferencias predeterminadas, como las opciones de JVM. Puede personalizar la configuración de este archivo. JBoss EAP usa el archivo de configuración standalone.xml para iniciarse en modo independiente de forma predeterminada, pero se puede usar otro modo para iniciarlo. 

Para más información sobre los archivos de configuración de servidor independiente disponibles y cómo usarlos, consulte [Archivos de configuración de servidor independiente para EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuration_guide/jboss_eap_management#standalone_server_configuration_files) o [Archivos de configuración de servidor independiente para EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuration_guide/jboss_eap_management#standalone_server_configuration_files). 

Para iniciar JBoss EAP con una configuración diferente, use el argumento `--server-config`. Por ejemplo:
    
 ```
 $EAP_HOME/bin/standalone.sh --server-config=standalone-full.xml
 ```
    
Para obtener una lista completa de todos los argumentos de script de inicio disponibles y su finalidad, use el argumento `--help`. Para obtener más información, vea [Argumentos de tiempo de ejecución de servidor en EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuration_guide/reference_material#reference_of_switches_and_arguments_to_pass_at_server_runtime) o [Argumentos de tiempo de ejecución de servidor en EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuration_guide/reference_material#reference_of_switches_and_arguments_to_pass_at_server_runtime).
    
JBoss EAP también puede funcionar en modo de clúster. La mensajería en clústeres de JBoss EAP permite la agrupación de servidores de mensajería de JBoss EAP para compartir la carga de procesamiento de mensajes. Cada nodo activo del clúster es un servidor de mensajería de JBoss EAP activo, que administra sus propios mensajes y controla sus propias conexiones. Para más información, consulte [Introducción a los clústeres en EAP 7.2](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/configuring_messaging/clusters_overview) o [Introducción a los clústeres en EAP 7.3](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html/configuring_messaging/clusters_overview). 

## <a name="support-and-subscription-notes"></a>Compatibilidad y notas de suscripción
Estas plantillas de inicio rápido se ofrecen como se indica a continuación: 

- El sistema operativo RHEL se ofrece como PAYG o BYOS mediante el modelo de imagen de Red Hat Gold.
- JBoss EAP se ofrece solo como BYOS.

#### <a name="using-rhel-os-with-the-payg-model"></a>Uso del sistema operativo RHEL con el modelo PAYG

De forma predeterminada, estas plantillas de inicio rápido usan la imagen de PAYG de RHEL 7.7 u 8.0 a petición de Azure Marketplace. Las imágenes de PAYG tienen un cargo de suscripción de RHEL adicional por hora sobre los costos de proceso, red y almacenamiento habituales. Al mismo tiempo, la instancia se registrará en la suscripción de Red Hat. Esto significa que utilizará uno de sus derechos. 

Esta imagen de PAYG dará lugar a una "doble facturación". Puede evitar este problema si crea su propia imagen de RHEL. Para más información, lea este artículo de la Base de conocimiento de Red Hat sobre [cómo aprovisionar una máquina virtual de RHEL para Microsoft Azure](https://access.redhat.com/articles/uploading-rhel-image-to-azure). También puede activar su imagen Gold de RHEL de [Red Hat Cloud Access](https://access.redhat.com/).

Para más información sobre los precios de máquinas virtuales con el modelo PAYG, consulte [Precios de máquinas virtuales Red Hat](https://azure.microsoft.com/pricing/details/virtual-machines/red-hat/). Para usar RHEL con el modelo PAYG, necesitará una suscripción de Azure con el método de pago especificado para [RHEL 7.7 en Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux77-ARM) o [RHEL 8.0 en Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux80-ARM). Estas ofertas requieren que se especifique un método de pago en la suscripción de Azure.

#### <a name="using-rhel-os-with-the-byos-model"></a>Uso del sistema operativo RHEL con el modelo BYOS

Para usar el modelo BYOS para el sistema operativo de RHEL, debe tener una suscripción válida de Red Hat con derechos para usar el sistema operativo RHEL en Azure. Antes de implementar el sistema operativo RHEL con el modelo BYOS, complete los siguientes requisitos previos:

1. Asegúrese de que tiene los derechos del sistema operativo RHEL y JBoss EAP asociados a su suscripción de Red Hat.
2. Autorice su identificador de suscripción de Azure para usar imágenes de BYOS en RHEL. Siga la [documentación de administración de suscripciones de Red Hat](https://access.redhat.com/documentation/red_hat_subscription_management/1/) para completar el proceso, que incluye estos pasos:

   1. Habilite Microsoft Azure como proveedor en el panel de Red Hat Cloud Access.

   1. Agregue sus identificadores de suscripción de Azure.

   1. Habilite nuevos productos para Cloud Access en Microsoft Azure.
    
   1. Active imágenes de Red Hat Gold para su suscripción de Azure. Para más información, consulte las [imágenes de Red Hat Gold en Microsoft Azure](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/cloud-access-gold-images_cloud-access#proc_using-gold-images-azure_cloud-access).

   1. Espere a que las imágenes de Red Hat Gold estén disponibles en su suscripción de Azure. Estas imágenes suelen estar disponibles en el plazo de 3 horas desde el envío.
    
3. Acepte los términos y condiciones de Azure Marketplace para las imágenes BYOS de RHEL. Para completar este proceso, ejecute los siguientes comandos de la CLI de Azure. Para más información, consulte la documentación de las [imágenes Gold de tipo BYOS de RHEL en Azure](./byos.md). Es importante que ejecute la versión más reciente de la CLI de Azure.

   1. Inicie una sesión de la CLI de Azure y autentíquese con su cuenta de Azure. Para obtener ayuda, consulte el [inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

   1. Compruebe que las imágenes BYOS de RHEL están disponibles en su suscripción mediante la ejecución del siguiente comando de la CLI. Si no obtiene ningún resultado, asegúrese de que su suscripción de Azure está activada para imágenes BYOS de RHEL.
   
      ```
      az vm image list --offer rhel-byos --all
      ```

   1. Ejecute el siguiente comando para aceptar los términos de Marketplace para BYOS de RHEL 7,7 y BYOS de RHEL 8.0, respectivamente:
      ```
      az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm77
      ``` 

      ```
      az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm8
      ``` 
   
La suscripción ya estará lista para implementar BYOS en RHEL 7.7 u 8.0 en máquinas virtuales de Azure.

#### <a name="using-jboss-eap-with-the-byos-model"></a>Uso de JBoss EAP con el modelo BYOS

JBoss EAP solo está disponible en Azure mediante el modelo BYOS. Al implementar esta plantilla, debe proporcionar sus credenciales de RHSM junto con el identificador de grupo de RHSM con derechos de EAP válidos. Si no tiene el derecho de EAP, obtenga una [suscripción de evaluación de JBoss EAP](https://access.redhat.com/products/red-hat-jboss-enterprise-application-platform/evaluation) antes de empezar.

## <a name="deployment-options"></a>Opciones de implementación

Puede implementar la plantilla de las siguientes maneras:

- **PowerShell**. Ejecute los siguientes comandos para implementa la plantilla: 

  ```
  New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  ```

  ```
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri <raw link to the template which can be obtained from github>
  ```
 
  Para obtener información sobre la instalación y configuración de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).  

- **Azure CLI**. Ejecute los siguientes comandos para implementa la plantilla:

  ```
  az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  ```

  ```
  az deployment group create --resource-group <my-resource-group> --template-uri <raw link to the template which can be obtained from github>
  ```

  Para más información sobre la instalación y configuración de la CLI de Azure, consulte la [instalación de la CLI](/cli/azure/install-azure-cli).

- **Azure Portal**. Para realizar la implementación en Azure Portal, acceda a las plantillas de inicio rápido de Azure, tal y como se indica en la sección siguiente. En el inicio rápido, seleccione **Implementar en Azure** o el botón **Explorar en GitHub**.

## <a name="azure-quickstart-templates"></a>Plantillas de inicio rápido de Azure

Para empezar, use una de las siguientes plantillas de inicio rápido para JBoss EAP en RHEL siempre que cumpla su objetivo de implementación:

* <a href="https://azure.microsoft.com/resources/templates/jboss-eap-standalone-rhel/"> JBoss EAP en RHEL (máquina virtual independiente)</a>. Esta plantilla implementará una aplicación web denominada JBoss-EAP en Azure para JBoss EAP 7.2 o 7.3 que se ejecuta en la máquina virtual de RHEL 7.7 u 8.0.

* <a href="https://azure.microsoft.com/resources/templates/jboss-eap-clustered-multivm-rhel/"> JBoss EAP en RHEL (en clúster, varias máquinas virtuales)</a>. Esta plantilla implementará una aplicación web denominada eap-session-replication en un clúster de JBoss EAP 7.2 o 7.3 que se ejecuta en un número *n* de máquinas virtuales de RHEL 7.7 u 8.0. El usuario decide el valor *n*. Todas las máquinas virtuales se agregan al grupo de back-end de un equilibrador de carga.

* <a href="https://azure.microsoft.com/en-us/resources/templates/jboss-eap-clustered-vmss-rhel/"> JBoss EAP en RHEL (en clúster, conjunto de escalado de máquinas virtuales)</a>. Esta plantilla implementará una aplicación web denominada eap-session-replication en un clúster de JBoss EAP 7.2 o 7.3 que se ejecuta en conjuntos de escalado de máquinas virtuales de RHEL 7.7 o 8.0.

## <a name="resource-links"></a>Vínculos de recursos

* [Ventaja híbrida de Azure](../../windows/hybrid-use-benefit-licensing.md)
* [Configuración de una aplicación Java para Azure App Service](../../../app-service/configure-language-java.md)
* [JBoss EAP en Red Hat OpenShift en Azure](https://azure.microsoft.com/services/openshift/)
* [JBoss EAP en Azure App Service (Linux)](../../../app-service/quickstart-java.md)
* [Implementación de JBoss EAP en Azure App Service](https://github.com/JasonFreeberg/jboss-on-app-service)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [JBoss EAP 7.2](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/).
* Más información sobre [JBoss EAP 7.3](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.3/).
* Más información sobre la [administración de suscripciones de Red Hat](https://access.redhat.com/products/red-hat-subscription-management).
* Información sobre las [cargas de trabajo de Red Hat en Azure](./overview.md).
* Implementación de [JBoss EAP en una máquina virtual o un conjunto de escalado de máquinas virtuales de RHEL desde Azure Marketplace](https://aka.ms/AMP-JBoss-EAP).
* Implementación de [JBoss EAP en una máquina virtual o un conjunto de escalado de máquinas virtuales de RHEL desde plantilla de inicio rápido de Azure](https://aka.ms/Quickstart-JBoss-EAP).
