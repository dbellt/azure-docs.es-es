---
title: Procedimientos recomendados de seguridad de Azure Service Fabric
description: En este artículo se proporciona un conjunto de procedimientos recomendados para la reforzar la seguridad de Azure Service Fabric.
author: unifycloud
ms.author: tomsh
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.date: 01/16/2019
ms.openlocfilehash: a7d87e2496158fec8ff33ab8586c845a6207f810
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816069"
---
# <a name="azure-service-fabric-security-best-practices"></a>Procedimientos recomendados de seguridad de Azure Service Fabric
La implementación de una aplicación en Azure es un proceso rápido, sencillo y rentable. Antes de implementar la aplicación en la nube en producción, revise la lista de procedimientos recomendados y fundamentales para implementar clústeres seguros en la aplicación.

Azure Service Fabric es una plataforma de sistemas distribuidos que facilita el empaquetamiento, la implementación y la administración de microservicios escalables y confiables. Service Fabric también aborda los desafíos importantes en el desarrollo y la administración de aplicaciones en la nube. Los desarrolladores y administradores pueden evitar problemas complejos de infraestructura y centrarse en su lugar en las cargas de trabajo más exigentes y críticas que son escalables, confiables y fáciles de administrar.

Para cada procedimiento recomendado, explicaremos:

-   El procedimiento recomendado.
-   Por qué debería implementar el procedimiento recomendado.
-   Qué puede ocurrir si no se implementa el procedimiento recomendado.
-   Cómo aprender a implementar el procedimiento recomendado.

Disponemos de los siguientes procedimientos recomendados de seguridad de Azure Service Fabric:

-   Usar plantillas de Azure Resource Manager y el módulo de PowerShell de Service Fabric para crear clústeres seguros.
-   Usar certificados X.509.
-   Configurar directivas de seguridad.
-   Implementar la configuración de seguridad de Reliable Actors.
-   Configurar TLS para Azure Service Fabric.
-   Usar aislamiento y seguridad de red con Azure Service Fabric.
-   Configurar Azure Key Vault para la seguridad.
-   Asignar usuarios a roles.


## <a name="best-practices-for-securing-your-clusters"></a>Procedimientos recomendados para proteger los clústeres

Use siempre un clúster seguro:
-   Implemente la seguridad del clúster mediante el uso de certificados.
-   Proporcione acceso de cliente (administrador y de solo lectura) mediante el uso de Azure Active Directory (Azure AD).

Use implementaciones automatizadas:
-   Use scripts para generar, implementar y sustituir secretos.
-   Almacene los secretos en Azure Key Vault y use Azure AD para todos los otros accesos de cliente.
-   Requiera autenticación para el acceso de usuario a los secretos.

Además, tenga en cuenta las siguientes opciones de configuración:
-   Cree redes perimetrales (también conocido como DMZ y subredes filtradas) mediante el uso de grupos de seguridad de red (NSG) de Azure.
-   Use servidores de salto con conexión a Escritorio remoto para obtener acceso a las máquinas virtuales (VM) del clúster o administrar el clúster.

Los clústeres deben estar protegidos para evitar que usuarios no autorizados se conecten a ellos, especialmente cuando un clúster está en producción. Aunque es posible crear un clúster sin protección, si este expone los puntos de conexión de administración a Internet público, podrían conectarse a él usuarios anónimos.

Hay tres [escenarios](../../service-fabric/service-fabric-cluster-security.md) para implementar la seguridad del clúster mediante el uso de varias tecnologías:

-   Seguridad de nodo a nodo: este escenario protege la comunicación entre las máquinas virtuales y los equipos del clúster. Esta forma de seguridad garantiza que solo los equipos que están autorizados a unirse al clúster pueden hospedar aplicaciones y servicios en el clúster.
En este escenario, los clústeres que se ejecutan en Azure o los independientes que se ejecutan en Windows pueden utilizar una [seguridad basada en certificados](../../service-fabric/service-fabric-windows-cluster-x509-security.md) o la [seguridad de Windows](../../service-fabric/service-fabric-windows-cluster-windows-security.md) para las máquinas con Windows Server.
-   Seguridad de cliente a nodo: este escenario protege la comunicación entre un cliente de Service Fabric y los nodos individuales del clúster.
-   Control de acceso basado en rol de Service Fabric (RBAC de Service Fabric): este escenario utiliza identidades diferentes (certificados, Azure AD, etc) para cada rol de cliente de administrador y usuario que tiene acceso al clúster. Se especifican las identidades de rol cuando se crea el clúster.

>[!NOTE]
>**Recomendación de seguridad para los clústeres de Azure**: utilice la seguridad de Azure AD para autenticar a los clientes y certificados para la seguridad de nodo a nodo.

Para configurar el clúster de Windows independiente, consulte [Opciones de configuración de un clúster de Windows independiente](../../service-fabric/service-fabric-cluster-manifest.md).

Use plantillas de Azure Resource Manager y el módulo de PowerShell de Service Fabric para crear un clúster seguro.
Para instrucciones paso a paso sobre la creación de un clúster seguro de Service Fabric con las plantillas de Azure Resource Manager, consulte [Creación de un clúster de Service Fabric](../../service-fabric/service-fabric-cluster-creation-via-arm.md).

Utilice la plantilla de Azure Resource Manager:
-   Personalice el clúster mediante la plantilla para configurar el almacenamiento administrado para los discos duros virtuales (VHD) de la máquina virtual.
-   Realice los cambios al grupo de recursos mediante la plantilla para facilitar la administración de la configuración y la auditoría.

Trate la configuración del clúster como código:
-   Sea exhaustivo al comprobar las configuraciones de implementación.
-   Evite el uso de comandos implícitos para modificar directamente los recursos.

Muchos aspectos del [ciclo de vida de la aplicación de Service Fabric](../../service-fabric/service-fabric-application-lifecycle.md) pueden automatizarse. El [módulo de PowerShell de Service Fabric](../../service-fabric/service-fabric-deploy-remove-applications.md#upload-the-application-package) automatiza las tareas comunes para implementar, actualizar, eliminar y probar aplicaciones de Azure Service Fabric. También cuenta con API HTTP y administradas para la administración de aplicaciones.

## <a name="use-x509-certificates"></a>Uso de certificados X.509
Proteja siempre los clústeres mediante el uso de certificados X.509 o con la seguridad de Windows. Solo se configura la seguridad en el momento de creación de clúster. No es posible activar la seguridad una vez creado el clúster.

Para especificar un [certificado de clúster](../../service-fabric/service-fabric-windows-cluster-x509-security.md), establezca el valor de la propiedad **ClusterCredentialType** en X509. Para especificar un certificado de servidor para conexiones externas, establezca la propiedad **ServerCredentialType** en X509.

Además, siga estos procedimientos:
-   Cree los certificados para los clústeres de producción mediante un servicio de certificados de Windows Server configurado correctamente. También puede obtener los certificados de una entidad de certificación aprobada (CA).
-   Nunca utilice a un certificado temporal o de prueba para clústeres de producción si el certificado se creó mediante MakeCert.exe u otra herramienta similar.
-   Use un certificado autofirmado para clústeres de prueba, pero no para clústeres de producción.

Si el clúster no es seguro, cualquiera puede conectarse de forma anónima y realizar operaciones de administración. Por esta razón, proteja siempre los clústeres de producción mediante el uso de certificados X.509 o con la seguridad de Windows.

Para más información sobre el uso de certificados X.509, consulte [Agregar o quitar certificados para un clúster de Service Fabric](../../service-fabric/service-fabric-cluster-security-update-certs-azure.md).

## <a name="configure-security-policies"></a>Configuración de directivas de seguridad
Service Fabric también protege los recursos utilizados por las aplicaciones. Los recursos tales como archivos, directorios y certificados se almacenan en las cuentas de usuario cuando se implementa la aplicación. Esta característica aumenta la seguridad entre las aplicaciones en ejecución, incluso en un entorno hospedado compartido.

-   Use un usuario o un grupo de dominios de Active Directory: ejecute el servicio con las credenciales de una cuenta de usuario o grupo de Active Directory. Asegúrese de utilizar Active Directory local dentro del dominio y no Azure Active Directory. Utilice un usuario o grupo de dominio para tener acceso a otros recursos del dominio en los que se le hayan concedido permisos. Por ejemplo, recursos compartidos de archivos.

-   Asigne una directiva de acceso de seguridad a los puntos de conexión HTTP y HTTPS: especifique la propiedad **SecurityAccessPolicy** para aplicar una directiva **RunAs** a un servicio cuando el manifiesto de servicio declare los recursos del punto de conexión con HTTP. Los puertos asignados a los puntos de conexión HTTP son listas de acceso controlado para la cuenta de usuario RunAs en la que se ejecuta el servicio. Cuando la directiva no está establecida, http.sys no tiene acceso al servicio y pueden aparecer errores en las llamadas del cliente.

Para más información sobre el uso de directivas de seguridad en un clúster de Service Fabric, consulte [Configuración de directivas de seguridad para la aplicación](../../service-fabric/service-fabric-application-runas-security.md).

## <a name="implement-the-reliable-actors-security-configuration"></a>Implementación de la configuración de seguridad de Reliable Actors
Service Fabric Reliable Actors es una implementación del modelo de diseño del actor. Al igual que sucede con cualquier modelo de diseño de software, la decisión de usar un modelo específico se toma en función de si un problema de diseño de software se adapta al modelo o no.

En general, utilice el modelo de diseño de actor para ayudar a modelar soluciones para los siguientes problemas de software o escenarios de seguridad:
-   El espacio del problema implica un gran número (miles o más) de unidades pequeñas, independientes y aisladas de estado y lógica.
-   Trabaja con objetos de un único subproceso que no requieren una interacción importante con componentes externos, incluida la consulta del estado en un conjunto de actores.
-   Las instancias de actor no bloquean a los autores de llamadas con retrasos imprevisibles emitiendo operaciones de E/S.

En Service Fabric, los actores se implementan en el marco de trabajo de la aplicación de Reliable Actors. Este marco de trabajo se basa en el patrón de actor y está construido sobre [Reliable Services de Service Fabric](../../service-fabric/service-fabric-reliable-services-introduction.md). Cada servicio de Reliable Actors que escribe es un servicio de confianza con estado particionado.

Cada actor se define como una instancia de un tipo de actor, similar a la forma en que un objeto de .NET es una instancia de un tipo de .NET. Por ejemplo, un **tipo de actor** que implementa la funcionalidad de una calculadora puede tener muchos actores de ese tipo distribuidos en varios nodos en un clúster. Cada uno de los actores distribuidos se caracteriza de forma única por un identificador de actor.

[Las configuraciones de seguridad del replicador](../../service-fabric/service-fabric-reliable-actors-kvsactorstateprovider-configuration.md) se usan para proteger el canal de comunicación que se utiliza durante la replicación. Esta configuración impide que los servicios vean el tráfico de replicación de otros servicios y garantiza que los datos de alta disponibilidad son seguros. De forma predeterminada, una sección de configuración de seguridad vacía impide la seguridad de la replicación.
Las configuraciones de replicador configuran el replicador que es responsable de hacer que el proveedor de estado del actor resulte altamente confiable.

## <a name="configure-tls-for-azure-service-fabric"></a>Configuración de TLS para Azure Service Fabric
El proceso de autenticación de servidor [autentica](../../service-fabric/service-fabric-cluster-creation-via-arm.md) los puntos de conexión de administración del clúster a un cliente de administración. El cliente de administración, a continuación, reconoce que se comunica con el clúster real. Este certificado proporciona también un [TLS](../../service-fabric/service-fabric-cluster-creation-via-arm.md) para la API de administración de HTTPS y para Service Fabric Explorer sobre HTTPS.
Debe adquirir un nombre de dominio personalizado para el clúster. Cuando solicite un certificado de una entidad de certificación, el nombre de sujeto del certificado debe coincidir con el nombre de dominio personalizado del clúster.

Para configurar TLS para una aplicación, primero debe obtener un certificado SSL/TLS firmado por una entidad de certificación. La entidad de certificación es un tercero de confianza que emite certificados para TLS por motivos de seguridad. Si todavía no tiene un certificado SSL/TLS, deberá obtenerlo de una compañía que los venda.

El certificado debe cumplir los siguientes requisitos de certificados SSL/TLS en Azure:
-   El certificado debe contener una clave privada.

-   El certificado debe crearse para el intercambio de claves, que se puedan exportar a un archivo Personal Information Exchange (.pfx).

-   El nombre de sujeto del certificado debe coincidir con el nombre de dominio usado para tener acceso al servicio en la nube.

    - Debe adquirir un nombre de dominio personalizado para el acceso a su servicio en la nube.
    - Solicite un certificado a una entidad de certificación con un nombre de sujeto que coincida con el nombre de dominio personalizado del servicio. Por ejemplo, si el nombre de dominio personalizado es __contoso__ **.com**, el certificado de la CA debe tener el nombre de sujeto **.contoso.com** o __www__ **.contoso.com**.

    >[!NOTE]
    >No se puede obtener un certificado SSL/TLS de una entidad de certificación para el dominio __cloudapp__ **.net**.

-   Este certificado debe usar un cifrado de 2,048 bits como mínimo.

El protocolo HTTP no es seguro y está sujeto a ataques de interceptación. Los datos que se transmiten a través de HTTP se envían como texto sin formato desde el explorador web al servidor web o entre otros puntos de conexión. Los atacantes pueden interceptar y ver datos confidenciales enviados a través de HTTP, como los detalles de las tarjetas de crédito e inicios de sesión de cuenta. Cuando los datos se envían o se publican a través de un explorador mediante HTTPS, SSL garantiza que esa información se cifra y protege de la interceptación.

Para más información sobre el uso de certificados SSL/TLS, consulte [Configuración de TLS para una aplicación en Azure](../../cloud-services/cloud-services-configure-ssl-certificate-portal.md).

## <a name="use-network-isolation-and-security-with-azure-service-fabric"></a>Uso de aislamiento y seguridad de red con Azure Service Fabric
Configure un clúster seguro de 3 tipos de nodo mediante la [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) como ejemplo. Controle el tráfico de red entrante y saliente mediante el uso de la plantilla y los grupos de seguridad de red.

La plantilla tiene un grupo de seguridad de red para cada uno de los conjuntos de escalado de máquinas virtuales a fin de controlar el tráfico dentro y fuera del conjunto. De forma predeterminada, las reglas se configuran para permitir todo el tráfico que necesitan los servicios del sistema y los puertos de la aplicación especificados en la plantilla. Revise esas reglas y realice los cambios que necesite, incluido agregar nuevas reglas para las aplicaciones.

Para más información, consulte [Escenarios comunes de redes para Azure Service Fabric](../../service-fabric/service-fabric-patterns-networking.md).

## <a name="set-up-azure-key-vault-for-security"></a>Configuración de Azure Key Vault para la seguridad
Service Fabric usa certificados para proporcionar autenticación y cifrado con el fin de proteger un clúster y sus aplicaciones.

Service Fabric usa certificados X.509 para proteger un clúster y proporcionar características de seguridad de las aplicaciones. Azure Key Vault se usa para [administrar certificados](../../service-fabric/service-fabric-cluster-security-update-certs-azure.md) para clústeres de Service Fabric en Azure. El proveedor de recursos de Azure que crea los clústeres extrae los certificados de un almacén de claves. El proveedor, a continuación, instala los certificados en las máquinas virtuales cuando el clúster se implementa en Azure.

Existe una relación de certificados entre [Azure Key Vault](../../key-vault/general/security-features.md), el clúster de Service Fabric y el proveedor de recursos que usa los certificados. Cuando se crea el clúster, la información sobre la relación de certificados se almacena en un almacén de claves.

Hay dos pasos básicos para configurar un almacén de claves:
1. Crear un grupo de recursos específico para el almacén de claves.

    Se recomienda colocar el almacén de claves en su propio grupo de recursos. Esta acción ayuda a evitar la pérdida de las claves y secretos si se eliminan otros grupos de recursos, como almacenamiento, proceso o el grupo que contiene el clúster. El grupo de recursos que contiene el almacén de claves debe estar en la misma región que el clúster que lo usa.

2. Crear un almacén de claves en el nuevo grupo de recursos.

    El almacén de claves debe estar habilitado para la implementación. El proveedor de recursos de proceso, a continuación, puede obtener los certificados del almacén e instalarlos en las instancias de máquina virtual.

Para obtener más información sobre cómo configurar un almacén de claves, consulte [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md).

## <a name="assign-users-to-roles"></a>Asignación de usuarios a roles
Una vez que haya creado las aplicaciones para representar el clúster, debe asignar los usuarios a los roles compatibles con Service Fabric: solo lectura y administrador. Puede asignar los roles mediante Azure Portal.

>[!NOTE]
> Para más información sobre el uso de roles en Service Fabric, consulte [Control de acceso basado en roles de Service Fabric para clientes de Service Fabric](../../service-fabric/service-fabric-cluster-security-roles.md).

Azure Service Fabric admite dos tipos de control de acceso para los clientes que están conectados a un [clúster de Service Fabric](../../service-fabric/service-fabric-cluster-creation-via-arm.md): administrador y usuario. El administrador del clúster puede usar el control de acceso para limitarlo a determinadas operaciones del clúster para los diferentes grupos de usuarios. El control de acceso hace que el clúster sea más seguro.

## <a name="next-steps"></a>Pasos siguientes

- [Lista de comprobación de seguridad de Service Fabric](../../service-fabric/service-fabric-best-practices-security.md)
- Configurar el [entorno de desarrollo](../../service-fabric/service-fabric-get-started.md) de Service Fabric.
- Más información sobre las [opciones de soporte técnico de Service Fabric](../../service-fabric/service-fabric-support.md).