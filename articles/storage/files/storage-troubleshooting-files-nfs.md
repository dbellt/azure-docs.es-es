---
title: 'Solución de problemas de recursos compartidos de archivos NFS de Azure: Azure Files'
description: Solución de problemas de recursos compartidos de archivos NFS de Azure
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 09/15/2020
ms.author: jeffpatt
ms.subservice: files
ms.custom: references_regions, devx-track-azurepowershell
ms.openlocfilehash: 6554fefe035267416a47af7e28270129b443a550
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110663316"
---
# <a name="troubleshoot-azure-nfs-file-shares"></a>Solución de problemas de recursos compartidos de archivos NFS de Azure

En este artículo se enumeran algunos problemas habituales relacionados con los recursos compartidos de archivos NFS de Azure. Se proporcionan las posibles causas y soluciones alternativas cuando se producen estos problemas.

## <a name="chgrp-filename-failed-invalid-argument-22"></a>Error en chgrp "nombre de archivo": argumento no válido (22)

### <a name="cause-1-idmapping-is-not-disabled"></a>Causa 1: idmapping no está deshabilitado
Azure Files no permite UID/GID alfanuméricos. Por lo tanto, idmapping debe estar deshabilitado. 

### <a name="cause-2-idmapping-was-disabled-but-got-re-enabled-after-encountering-bad-filedir-name"></a>Causa 2: idmapping estaba deshabilitado, pero se volvió a habilitar tras encontrar un nombre de archivo o de directorio incorrecto
Incluso si idmapping se ha deshabilitado correctamente, la configuración para deshabilitar idmapping se invalida en algunos casos. Por ejemplo, cuando Azure Files encuentra un nombre de archivo incorrecto, devuelve un error. Tras ver este código de error concreto, el cliente de Linux de NFS v4.1 decide volver a habilitar idmapping y las solicitudes futuras se envían de nuevo con UID/GID alfanuméricos. Para obtener una lista de los caracteres no admitidos en Azure Files, consulte este [artículo](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata). El carácter de los dos puntos es uno de los caracteres no admitidos. 

### <a name="workaround"></a>Solución alternativa
Compruebe que idmapping está deshabilitado y que nada lo vuelve a habilitar; luego, haga lo siguiente:

- Desmontaje del recurso compartido
- Deshabilite idmapping con # echo Y > /sys/module/nfs/parameters/nfs4_disable_idmapping.
- Vuelva a montar el recurso compartido.
- Si ejecuta rsync, hágalo con el argumento "—numeric-ids" desde un directorio que no tenga un nombre de directorio o archivo incorrecto.

## <a name="unable-to-create-an-nfs-share"></a>No se puede crear un recurso compartido de NFS

### <a name="cause-1-subscription-is-not-enabled"></a>Causa 1: La suscripción no está habilitada

Es posible que la suscripción no se haya registrado para la versión preliminar de NFS en Azure Files. Tendrá que ejecutar algunos commandlets adicionales desde Cloud Shell o desde un terminal local para habilitar la característica.

> [!NOTE]
> El registro puede tardar hasta 30 minutos en completarse.


#### <a name="solution"></a>Solución

Use el siguiente script para registrar la característica y el proveedor de recursos, y reemplace `<yourSubscriptionIDHere>` antes de ejecutar el script:

```azurepowershell
Connect-AzAccount

#If your identity is associated with more than one subscription, set an active subscription
$context = Get-AzSubscription -SubscriptionId <yourSubscriptionIDHere>
Set-AzContext $context

Register-AzProviderFeature -FeatureName AllowNfsFileShares -ProviderNamespace Microsoft.Storage

Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

### <a name="cause-2-unsupported-storage-account-settings"></a>Causa 2: Configuración no admitida de la cuenta de almacenamiento

NFS solo está disponible en las cuentas de almacenamiento con la siguiente configuración:

- Nivel: Prémium
- Tipo de cuenta: FileStorage
- Regiones: [lista de las regiones admitidas](./storage-files-how-to-create-nfs-shares.md?tabs=azure-portal#regional-availability)

#### <a name="solution"></a>Solución

Siga las instrucciones del artículo: [Procedimiento para crear un recurso compartido de NFS](storage-files-how-to-create-nfs-shares.md).

### <a name="cause-3-the-storage-account-was-created-prior-to-registration-completing"></a>Causa 3: La cuenta de almacenamiento se creó antes de que se completara el registro

Para que una cuenta de almacenamiento use la característica, debe crearse una vez que la suscripción haya completado el registro de NFS. El registro puede tardar hasta 30 minutos en completarse.

#### <a name="solution"></a>Solución

Una vez completado el registro, siga las instrucciones del artículo: [Procedimiento para crear un recurso compartido de NFS](storage-files-how-to-create-nfs-shares.md).

## <a name="cannot-connect-to-or-mount-an-azure-nfs-file-share"></a>No se puede conectar a un recurso compartido de archivos NFS de Azure ni montarlo

### <a name="cause-1-request-originates-from-a-client-in-an-untrusted-networkuntrusted-ip"></a>Causa 1: La solicitud se origina desde un cliente en una red o una IP que no son de confianza

A diferencia de SMB, la autenticación de NFS no se basa en el usuario. La autenticación de un recurso compartido se basa en la configuración de la regla de seguridad de red. Por eso, para tener la certeza de que solo se establecen conexiones seguras con su recurso compartido de NFS, debe usar el punto de conexión de servicio o puntos de conexión privados. Para acceder a recursos compartidos desde un entorno local, además de los puntos de conexión privados, debe configurar una VPN o ExpressRoute. Se omiten las direcciones IP agregadas a la lista de permitidos de la cuenta de almacenamiento para el firewall. Debe usar uno de los métodos siguientes para configurar el acceso a un recurso compartido de NFS:


- [Punto de conexión de servicio](storage-files-networking-endpoints.md#restrict-public-endpoint-access)
    - Se accede a través del punto de conexión público.
    - Solo está disponible en la misma región.
    - El emparejamiento de VNet no proporcionará acceso al recurso compartido.
    - Cada red virtual o subred debe agregarse individualmente a la lista de permitidos.
    - En el caso del acceso local, los puntos de conexión de servicio se pueden usar con ExpressRoute y las VPN de punto a sitio y de sitio a sitio, pero se recomienda usar un punto de conexión privado porque es más seguro.

En el diagrama siguiente se muestra la conectividad mediante puntos de conexión públicos.

:::image type="content" source="media/storage-troubleshooting-files-nfs/connectivity-using-public-endpoints.jpg" alt-text="Diagrama de conectividad mediante puntos de conexión públicos" lightbox="media/storage-troubleshooting-files-nfs/connectivity-using-public-endpoints.jpg":::

- [Punto de conexión privado](storage-files-networking-endpoints.md#create-a-private-endpoint)
    - El acceso es más seguro que con el punto de conexión de servicio.
    - El acceso al recurso compartido de NFS a través de un vínculo privado está disponible desde y hacia la región de Azure de la cuenta de almacenamiento (entre regiones o local).
    - El emparejamiento de red virtual con redes virtuales hospedadas en el punto de conexión privado proporciona acceso al recurso compartido de NFS a los clientes en las redes virtuales emparejadas.
    - Los puntos de conexión privados se pueden usar con ExpressRoute y las VPN de punto a sitio y de sitio a sitio.

:::image type="content" source="media/storage-troubleshooting-files-nfs/connectivity-using-private-endpoints.jpg" alt-text="Diagrama de conectividad mediante puntos de conexión privados" lightbox="media/storage-troubleshooting-files-nfs/connectivity-using-private-endpoints.jpg":::

### <a name="cause-2-secure-transfer-required-is-enabled"></a>Causa 2: La opción Se requiere transferencia segura está habilitada

Todavía no se admite el cifrado doble de los recursos compartidos de NFS. Azure proporciona una capa de cifrado para todos los datos en tránsito entre los centros de datos de Azure mediante MACSec. Solo se puede acceder a los recursos compartidos de NFS desde redes virtuales de confianza y a través de túneles VPN. No hay ningún cifrado de capa de transporte adicional disponible en los recursos compartidos de NFS.

#### <a name="solution"></a>Solución

Deshabilite Se requiere transferencia segura en la hoja Configuración de la cuenta de almacenamiento.

:::image type="content" source="media/storage-files-how-to-mount-nfs-shares/storage-account-disable-secure-transfer.png" alt-text="Captura de pantalla de la hoja Configuración de la cuenta de almacenamiento, en la que se deshabilita la opción Se requiere transferencia segura":::

### <a name="cause-3-nfs-common-package-is-not-installed"></a>Causa 3: el paquete nfs-common no está instalado
Antes de ejecutar el comando de montaje, instale el paquete mediante la ejecución del comando específico de la distribución que se muestra a continuación.

Para comprobar si el paquete NFS está instalado, ejecute: `rpm qa | grep nfs-utils`.

#### <a name="solution"></a>Solución

Si el paquete no está instalado, instálelo en la distribución.

##### <a name="ubuntu-or-debian"></a>Ubuntu o Debian

```
sudo apt update
sudo apt install nfs-common
```
##### <a name="fedora-red-hat-enterprise-linux-8-centos-8"></a>Fedora, Red Hat Enterprise Linux 8+ y CentOS 8+

Use el administrador de paquetes dnf: `sudo dnf install nfs-utils`.

En las versiones anteriores de Red Hat Enterprise Linux y CentOS, use el administrador de paquetes yum: `sudo yum install nfs-common`.

##### <a name="opensuse"></a>openSUSE

Use el administrador de paquetes zypper: `sudo zypper install-nfscommon`.

### <a name="cause-4-firewall-blocking-port-2049"></a>Causa 4: El firewall bloquea el puerto 2049

El protocolo NFS se comunica con su servidor a través del puerto 2049; asegúrese de que este puerto está abierto para la cuenta de almacenamiento (el servidor NFS).

#### <a name="solution"></a>Solución

Compruebe que el puerto 2049 está abierto en el cliente mediante la ejecución del siguiente comando: `telnet <storageaccountnamehere>.file.core.windows.net 2049`. Si el puerto no está abierto, ábralo.

## <a name="ls-list-files-shows-incorrectinconsistent-results"></a>ls (list files) muestra resultados incorrectos o incoherentes

### <a name="cause-inconsistency-between-cached-values-and-server-file-metadata-values-when-the-file-handle-is-open"></a>Causa: Incoherencia entre valores almacenados en caché y valores de metadatos de archivo de servidor cuando el identificador de archivo está abierto
A veces, el comando "list files" muestra un tamaño distinto de cero según lo esperado y, en su lugar, en los comandos list files siguientes muestra un tamaño de cero o una marca de tiempo muy antigua. Se trata de un problema conocido debido al almacenamiento en caché incoherente de los valores de metadatos de archivo mientras el archivo está abierto. Para resolver el problema, puede usar una de las siguientes soluciones alternativas:

#### <a name="workaround-1-for-fetching-file-size-use-wc--c-instead-of-ls--l"></a>Alternativa 1: Para obtener el tamaño de archivo, use wc -c en lugar de ls -l
El uso de wc -c siempre recuperará el valor más reciente del servidor y no tendrá ninguna incoherencia.

#### <a name="workaround-2-use-noac-mount-flag"></a>Alternativa 2: Use la marca de montaje "noac"
Vuelva a montar el sistema de archivos mediante la marca "noac" con el comando mount. De esta forma, siempre obtendrá todos los valores de metadatos del servidor. Puede haber cierta sobrecarga de rendimiento secundaria para todas las operaciones de metadatos si se usa esta solución alternativa.


## <a name="unable-to-mount-an-nfs-share-that-is-restored-back-from-soft-deleted-state"></a>No se puede montar un recurso compartido NFS que se restauró a partir del estado de eliminación temporal
Hay un problema conocido durante la versión preliminar en el que los recursos compartidos NFS se eliminan temporalmente a pesar de que la plataforma no lo admite por completo. Estos recursos compartidos se eliminarán de manera rutinaria al expirar. También puede eliminarlas anticipadamente mediante el flujo "recuperar recurso compartido + deshabilitar la eliminación temporal + eliminar recurso compartido". Sin embargo, si intenta recuperar y usar los recursos compartidos, se le denegará el acceso o el permiso, o se producirá un error de E/S de NFS en el cliente.

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.
Si sigue necesitando ayuda, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.
