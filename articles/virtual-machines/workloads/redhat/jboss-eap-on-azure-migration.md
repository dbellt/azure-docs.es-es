---
title: Guía de migración de conjuntos de escalado de máquinas virtuales de JBoss EAP a Azure Virtual Machines
description: En esta guía se proporciona información sobre cómo migrar las aplicaciones empresariales Java desde otro servidor de aplicaciones a JBoss EAP y desde un servidor local tradicional a máquinas virtuales y a conjuntos de escalado de máquinas virtuales de RHEL de Azure.
author: theresa-nguyen
ms.author: bicnguy
ms.topic: article
ms.service: virtual-machines
ms.subservice: redhat
ms.assetid: 9b37b2c4-5927-4271-85c7-19adf33d838b
ms.date: 06/08/2021
ms.openlocfilehash: deded6927998daa5956cff2f63a0a53d3b973b6d
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111855132"
---
# <a name="how-to-migrate-java-applications-to-jboss-eap-on-azure-vms-and-virtual-machine-scale-sets"></a>Cómo migrar aplicaciones Java a JBoss EAP en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure

En esta guía se proporciona información sobre cómo migrar las aplicaciones empresariales Java en la [plataforma de aplicación empresarial (EAP) de Red Hat JBoss](https://www.redhat.com/en/technologies/jboss-middleware/application-platform) desde un servidor local tradicional a máquinas virtuales y conjuntos de escalado de máquinas virtuales de Red Hat Enterprise Linux (RHEL) de Azure si la estrategia de la nube es migrar mediante lift-and-shift las aplicaciones Java tal y como están. Sin embargo, si quiere migrar mediante lift-and-optimize, puede migrar las aplicaciones contenedorizadas a [Red Hat OpenShift en Azure (ARO)](https://azure.microsoft.com/services/openshift/) con imágenes de EAP de JBoss de la galería de Red Hat, o bien colocar el código de la aplicación Java directamente en una EAP de JBoss en una instancia de Azure App Service.

## <a name="best-practice-starting-with-azure-marketplace-offers-and-quickstarts"></a>Procedimiento recomendado para comenzar con ofertas de Azure Marketplace e inicios rápidos

Red Hat y Microsoft se han asociado para incorporar un conjunto de [ofertas de JBoss EAP en Azure Marketplace](https://aka.ms/AMP-JBoss-EAP) y ofrecer un punto inicial sólido para la migración a Azure. Consulte la documentación para obtener la lista de ofertas y planeamientos, y elija la que mejor se ajuste a su implementación existente. Consulte el artículo sobre [JBoss EAP en Procedimientos recomendados de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/redhat/jboss-eap-azure-best-practices/)

Si ninguna de las ofertas existentes es un buen punto inicial, puede reproducir manualmente la implementación mediante máquinas virtuales de Azure y otros recursos disponibles. Para más información, consulte [¿Qué es IaaS?](https://azure.microsoft.com/overview/what-is-iaas/)

### <a name="azure-marketplace-offers"></a>Ofertas de Azure Marketplace

Red Hat en colaboración con Microsoft ha publicado las siguientes ofertas en Azure Marketplace. Puede acceder a estas ofertas en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/) o en [Azure Portal](https://azure.microsoft.com/features/azure-portal/). Consulte el artículo sobre cómo [implementar JBoss EAP de Red Hat en máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure mediante la oferta de Azure Marketplace](https://docs.microsoft.com/azure/virtual-machines/workloads/redhat/jboss-eap-azure-marketplace/) para obtener más detalles.

Esta oferta de Marketplace incluye varias combinaciones de versiones de JBoss EAP y RHEL con modelos de suscripción de soporte flexible. JBoss EAP está disponible en el modelo "Traiga su propia suscripción" (BYOS), pero para RHEL puede elegir entre BYOS o Pago por uso (PAYG). La oferta de Azure Marketplace incluye opciones de configuración del plan para JBoss EAP en RHEL, como máquinas virtuales independientes, máquinas virtuales en clúster o conjuntos de escalado de máquinas virtuales en clúster. Los 6 planes son:

- JBoss EAP 7.3 en RHEL 8.3 como máquina virtual independiente (Pago por uso)
- JBoss EAP 7.3 en RHEL 8.3 como máquina virtual independiente (BYOS)
- JBoss EAP 7.3 en RHEL 8.3 como máquina virtual en clúster (Pago por uso)
- JBoss EAP 7.3 en RHEL 8.3 como máquina virtual en clúster (BYOS)
- JBoss EAP 7.3 en RHEL 8.3 como conjuntos de escalado de máquinas virtuales en clúster (Pago por uso)
- JBoss EAP 7.3 en RHEL 8.3 como conjuntos de escalado de máquinas virtuales en clúster (BYOS)

### <a name="azure-quickstart-templates"></a>Plantillas de inicio rápido de Azure

Junto con las ofertas de Azure Marketplace, hay plantillas de inicio rápido disponibles para realizar la versión de prueba de EAP en Azure. Estos inicios rápidos incluyen scripts y plantillas de ARM precompilados para implementar JBoss EAP en Azure en varias configuraciones y combinaciones de versiones. La arquitectura de la solución incluye:

- JBoss EAP en RHEL (máquina virtual independiente)
- JBoss EAP en RHEL (máquinas virtuales en clúster)
- JBoss EAP en RHEL (conjunto de escalado de máquinas virtuales en clúster).

Para empezar, seleccione una de las plantillas de inicio rápido que coincidan con la combinación de la versión de JBoss EAP en RHEL. Consulte la documentación [Inicio rápido de JBoss EAP en Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/redhat/jboss-eap-on-rhel) para obtener más información. 

## <a name="prerequisites"></a>Prerrequisitos

* **Una cuenta de Azure con una suscripción activa**: si no tiene una suscripción de Azure, puede activar las [ventajas de suscriptor de Visual Studio Subscription](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) (anteriormente MSDN) o [crear una cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

- **Instalación de JBoss EAP**: debe tener una cuenta de Red Hat con el derecho de administración de suscripciones de Red Hat (RHSM) para el JBoss EAP. Este derecho le permitirá descargar la versión de JBoss EAP probada y certificada por Red Hat.  Si no tiene derechos de EAP, puede registrarse para obtener una suscripción gratuita para desarrolladores a través de la [suscripción para desarrolladores de Red Hat para usuarios](https://developers.redhat.com/register). Después de registrarse, puede encontrar las credenciales necesarias (los identificadores de grupo) en el [portal de clientes de Red Hat](https://access.redhat.com/management/).

- **Opciones de RHEL**: elija entre Pago por uso (PAYG) o Traiga su propia suscripción (BYOS). Con BYOS, debe activar su [imagen Gold de RHEL](https://access.redhat.com/) de [Red Hat Cloud Access](https://azure.microsoft.com/updates/red-hat-enterprise-linux-gold-images-now-available-on-azure/) antes de implementar la oferta de Marketplace con plantilla de soluciones. Siga [estas instrucciones](https://access.redhat.com/documentation/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/enabling-and-maintaining-subs_cloud-access) para habilitar las imágenes Gold de RHEL y usarlas en Microsoft Azure.

- **[Interfaz de la línea de comandos (CLI) de Azure](https://docs.microsoft.com/cli/azure/overview)** .

- **Código fuente de Java y [versión del Kit de desarrollo de Java (JDK)](https://www.oracle.com/java/technologies/javase-downloads.html)**

- **[Aplicación Java basada en JBoss EAP 7.2](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/development_guide/index#become_familiar_with_java_enterprise_edition_8)** o **[aplicación Java basada en JBoss EAP 7.3.](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.3/html-single/development_guide/index#get_started_developing_applications)**

**Opciones de RHEL**: elija entre PAYG o BYOS. En el caso de BYOS, deberá activar su imagen Gold de RHEL de [Red Hat Cloud Access](https://access.redhat.com/documentation/red_hat_subscription_management/1/html-single/red_hat_cloud_access_reference_guide/index) para usar la oferta de Azure Marketplace. Las ofertas de BYOS aparecerán en la sección "Oferta privada" de Azure Portal. 

**Versiones de productos**

* [JBoss EAP 7.2](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/)
* [JBoss EAP 7.3](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.3/)
* [RHEL 7.7](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux77-ARM)
* [RHEL 8.0](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux80-ARM)

## <a name="migration-flow-and-architecture"></a>Flujo de migración y arquitectura

En esta sección se describen las herramientas gratuitas para migrar aplicaciones JBoss EAP desde otro servidor de aplicaciones para que se ejecuten en JBoss EAP y desde servidores locales tradicionales al entorno de la nube de Microsoft Azure. 

### <a name="red-hat-migration-toolkit-for-applications-mta"></a>Kit de herramientas de migración para aplicaciones (MTA) de Red Hat

Se recomienda usar el MTA de Red Hat, para migrar aplicaciones Java, al principio del ciclo de planeamiento antes de ejecutar cualquier proyecto de migración relacionado con EAP. MTA es un ensamblado de herramientas que admite proyectos de migración y modernización de aplicaciones Java a gran escala en una [amplia gama de transformaciones y casos de uso](https://developers.redhat.com/products/mta/use-cases). Acelera el análisis de código de aplicación, admite la estimación del trabajo, acelera la migración de código y le ayuda a mover aplicaciones a la nube y los contenedores.

:::image type="content" source="./media/migration-toolkit.png" alt-text="La imagen muestra la página del panel de la aplicación de kit de herramientas de migración.":::

MTA de Red Hat permite migrar aplicaciones de otros servidores de aplicaciones a JBoss EAP de Red Hat.

## <a name="pre-migration"></a>Antes de la migración

Para asegurarse de que la migración se realiza correctamente, antes de empezar, complete los pasos de evaluación e inventario descritos en las secciones siguientes.

### <a name="validate-the-compatibility"></a>Validación de la compatibilidad

Se recomienda que valide el modelo y la versión de implementación actual antes de planear la migración. Es posible que tenga que realizar cambios importantes en la aplicación si no se admite su versión actual.

MTA admite migraciones desde servidores de aplicaciones empresariales de terceros, como el servidor WebLogic de Oracle, a JBoss EAP y actualizaciones a la versión más reciente de JBoss EAP.

En la tabla siguiente se describen las rutas de migración admitidas más comunes.

**Tabla: rutas de migración admitidas: de origen a destino**

|Plataforma de origen ⇒ | JBoss EAP 6 | JBoss EAP 7 | Red Hat OpenShift | OpenJDK 8 y 11 | Apache Camel 3 | Spring Boot en runtimes de RH | Quarkus
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Servidor WebLogic de Oracle | &#x2714; | &#x2714; | &#x2714; | &#x2714; | - | - | - |
| Servidor de aplicaciones WebSphere de IBM | &#x2714; | &#x2714; | &#x2714; | &#x2714; | - | - | - |
| JBoss EAP 4 | &#x2714; | &#x2714; | X<sup>1</sup> | &#x2714; | - | - | - |
| JBoss EAP 5 | &#x2714; | &#x2714; | &#x2714; | &#x2714; | - | - | - |
| JBoss EAP 7 | N/D | &#x2714; | &#x2714; | &#x2714; | - | - | - |
| Oracle JDK | - | - | &#x2714; | &#x2714; | - | - | - |
| Apache Camel 2 | - | - | &#x2714; | &#x2714; | &#x2714; | - | - |
| SpringBoot | - | - | &#x2714; | &#x2714; | - | &#x2714; | &#x2714; |
| Aplicación Java | - | - | &#x2714; | &#x2714; | - | - | - |

<sup>1</sup> Aunque MTA no proporciona reglas para esta ruta de migración actualmente, Red Hat Consulting puede ayudar con la migración desde cualquier plataforma de origen a JBoss EAP 7.

:::image type="content" source="./media/jboss-cli-image.png" alt-text="La imagen muestra la salida en la ventana de la CLI.":::

También puede consultar los [requisitos del sistema](https://access.redhat.com/documentation/en/migration_toolkit_for_applications/5.0/html-single/introduction_to_the_migration_toolkit_for_applications/index#system_requirements_getting-started-guide) para el MTA.

Compruebe las [configuraciones compatibles con JBoss EAP 7.3](https://access.redhat.com/articles/2026253#EAP_73) y las [configuraciones compatibles con JBoss EAP 7.2](https://access.redhat.com/articles/2026253#EAP_72) antes de planear la migración.

Para obtener la versión actual de Java, inicie sesión en su servidor y ejecute el siguiente comando:

```
java -version
```

### <a name="validate-operating-mode"></a>Validación del modo de funcionamiento

JBoss EAP es compatible con RHEL, Windows Server y Oracle Solaris. JBoss EAP se ejecuta en un modo de funcionamiento de servidor independiente para administrar instancias discretas o en modo de funcionamiento de dominio administrado para administrar grupos de instancias desde un único punto de control.

Los dominios administrados de JBoss EAP no se admiten en Microsoft Azure. Solo se admiten instancias de servidor de JBoss EAP independientes. Tenga en cuenta que la configuración de clústeres de JBoss EAP mediante servidores JBoss EAP independientes se admite en Azure y así es como la oferta de Azure Marketplace crea las máquinas virtuales en clúster o los conjuntos de escalado de máquinas virtuales.

### <a name="inventory-server-capacity"></a>Capacidad del servidor de inventario

Documente el hardware (memoria, CPU, disco, etc.) de los servidores de producción actuales, así como el promedio y máximo del número de solicitudes y el uso de recursos. Necesitará esta información independientemente de la ruta de migración que elija. Para obtener información adicional sobre los tamaños, consulte [Tamaños de Cloud Services](https://docs.microsoft.com/azure/cloud-services/cloud-services-sizes-specs).

### <a name="inventory-all-secrets"></a>Inventario de todos los secretos

Antes de la llegada de las tecnologías de "configuración como servicio", como [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) o [Azure App Configuration](https://azure.microsoft.com/services/app-configuration/) no había un concepto bien definido de "secretos". En su lugar, hay un conjunto dispar de opciones de configuración que funcionaban de forma eficaz como lo que ahora llamamos "secretos". Con los servidores de aplicaciones como JBoss EAP, estos secretos se encuentran en muchos archivos y almacenes de configuración diferentes. Compruebe los secretos y las contraseñas en todas las propiedades y los archivos de configuración de los servidores de producción. Asegúrese de comprobar *jboss-web.xml* en sus archivos WAR. Los archivos de configuración que contienen contraseñas o credenciales también se pueden encontrar dentro de la aplicación. Para obtener información adicional sobre Azure Key Vault, consulte [Conceptos básicos de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/basic-concepts).

### <a name="inventory-all-certificates"></a>Inventario de todos los certificados

Documente todos los certificados usados para los puntos de conexión SSL públicos. Para ver todos los certificados de los servidores de producción, ejecute el siguiente comando:

```cli
keytool -list -v -keystore <path to keystore>
```

### <a name="inventory-jndi-resources"></a>Inventario de los recursos de JNDI

- Inventario de todos los recursos de Java Naming and Directory Interface (JNDI). Otros recursos, como los agentes de Java Message Service (JMS), pueden requerir una migración o reconfiguración.

### <a name="insideyyour-application"></a>Aplicación InsideyYour 

Inspeccione los archivos WEB-INF/jboss-web.xml y WEB-INF/web.xml.

### <a name="document-data-sources"></a>Orígenes de datos de documentos

Si su aplicación usa bases de datos, debe capturar la siguiente información:

* ¿Cuál es el nombre del origen de datos?
* ¿Cuál es la configuración del grupo de conexiones?
* ¿Dónde puedo encontrar el archivo JAR controlador de Java Database Connectivity (JDBC)?

Para más información, consulte [Acerca de los orígenes de datos de JBoss EAP](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.3/html/configuration_guide/datasource_management) en la documentación de JBoss EAP.

### <a name="determine-whether-and-how-the-file-system-is-used"></a>Determinación de si se usa el sistema de archivos y cómo

Para usar el sistema de archivos en el servidor de aplicaciones será necesario cambiar la configuración o, en raras ocasiones, la arquitectura. Los módulos de JBoss EAP o el código de aplicación pueden usar el sistema de archivos. Es posible que identifique algunos o todos los escenarios descritos en las secciones siguientes.

**Contenido estático de solo lectura**

Si su aplicación actualmente sirve contenido estático, necesitará una ubicación alternativa para él. Quizás quiera considerar la posibilidad de mover el contenido estático a Azure Blob Storage y agregar [Azure Content Delivery Network (CDN)](https://docs.microsoft.com/azure/cdn/) para tener descargas de alta velocidad a nivel global. Para más información, consulte [Hospedaje de sitios web estáticos en Azure Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-static-website) e [Inicio rápido: Integración de una cuenta de una instancia de Azure Storage con Azure CDN](https://docs.microsoft.com/azure/cdn/cdn-create-a-storage-account-with-cdn).

**Contenido estático publicado dinámicamente**

Si su aplicación permite que haya contenido estático que la aplicación carga o produce, pero que es inmutable una vez creado, puede usar [Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/) y Azure CDN con una [función de Azure](https://docs.microsoft.com/azure/azure-functions/) para controlar las cargas y la actualización de la red CDN. Hemos proporcionado una implementación de ejemplo para su uso en [Cargar y carga previa en CDN de contenido estático con Azure Functions](https://github.com/Azure-Samples/functions-java-push-static-contents-to-cdn).

**Contenido dinámico o interno**

En el caso de los archivos que la aplicación lee y escribe con frecuencia (por ejemplo, los archivos de datos temporales) o los archivos estáticos que solo son visibles para la aplicación, puede montar recursos compartidos de [Azure Storage](https://docs.microsoft.com/azure/storage/) como volúmenes persistentes. Para más información, consulte [Creación dinámica y uso de un volumen persistente con Azure Files en Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/azure-files-dynamic-pv).

### <a name="determine-whether-a-connection-to-on-premises-is-needed"></a>Determinación de si se necesita una conexión al entorno local

Si su aplicación necesita acceder a cualquiera de los servicios locales, deberá aprovisionar uno de los servicios de conectividad de Azure. Para más información, consulte [Connect an on-premises network to Azure](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/) (Conexión de una red local a Azure). También tendrá que refactorizar la aplicación para que use las API disponibles públicamente que exponen los recursos locales.

### <a name="determine-whether-jms-queues-or-topics-are-in-use"></a>Determinación de si las colas o los temas de JMS están en uso

Si la aplicación utiliza colas o temas de JMS, deberá migrarlos a un servidor de JMS hospedado externamente. Azure Service Bus y Advanced Message Queuing Protocol (AMQP) pueden ser una estrategia de migración excelente para los usuarios que usan JMS. Para más información, consulte [Uso de JMS con Azure Service Bus y AMQP 1.0](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-java-how-to-use-jms-api-amqp) o [Envío y recepción de mensajes de colas de Azure Service Bus (Java)](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-java-how-to-use-queues)

Si se han configurado almacenes persistentes de JMS, debe capturar su configuración y aplicarla después de la migración.

### <a name="determine-whether-your-application-is-composed-of-multiple-wars"></a>Determinación de si la aplicación se compone de varios WAR

Si la aplicación se compone de varios WAR, debe tratar cada uno como aplicaciones independientes y seguir esta guía para cada una de ellas.

### <a name="determine-whether-your-application-is-packaged-as-an-ear"></a>Determinación de si la aplicación está empaquetada como EAR

Si la aplicación está empaquetada como un archivo EAR, examine el archivo application.xml y capture la configuración.

### <a name="identify-all-outside-processes-and-daemons-running-on-the-production-servers"></a>Identificación de todos los procesos externos y los demonios que se ejecutan en los servidores de producción

Si tiene procesos que se ejecutan fuera del servidor de aplicaciones, como los demonios de supervisión, tendrá que eliminarlos o migrarlos a otro lugar.


## <a name="migration"></a>Migración

### <a name="provision-the-target-infrastructure"></a>Aprovisionamiento de la infraestructura de destino

Para iniciar la migración, primero debe implementar la infraestructura de JBoss EAP. Existen varias opciones de implementación

- [**Máquina virtual de Azure**](https://azure.microsoft.com/overview/what-is-a-virtual-machine/)
- [**Conjunto de escalado de máquinas virtuales de Azure**](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview)
- [**Azure App Service**](https://aka.ms/jboss-app-service-overview)
- [**Red Hat OpenShift en Azure (ARO) para contenedores**](https://azure.microsoft.com/services/openshift)
- [**Azure Container Service**](https://azure.microsoft.com/product-categories/containers/)

Consulte la sección "Introducción a Azure Marketplace" para evaluar la infraestructura de implementación antes de compilar el entorno.

### <a name="perform-the-migration"></a>Realización de la migración

Existen herramientas que pueden ayudarle en la migración:

* [Kit de herramientas para migración de aplicaciones de Red Hat para analizar aplicaciones para la migración](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/migration_guide/index#use_windup_to_analyze_applications_for_migration).
* [Herramienta de migración de servidor de JBoss para migrar configuraciones de servidor](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/migration_guide/index#migration_tool_server_migration_tool)

Para migrar la configuración de su servidor de la versión anterior de JBoss EAP a la versión más reciente del mismo, puede usar la [herramienta de migración del servidor de JBoss](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/migration_guide/index#migrate_server_migration_tool_option) o puede realizar una migración manual con la ayuda de la [operación de migración de la CLI de administración](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/migration_guide/index#migrate__migrate_operation_option).

### <a name="run-red-hat-application-migration-toolkit"></a>Ejecución del kit de herramientas para migración de aplicaciones de Red Hat

Puede [ejecutar la herramienta de migración del servidor de JBoss en modo interactivo](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html-single/using_the_jboss_server_migration_tool/index#migration_tool_server_run_interactive_mode). Ppr defecto, la herramienta de migración del servidor de JBoss se ejecuta en modo interactivo. Este modo le permite elegir exactamente qué configuraciones de servidor desea migrar.

También puede [ejecutar la herramienta de migración del servidor de JBoss en modo no interactivo](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/using_the_jboss_server_migration_tool/index#migration_tool_server_run_noninteractive_mode). Este modo le permite ejecutarse sin pedir confirmación.

### <a name="review-the-result-of-jboss-server-migration-toolkit-execution"></a>Revisión del resultado de la ejecución del kit de herramientas de migración del servidor de JBoss

Una vez completada la migración, revise los archivos de configuración del servidor migrados en los directorios *EAP_HOME/standalone/configuration/* y *EAP_HOME/domain/configuration/* . Para obtener más información, consulte la [Revisión de los resultados de la ejecución de la herramienta de migración del servidor de JBoss](https://access.redhat.com/documentation/red_hat_jboss_enterprise_application_platform/7.2/html-single/using_the_jboss_server_migration_tool/index#migration_tool_server_results).

### <a name="expose-the-application"></a>Exposición de la aplicación

Puede exponer la aplicación mediante los métodos siguientes, lo que es adecuado para su entorno.

* [Cree una dirección IP pública](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address) para acceder al servidor y a la aplicación.
* [Cree una máquina virtual de salto en la misma red virtual (VNet)](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#create-virtual-machine) en una subred diferente (nueva subred) en la misma red virtual y acceda al servidor a través de una máquina virtual de salto. Esta máquina virtual de salto se puede usar para exponer la aplicación.
* [Cree una máquina virtual de salto con emparejamiento de red virtual](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#create-virtual-machine) en una red virtual distinta, acceda al servidor y exponga la aplicación mediante [emparejamiento de red virtual](https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal#peer-virtual-networks).
* Exponga la aplicación mediante una [puerta de enlace de aplicación](https://docs.microsoft.com/azure/application-gateway/quick-create-portal#create-an-application-gateway)
* Exponga la aplicación mediante un [equilibrador de carga externo](https://docs.microsoft.com/azure/load-balancer/tutorial-load-balancer-standard-manage-portal#create-a-standard-load-balancer) (ELB).

## <a name="post-migration"></a>Después de la migración

Una vez alcanzados los objetivos de migración que se han definido antes de la migración, realice pruebas integrales de aceptación para comprobar que todo funciona según lo previsto. Entre los temas de mejoras posteriores a la migración se incluyen, entre otros:

* Usar Azure Storage para servir el contenido estático montado en las máquinas virtuales. Para obtener más información, consulte [Adjuntar o desasociar un disco de datos a una máquina virtual](https://docs.microsoft.com/azure/devtest-labs/devtest-lab-attach-detach-data-disk)
* Implementar las aplicaciones en el clúster de JBoss migrado con Azure DevOps. Para más información, consulte la [documentación de introducción a Azure DevOps](https://docs.microsoft.com/azure/devops/get-started/?view=azure-devops).
* Considere la posibilidad de usar [Application Gateway](https://docs.microsoft.com/azure/application-gateway/).
* Mejorar la topología de red con servicios avanzados de equilibrio de carga. Para más información, consulte [Uso de servicios de equilibrio de carga en Azure](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-load-balancing-azure).
* Aproveche las identidades administradas de Azure para los secretos administrados y asigne un control de acceso basado en roles (RBAC) a los recursos de Azure. Para obtener más información, consulte [¿Qué son las identidades administradas para recursos de Azure?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)
* Usar Azure Key Vault para almacenar toda la información que funcione como un "secreto". Para más información, consulte [Conceptos básicos de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/basic-concepts).

## <a name="resource-links-and-support"></a>Vínculos de recursos y soporte técnico

Para cualquier pregunta relacionada con el soporte técnico, incidencias o requisitos de personalización, póngase en contacto con el [Soporte técnico de Red Hat](https://access.redhat.com/support) o el [Soporte técnico de Microsoft Azure](https://ms.portal.azure.com/?quickstart=true#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

* Más información sobre [JBoss EAP](https://access.redhat.com/documentation/en/red_hat_jboss_enterprise_application_platform/7.2/html/getting_started_with_jboss_eap_for_openshift_online/introduction)
* Más información sobre el [administrador de suscripciones de Red Hat (Cloud Access)](https://access.redhat.com/documentation/en/red_hat_subscription_management/1/html-single/red_hat_cloud_access_reference_guide/index)
* Obtenga más información sobre [Azure Virtual Machines](https://azure.microsoft.com/overview/what-is-a-virtual-machine/).
* Más información sobre los [conjuntos de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview)
* Más información sobre [Red Hat OpenShift en Azure](https://azure.microsoft.com/services/openshift/)
* Más información sobre [¿Qué es App Service en Linux?](https://docs.microsoft.com/azure/app-service/overview#app-service-on-linux)
* Más información sobre [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-introduction)
* Más información sobre [Redes de Azure](https://docs.microsoft.com/azure/networking/networking-overview)

## <a name="next-steps"></a>Pasos siguientes
* [Implementación de JBoss EAP en máquinas virtuales o en un conjunto de escalado de máquinas virtuales de RHEL desde Azure Marketplace](https://aka.ms/AMP-JBoss-EAP)
* [Configuración de una aplicación Java para Azure App Service](https://docs.microsoft.com/azure/app-service/configure-language-java)
* Tutorial [Cómo implementar JBoss EAP en Azure App Service](https://github.com/JasonFreeberg/jboss-on-app-service)
* [Uso de la asistencia para la migración de Azure App Service](https://azure.microsoft.com/services/app-service/migration-assistant/)
* [Uso del kit de herramientas de migración para aplicaciones de Red Hat](https://developers.redhat.com/products/mta)
