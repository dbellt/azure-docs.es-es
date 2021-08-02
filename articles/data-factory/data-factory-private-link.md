---
title: Azure Private Link para Azure Data Factory
description: Más información sobre el funcionamiento de Azure Private Link en Azure Data Factory.
ms.author: lle
author: lrtoyou1223
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 06/10/2021
ms.openlocfilehash: 9d41ff8d2b0bfd1e83f15366e152398f5de8ccf9
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112020984"
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
> Si habilita Private Link en Azure Data Factory y bloquea el acceso público al mismo tiempo, asegúrese de que, al crear un servicio vinculado, las credenciales se almacenan en un almacén de claves de Azure. De lo contrario, las credenciales no funcionarán.

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


## <a name="set-up-a-private-endpoint-link-for-azure-data-factory"></a>Configuración de un vínculo de punto de conexión privado para Azure Data Factory

En esta sección configurará un vínculo de punto de conexión privado para Azure Data Factory.

Puede elegir si quiere conectar su entorno de ejecución de integración autohospedado (SHIR) a Azure Data Factory mediante un punto de conexión público o privado durante el paso de creación de la factoría de datos, que se muestra aquí: 

:::image type="content" source="./media/data-factory-private-link/disable-public-access-shir.png" alt-text="Captura de pantalla de bloqueo del acceso público de entorno de ejecución de integración autohospedado.":::

Puede cambiar la selección en cualquier momento después de la creación desde la página del portal de Data Factory en la hoja Redes.  Después de habilitar allí los puntos de conexión privados, también debe agregar un punto de conexión privado a la factoría de datos.

Un punto de conexión privado requiere una red virtual y una subred para el vínculo, y una máquina virtual dentro de la subred, que se usará para ejecutar el entorno de ejecución de integración autohospedado (SHIR), que se conecta a través del vínculo del punto de conexión privado.

### <a name="create-the-virtual-network"></a>Crear la red virtual
Si no tiene una red virtual para usarla con el vínculo de punto de conexión privado, debe crear una y asignar una subred.  

1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Red virtual** o busque **Red virtual** en el cuadro de búsqueda.

3. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | **Configuración**          | **Valor**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Detalles del proyecto**  |                                                                 |
    | Suscripción     | Selección de su suscripción a Azure                                  |
    | Grupo de recursos   | Seleccione un grupo de recursos para la red virtual. |
    | **Detalles de instancia** |                                                                 |
    | Nombre             | Escriba un nombre para la red virtual. |
    | Region           | IMPORTANTE: Seleccione la misma región que usará el punto de conexión privado. |

4. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

5. En la pestaña **Direcciones IP**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Espacio de direcciones IPv4 | Escriba **10.1.0.0/16**. |

6. En **Nombre de subred**, seleccione la palabra **predeterminada**.

7. En **Editar subred**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre de subred | Escriba un nombre para la subred. |
    | Intervalo de direcciones de subred | Escriba **10.1.0.0/24**. |

8. Seleccione **Guardar**.

9. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

10. Seleccione **Crear**.

### <a name="create-a-virtual-machine-for-the-self-hosted-integration-runtime-shir"></a>Creación de una máquina virtual para el entorno de ejecución de integración autohospedado (SHIR)
También debe crear o asignar una máquina virtual para ejecutar el entorno de ejecución de integración autohospedado en la nueva subred creada anteriormente.

1. En la parte superior izquierda del portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual** o **Máquina virtual** en el cuadro de búsqueda.
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Valor                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Selección de un grupo de recursos |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba un nombre para la máquina virtual. |
    | Region | Seleccione la región usada anteriormente para la red virtual. |
    | Opciones de disponibilidad | Seleccione **No se requiere redundancia de la infraestructura** |
    | Imagen | Seleccione **Windows Server 2019 Datacenter - Gen1** (o cualquier otra imagen de Windows que admita el entorno de ejecución de integración autohospedado). |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Valor |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione la red virtual que creó anteriormente. |
    | Subnet | Seleccione la subred creada anteriormente. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | **Basic**|
    | Puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione **Revisar + crear**. 
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

### <a name="create-the-private-endpoint"></a>Creación del punto de conexión privado 
Por último, debe crear el punto de conexión privado en la factoría de datos.

1. En la página de Azure Portal de la factoría de datos, seleccione la hoja **Redes** y la pestaña **Conexiones de punto de conexión privado** y, a continuación, seleccione **+ Punto de conexión privado**. 

:::image type="content" source="./media/data-factory-private-link/create-private-endpoint.png" alt-text="Captura de pantalla del panel &quot;Conexiones de punto de conexión privado&quot; para crear un punto de conexión privado.":::

2. En la pestaña **Aspectos básicos** de **Crear un punto de conexión privado**, escriba o seleccione esta información:

    | Parámetro | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Selección de un grupo de recursos |
    | **Detalles de instancia** |  |
    | Nombre  | Escriba un nombre para el punto de conexión. |
    | Region | Seleccione la región de la red virtual que creó anteriormente. |

3. Seleccione la pestaña **Recurso** o el botón **Siguiente: Recurso** en la parte inferior de la página.
    
4. En **Recurso**, escriba o seleccione esta información:

    | Parámetro | Value |
    | ------- | ----- |
    | Método de conexión | Seleccione **Conectarse a un recurso de Azure en mi directorio**. |
    | Subscription | Seleccione su suscripción. |
    | Tipo de recurso | Seleccione **Microsoft.Datafactory/factories**. |
    | Recurso | Seleccione su factoría de datos |
    | Recurso secundario de destino | Y si desea crear un punto de conexión privado para las comunicaciones de comandos entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory, seleccione **DataFactory** para **Subrecurso de destino**. Si desea usar un punto de conexión privado para la creación y supervisión de la factoría de datos en la red virtual, seleccione **Portal** para **Subrecurso de destino**.|

5. Seleccione la pestaña **Configuración** o el botón **Siguiente: Configuración** situado en la parte inferior de la pantalla.

6. En **Configuración**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **Redes** |  |
    | Virtual network | Seleccione la red virtual que creó anteriormente. |
    | Subnet | Seleccione la subred creada anteriormente. |
    | **Integración de DNS privado** |  |
    | Integración con una zona DNS privada | Deje el valor predeterminado de **Sí**. |
    | Subscription | Seleccione su suscripción. |
    | Zonas DNS privadas | Deje el valor predeterminado de **(Nuevo) privatelink.azurewebsites.net**.
    

7. Seleccione **Revisar + crear**.

8. Seleccione **Crear**.

> [!NOTE]
> Deshabilitar el acceso a la red pública solo es aplicable al entorno de ejecución de integración autohospedado, no a Azure Integration Runtime ni a SQL Server Integration Services (SSIS) Integration Runtime.

> [!NOTE]
> Puede seguir accediendo al portal de Azure Data Factory a través de la red pública después de crear el punto de conexión privado para el portal.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](quickstart-create-data-factory-portal.md)
- [Introducción al servicio Azure Data Factory](introduction.md)
- [Creación visual de Azure Data Factory](author-visually.md)
