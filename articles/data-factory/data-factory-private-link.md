---
title: Azure Private Link para Azure Data Factory
description: Más información sobre el funcionamiento de Azure Private Link en Azure Data Factory.
ms.author: lle
author: lrtoyou1223
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/01/2020
ms.openlocfilehash: f84d7d7a02b75723f78cfbed9ee23e19ebea9a15
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481686"
---
# <a name="azure-private-link-for-azure-data-factory"></a>Azure Private Link para Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

Con Azure Private Link, puede conectarse a varias implementaciones de plataformas como servicio (PaaS) en Azure a través de un punto de conexión privado. Un punto de conexión privado es una dirección IP privada dentro de una red virtual y una subred específicas. Para obtener una lista de las implementaciones PaaS que admiten la funcionalidad Private Link, vea la [documentación de Private Link](../private-link/index.yml). 

## <a name="secure-communication-between-customer-networks-and-azure-data-factory"></a>Comunicación segura entre las redes del cliente y Azure Data Factory 
Puede configurar una red virtual de Azure como una representación lógica de la red en la nube. Esta acción tiene estas ventajas derivadas:
* Ayuda a proteger los recursos de Azure frente a ataques en redes públicas.
* Permite que las redes y Data Factory se comuniquen entre sí de forma segura. 

También puede conectar una red local a la red virtual mediante la configuración de una conexión VPN del protocolo de seguridad de Internet (IPsec) (de sitio a sitio) o una conexión de Azure ExpressRoute (emparejamiento privado). 

Asimismo, puede instalar un entorno de ejecución de integración autohospedado en una máquina local o en una máquina virtual de la red virtual. Esto le permite:
* Ejecutar actividades de copia entre un almacén de datos en la nube y un almacén de datos en una red privada.
* Distribuir las actividades de transformación frente a los recursos de proceso en una red local o una red virtual de Azure. 

Se requieren varios canales de comunicación entre Azure Data Factory y la red virtual del cliente, tal y como se muestra en la tabla siguiente:

| Domain | Port | Descripción |
| ---------- | -------- | --------------- |
| `adf.azure.com` | 443 | Un plano de control, necesario para la creación y supervisión en Data Factory. |
| `*.{region}.datafactory.azure.net` | 443 | El entorno de ejecución de integración autohospedado lo necesita para conectarse al servicio Data Factory. |
| `*.servicebus.windows.net` | 443 | Lo necesita el entorno de ejecución de integración autohospedado para la creación interactiva. |
| `download.microsoft.com` | 443 | Lo necesita el entorno de ejecución de integración autohospedado para descargar las actualizaciones. |

Con la compatibilidad de Private Link para Azure Data Factory, puede:
* Crear un punto de conexión privado en la red virtual.
* Habilitar la conexión privada a una instancia de factoría de datos específica. 

Las comunicaciones con el servicio Azure Data Factory pasan a través de Private Link y ayudan a ofrecer conectividad privada segura. 

![Diagrama de Private Link para la arquitectura de Azure Data Factory.](./media/data-factory-private-link/private-link-architecture.png)

La habilitación del servicio Private Link para cada uno de los canales de comunicación anteriores ofrece la siguiente funcionalidad:
- **Admitido**:
   - Puede crear y supervisar la factoría de datos en la red virtual, incluso si bloquea todas las comunicaciones salientes.
   - Las comunicaciones de comandos entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory se pueden realizar de forma segura en un entorno de red privado. El tráfico entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory pasa a través Private Link. 
- **No admitido actualmente**:
   - La creación interactiva que usa el entorno de ejecución de integración autohospedado pasa a través de Private Link como, por ejemplo, en el caso de las pruebas de conexiones, el examen de la lista de carpetas y de tablas, la obtención de esquemas y la vista previa de los datos.
   - La nueva versión del entorno de ejecución de integración autohospedado que se puede descargar automáticamente desde el Centro de descarga de Microsoft si habilita la actualización automática no se admite en este momento.

   > [!NOTE]
   > Para las funcionalidades que no se admiten actualmente deberá configurar el dominio y el puerto previamente mencionados en la red virtual o el firewall corporativo. 

   > [!NOTE]
   > La conexión a Azure Data Factory a través de un punto de conexión privado solo es aplicable al entorno de ejecución de integración autohospedado en la factoría de datos. No se admite en Synapse.

> [!WARNING]
> Cuando cree un servicio vinculado, asegúrese de que las credenciales se almacenan en un almacén de claves de Azure. De lo contrario, las credenciales no funcionarán al habilitar Private Link en Azure Data Factory.

## <a name="dns-changes-for-private-endpoints"></a>Cambios de DNS en puntos de conexión privados
Al crear un punto de conexión privado, el registro del recurso CNAME de DNS de Data Factory se actualiza a un alias de un subdominio con el prefijo "privatelink". De forma predeterminada, también se crea una [zona DNS privada](../dns/private-dns-overview.md), que se corresponde con el subdominio "privatelink", con los registros de recursos A de DNS para los puntos de conexión privados.

Cuando se resuelve la dirección URL del punto de conexión de la factoría de datos desde fuera de VNet con el punto de conexión privado, se resuelve en el punto de conexión público del servicio de factoría de datos. Cuando se resuelve desde la red virtual que hospeda el punto de conexión privado, la dirección URL del punto de conexión de almacenamiento se resuelve en la dirección IP del punto de conexión privado.

En el ejemplo anterior, los registros de recursos DNS de Data Factory "DataFactoryA", cuando se resuelven desde fuera de la red virtual que hospeda el punto de conexión privado, serán:

| Nombre | Tipo | Value |
| ---------- | -------- | --------------- |
| DataFactoryA.{region}.datafactory.azure.net | CNAME   | DataFactoryA.{region}.privatelink.datafactory.azure.net |
| DataFactoryA.{region}.privatelink.datafactory.azure.net | CNAME   | < punto de conexión público del servicio de factoría de datos > |
| < punto de conexión público del servicio de factoría de datos >  | A | < dirección IP pública del servicio de factoría de datos > |

Los registros de recursos DNS para DataFactoryA, cuando se resuelvan en la instancia de VNet que hospeda el punto de conexión privado, serán:

| Nombre | Tipo | Value |
| ---------- | -------- | --------------- |
| DataFactoryA.{region}.datafactory.azure.net | CNAME   | DataFactoryA.{region}.privatelink.datafactory.azure.net |
| DataFactoryA.{region}.privatelink.datafactory.azure.net   | A | < dirección IP del punto de conexión privado > |

Si va a usar un servidor DNS personalizado en la red, los clientes deben ser capaces de resolver el FQDN del punto de conexión de Data Factory en la dirección IP del punto de conexión privado. Debería configurar el servidor DNS para delegar el subdominio del vínculo privado en la zona DNS privada de la red virtual o configurar los registros A para "DataFactoryA.{region}.privatelink.datafactory.azure.net" con la dirección IP del punto de conexión privado.

Para más información sobre cómo configurar su propio servidor DNS para que admita puntos de conexión privados, consulte los siguientes artículos:
- [Resolución de nombres de recursos en redes virtuales de Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
- [Configuración de DNS para puntos de conexión privados](../private-link/private-endpoint-overview.md#dns-configuration)


## <a name="set-up-private-link-for-azure-data-factory"></a>Configuración de Private Link para Azure Data Factory
Puede crear puntos de conexión privados mediante [Azure Portal](../private-link/create-private-endpoint-portal.md).

Puede elegir si desea conectar su entorno de ejecución de integración autohospedado a Azure Data Factory a través de un punto de conexión público o un punto de conexión privado. 

![Captura de pantalla de bloqueo del acceso público de entorno de ejecución de integración autohospedado.](./media/data-factory-private-link/disable-public-access-shir.png)


También puede ir a Azure Data Factory en Azure Portal y crear un punto de conexión privado, como se muestra aquí:

![Captura de pantalla del panel "Conexiones de punto de conexión privado" para crear un punto de conexión privado.](./media/data-factory-private-link/create-private-endpoint.png)

En el paso de **Recurso**, seleccione **Microsoft.Datafactory/factories** como **Tipo de recurso**. Y si desea crear un punto de conexión privado para las comunicaciones de comandos entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory, seleccione **DataFactory** para **Subrecurso de destino**.

![Captura de pantalla del panel "Conexiones de punto de conexión privado" para seleccionar el recurso.](./media/data-factory-private-link/private-endpoint-resource.png)

> [!NOTE]
> Deshabilitar el acceso a la red pública solo es aplicable al entorno de ejecución de integración autohospedado, no a Azure Integration Runtime ni a SQL Server Integration Services (SSIS) Integration Runtime.

Si desea crear un punto de conexión privado para la creación y supervisión de la factoría de datos en la red virtual, seleccione **Portal** para **Subrecurso de destino**.

> [!NOTE]
> Puede seguir accediendo al portal de Azure Data Factory a través de la red pública después de crear el punto de conexión privado para el portal.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](quickstart-create-data-factory-portal.md)
- [Introducción al servicio Azure Data Factory](introduction.md)
- [Creación visual de Azure Data Factory](author-visually.md)
