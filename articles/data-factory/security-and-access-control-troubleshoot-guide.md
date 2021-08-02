---
title: Solución de problemas de seguridad y control de acceso
description: Aprenda a solucionar problemas de seguridad y control de acceso en Azure Data Factory.
author: lrtoyou1223
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 05/31/2021
ms.author: lle
ms.openlocfilehash: ff95f5c3f8d978d58146529825adee94f82eaf07
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110782899"
---
# <a name="troubleshoot-azure-data-factory-security-and-access-control-issues"></a>Solución de problemas de seguridad y control de acceso de Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran los métodos comunes de solución de problemas de seguridad y control de acceso en Azure Data Factory.

## <a name="common-errors-and-messages"></a>Errores habituales y mensajes

### <a name="connectivity-issue-in-the-copy-activity-of-the-cloud-datastore"></a>Problema de conectividad en la actividad de copia del almacén de datos en la nube

#### <a name="symptoms"></a>Síntomas

Es posible que se devuelvan varios mensajes de error cuando se produzcan problemas de conectividad en el almacén de datos de origen o receptor.

#### <a name="cause"></a>Causa 

En general, la causa del problema es uno de los siguientes factores:

* La configuración del proxy en el nodo del entorno de ejecución de integración (IR) autohospedado, si usa un IR autohospedado.

* La configuración del firewall en el nodo del IR autohospedado, si usa un IR autohospedado.

* La configuración del firewall en el almacén de datos en la nube.

#### <a name="resolution"></a>Solución

* Para asegurarse de que se trate de un problema de conectividad, compruebe los puntos siguientes:

   - El error se produce desde los conectores de origen o receptor.
   - El error se produce al principio de la actividad de copia.
   - El error es uniforme para Azure IR o el IR autohospedado con un nodo, porque podría tratarse de un error aleatorio en un IR autohospedado de varios nodos si solo algunos de los nodos tienen el problema.

* Si usa un **IR autohospedado**, compruebe la configuración del proxy, del firewall y de red, ya que la conexión al mismo almacén de datos puede realizarse correctamente si usa Azure IR. Para solucionar esta escenario, consulte:

   * [Puertos y firewalls de IR autohospedado](./create-self-hosted-integration-runtime.md#ports-and-firewalls)
   * [Conector de Azure Data Lake Storage](./connector-azure-data-lake-store.md)
  
* Si usa **Azure IR**, pruebe a deshabilitar la configuración de firewall del almacén de datos. Este enfoque puede resolver los problemas en las dos situaciones siguientes:
  
   * Las [direcciones IP de Azure IR](./azure-integration-runtime-ip-addresses.md) no se encuentran en la lista de permitidos.
   * La característica *Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento* está desactivada para [Azure Blob Storage](./connector-azure-blob-storage.md#supported-capabilities) y [Azure Data Lake Storage Gen2](./connector-azure-data-lake-storage.md#supported-capabilities).
   * La opción *Permitir el acceso a servicios de Azure* no está habilitada para Azure Data Lake Storage Gen1.

Si no funciona ninguno de los métodos anteriores, póngase en contacto con Microsoft para obtener ayuda.


### <a name="invalid-or-empty-authentication-key-issue-after-public-network-access-is-disabled"></a>Problema de clave de autenticación vacía o no válida después de deshabilitar el acceso a la red pública

#### <a name="symptoms"></a>Síntomas

Después de deshabilitar el acceso a la red pública para Data Factory, el entorno de ejecución de integración autohospedado produce el siguiente error: "La clave de autenticación no es válida o está vacía".

#### <a name="cause"></a>Causa

Lo más probable es que el problema se deba a una incidencia en la resolución del sistema de nombres de dominio (DNS), ya que la deshabilitación de la conectividad pública y el establecimiento de un punto de conexión privado impide la reconexión.

Para comprobar si el nombre de dominio completo (FQDN) de Data Factory se resuelve en la dirección IP pública, haga lo siguiente:

1. Confirme que ha creado la máquina virtual (VM) de Azure en la misma red virtual que el punto de conexión privado de Data Factory.

2. Ejecute PsPing y Ping desde la VM de Azure hasta el FQDN de Data Factory:

   `psping.exe <dataFactoryName>.<region>.datafactory.azure.net:443`
   `ping <dataFactoryName>.<region>.datafactory.azure.net`

   > [!Note]
   > Debe especificar un puerto para el comando PsPing. Aquí se muestra el puerto 443, pero no es obligatorio.

3. Compruebe si ambos comandos se resuelven en una dirección IP pública de Azure Data Factory que se base en una región especificada. La dirección IP debe tener el siguiente formato: `xxx.xxx.xxx.0`.

#### <a name="resolution"></a>Solución

Para solucionar el problema, haga lo siguiente:

- Como opción, nos gustaría sugerirle que agregue manualmente un "vínculo de Virtual Network" en la "zona DNS de vínculo privado" de Data Factory. Para más información, consulte el artículo [Azure Private Link para Azure Data Factory](./data-factory-private-link.md#dns-changes-for-private-endpoints). La instrucción es para configurar el servidor DNS personalizado o la zona DNS privada para resolver el FQDN de Data Factory en una dirección IP privada. 

- Sin embargo, si no quiere configurar ninguno de estos, pruebe la siguiente solución temporal:

  1. Cambie el archivo de host en Windows y asigne la dirección IP privada (el punto de conexión privado de Azure Data Factory) al FQDN de Azure Data Factory.
  
     En la VM de Azure, vaya a `C:\Windows\System32\drivers\etc` y, a continuación, abra el archivo *host* en el Bloc de notas. Agregue la línea que asigna la dirección IP privada al FQDN al final del archivo y guarde el cambio.
     
     ![Captura de pantalla de la asignación de la dirección IP privada al host.](media/self-hosted-integration-runtime-troubleshoot-guide/add-mapping-to-host.png)

  1. Vuelva a ejecutar los mismos comandos que en los pasos de comprobación anteriores para verificar la respuesta, que debe contener la dirección IP privada.

  1. Vuelva a registrar el entorno de ejecución de integración autohospedado, el problema debería haberse resuelto.

### <a name="unable-to-register-ir-authentication-key-on-self-hosted-vms-due-to-private-link"></a>No se puede registrar la clave de autenticación del entorno de ejecución de integración en máquinas virtuales autohospedadas debido a un vínculo privado

#### <a name="symptoms"></a>Síntomas

No se puede registrar la clave de autenticación de IR en la VM autohospedada porque la conexión privada está habilitada. Aparece el siguiente mensaje de error:

"No se pudo obtener el token de servicio del servicio de ADF con la clave *************** y un costo de tiempo de 0,1250079 segundos. Código de error: InvalidGatewayKey; activityId: XXXXXXX; mensaje de error detallado: Client IP address is not valid private ip Cause Data factory couldn’t access the public network thereby not able to reach out to the cloud to make the successful connection" (La dirección IP de cliente no es una dirección IP privada válida porque Data Factory no pudo acceder a la red pública, por lo que no se puede comunicar con la nube para establecer la conexión correcta).

#### <a name="cause"></a>Causa

El problema puede deberse a la VM en la que está intentando instalar el IR autohospedado. Para conectarse a la nube, asegúrese de que está habilitado el acceso a la red pública.

#### <a name="resolution"></a>Solución

**Solución 1**
 
Para solucionar el problema, haga lo siguiente:

1. Vaya a la página [Factories - Update](/rest/api/datafactory/Factories/Update).

1. En la esquina superior derecha, seleccione el botón **Pruébelo**.
1. En **Parámetros**, complete la información necesaria. 
1. En **Cuerpo**, pegue la siguiente propiedad:

    ```
    { "tags": { "publicNetworkAccess":"Enabled" } }
    ```
1. Seleccione **Ejecutar** para ejecutar la función. 

1. En **Parámetros**, complete la información necesaria. 

1. En **Cuerpo**, pegue la siguiente propiedad:
    ```
    { "tags": { "publicNetworkAccess":"Enabled" } }
    ``` 

1. Seleccione **Ejecutar** para ejecutar la función. 
1. Confirme que aparezca **Código de respuesta: 200**. La propiedad que pegó también debe aparecer en la definición de JSON.

1. Agregue de nuevo la clave de autenticación del IR al entorno de ejecución de integración.

**Solución 2**

Para resolver el problema, vaya a [Azure Private Link para Azure Data Factory](./data-factory-private-link.md).

Pruebe a habilitar el acceso a la red pública en la interfaz de usuario, como se muestra en la siguiente captura de pantalla:

![Captura de pantalla del control "Habilitado" para "Permitir el acceso de red público" en el panel Redes.](media/self-hosted-integration-runtime-troubleshoot-guide/enable-public-network-access.png)

### <a name="adf-private-dns-zone-overrides-azure-resource-manager-dns-resolution-causing-not-found-error"></a>La zona DNS privada de ADF invalida la resolución DNS de Azure Resource Manager y causa el error "No encontrado"

#### <a name="cause"></a>Causa
Tanto Azure Resource Manager como ADF usan la misma zona privada, lo que crea un posible conflicto en el DNS privado del cliente con un escenario en el que no se encontrarán los registros de Azure Resource Manager.

#### <a name="solution"></a>Solución
1. Busque las zonas DNS privadas **privatelink.Azure.com** en Azure Portal.
![Captura de pantalla de la búsqueda de zonas DNS privadas.](media/security-access-control-troubleshoot-guide/private-dns-zones.png)
2. Compruebe si hay un registro A **adf**.
![Captura de pantalla de un registro A.](media/security-access-control-troubleshoot-guide/a-record.png)
3.  Vaya a **Vínculos de red virtual** y elimine todos los registros.
![Captura de pantalla del vínculo de red virtual.](media/security-access-control-troubleshoot-guide/virtual-network-link.png)
4.  Vaya a la factoría de datos en Azure Portal y vuelva a crear el punto de conexión privado para el portal de Azure Data Factory.
![Captura de pantalla de la recreación del punto de conexión privado.](media/security-access-control-troubleshoot-guide/create-private-endpoint.png)
5.  Vuelva a las zonas DNS privadas y compruebe si hay una nueva zona DNS privada **privatelink.adf.azure.com**.
![Captura de pantalla del nuevo registro DNS.](media/security-access-control-troubleshoot-guide/check-dns-record.png)

### <a name="connection-error-in-public-endpoint"></a>Error de conexión en el punto de conexión público

#### <a name="symptoms"></a>Síntomas

Al copiar datos con la cuenta de acceso público de Azure Blob Storage, el siguiente error se produce de forma aleatoria en las ejecuciones de canalización.

Por ejemplo: en el receptor de Azure Blob Storage se usaba Azure IR (red virtual pública y no administrada) y en el origen de Azure SQL Database IR de red virtual administrada. O bien en el origen o el receptor solo se usa IR de red virtual administrada con acceso público de almacenamiento.

`
<LogProperties><Text>Invoke callback url with req:
"ErrorCode=UserErrorFailedToCreateAzureBlobContainer,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Unable to create Azure Blob container. Endpoint: XXXXXXX/, Container Name: test.,Source=Microsoft.DataTransfer.ClientLibrary,''Type=Microsoft.WindowsAzure.Storage.StorageException,Message=Unable to connect to the remote server,Source=Microsoft.WindowsAzure.Storage,''Type=System.Net.WebException,Message=Unable to connect to the remote server,Source=System,''Type=System.Net.Sockets.SocketException,Message=A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond public ip:443,Source=System,'","Details":null}}</Text></LogProperties>.
`

#### <a name="cause"></a>Causa

ADF todavía puede usar IR de red virtual administrada, pero podría encontrar este error porque el punto de conexión público para Azure Blob Storage en la red virtual administrada no es confiable según el resultado de la prueba, y no se admiten Azure Blob Storage ni Azure Data Lake Gen2 para conectarse mediante un punto de conexión público de la red virtual administrada de ADF, de acuerdo a [Red virtual administrada y puntos de conexión privados administrados](./managed-virtual-network-private-endpoint.md#outbound-communications-through-public-endpoint-from-adf-managed-virtual-network).

#### <a name="solution"></a>Solución

- Habilite el punto de conexión privado en el origen y también el lado del receptor cuando se use IR de la red virtual administrada.
- Si todavía quiere usar el punto de conexión público, puede cambiar a solo IR público en lugar de usar IR de red virtual administrada para el origen y el receptor. Incluso si cambia de nuevo a IR público, ADF todavía puede usar IR de red virtual administrada en caso de que siga presente.

## <a name="sharing-self-hosted-integration-runtime"></a>Uso compartido de IR autohospedado

### <a name="sharing-a-self-hosted-ir-from-a-different-tenant-is-not-supported"></a>No se admite el uso compartido de un IR autohospedado desde un inquilino diferente 

#### <a name="symptoms"></a>Síntomas

Es posible que observe otras factorías de datos (en diferentes inquilinos) al intentar compartir el IR autohospedado desde la interfaz de usuario de Azure Data Factory, pero no puede compartirlo entre factorías de datos que se encuentran en distintos inquilinos.

#### <a name="cause"></a>Causa

El IR autohospedado no puede compartirse entre inquilinos.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda para solucionar problemas, pruebe los siguientes recursos:

*  [Private Link para Data Factory](data-factory-private-link.md)
*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)