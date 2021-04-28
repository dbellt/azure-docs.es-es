---
author: LittleBoxOfSunshine
manager: KumariSupriya
ms.service: virtual-machines
ms.subservice: monitoring
ms.topic: include
ms.date: 01/04/2021
ms.author: chhenk
ms.reviewer: azmetadatadev
ms.custom: references_regions
ms.openlocfilehash: 98866a4f06df0380d52d1aee3eede8aa2f70aaed
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588143"
---
Azure Instance Metadata Service (IMDS) le proporciona información sobre las instancias de máquina virtual que se ejecutan actualmente. Puede usarlo para administrar y configurar las máquinas virtuales.
Esta información incluye las SKU, almacenamiento, configuraciones de red y próximos eventos de mantenimiento. Para ver una lista completa de los datos disponibles, consulte el [resumen de categorías de los puntos de conexión](#endpoint-categories).

IMDS está disponible para ejecutar instancias de máquinas virtuales (VM) e instancias de un conjunto de escalado de máquinas virtuales. Todos los puntos de conexión admiten máquinas virtuales creadas y administradas mediante [Azure Resource Manager](/rest/api/resources/). Solo la categoría de atestiguados y la parte de red de la categoría de instancias admiten las máquinas virtuales creadas mediante el modelo de implementación clásica. El punto de conexión de atestiguados solo lo hace de forma limitada.

IMDS es una API REST disponible en una dirección IP conocida y no enrutable (`169.254.169.254`). Solo puede acceder a ella desde la máquina virtual. La comunicación entre la VM e IMDS nunca sale del host.
Puede sugerir a sus clientes HTTP que omitan los servidores proxy web dentro de la VM al consultar a IMDS y que consideren que `169.254.169.254` es lo mismo que [`168.63.129.16`](../articles/virtual-network/what-is-ip-address-168-63-129-16.md).

## <a name="usage"></a>Uso

### <a name="access-azure-instance-metadata-service"></a>Acceso a Azure Instance Metadata Service

Para obtener acceso a IMDS, cree una VM desde [Azure Resource Manager](/rest/api/resources/) o [Azure Portal](https://portal.azure.com) y siga los ejemplos que tiene a continuación.
Para obtener más ejemplos, consulte los [Ejemplos de Azure Instance Metadata](https://github.com/microsoft/azureimds).

Puede usar este código de ejemplo para recuperar todos los metadatos de una instancia. Para acceder a un origen de datos específico, consulte [Categorías de punto de conexión](#endpoint-categories), con información general de todas las características disponibles.

**Solicitud**

> [!IMPORTANT]
> En este ejemplo se omiten los servidores proxy. **Debe** omitir los servidores proxy al consultar IMDS. Consulte [Servidores proxy](#proxies) para más información.

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq
```

---

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

[!INCLUDE [virtual-machines-imds-full-instance-response](./virtual-machines-imds-full-instance-response.md)]

## <a name="security-and-authentication"></a>Seguridad y autenticación

Instance Metadata Service solo es accesible desde la instancia de máquina virtual en ejecución en una dirección IP no enrutable. Las máquinas virtuales se limitan a interactuar con los metadatos y la funcionalidad propios. La API es solo HTTP y nunca sale del host.

Para asegurarse de que las solicitudes se destinan directamente a IMDS y evitar la redirección imprevista o no deseada de las solicitudes, estas:
- **Deben** contener el encabezado `Metadata: true`.
- **No** deben contener un encabezado `X-Forwarded-For`.

El servicio rechazará cualquier solicitud que no cumpla **ambos** requisitos.

> [!IMPORTANT]
> IMDS **no** es un canal para datos confidenciales. La API no está autenticada y está abierta a todos los procesos de la máquina virtual. La información expuesta a través de este servicio se debe considerar como información compartida con todas las aplicaciones que se ejecutan dentro de la máquina virtual.

## <a name="proxies"></a>Servidores proxy

IMDS **no** está diseñado para el uso detrás de un proxy y no se admite. La mayoría de los clientes HTTP proporcionan una opción para deshabilitar los servidores proxy en las solicitudes; esta funcionalidad se debe usar en la comunicación con IMDS. Consulte la documentación del cliente para información detallada.

> [!IMPORTANT]
> Aunque no tenga configuración de proxy en su entorno, **debe invalidar la configuración de proxy de cliente predeterminada**. Las configuraciones de proxy se pueden detectar automáticamente y, si no se omite este tipo de configuración, quedará expuesto a abusos si la configuración de la máquina se cambia en el futuro.

## <a name="rate-limiting"></a>Limitación de frecuencia

En general, las solicitudes a IMDS se limitan a cinco por segundo. Las solicitudes que superen este umbral se rechazarán con respuestas 429. Las solicitudes a la categoría [Identidad administrada](#managed-identity) se limitan a 20 por segundo y cinco solicitudes simultáneas.

## <a name="http-verbs"></a>Verbos HTTP

Actualmente se admiten los verbos HTTP siguientes:

| Verbo | Descripción |
|------|-------------|
| `GET` | Recupera el recurso solicitado.

## <a name="parameters"></a>Parámetros

Los puntos de conexión pueden admitir parámetros obligatorios u opcionales. Consulte [Esquema](#schema) y la documentación del punto de conexión en cuestión para más información.

### <a name="query-parameters"></a>Parámetros de consulta

Los puntos de conexión de IMDS admiten parámetros de cadena de consulta HTTP. Por ejemplo: 

```
http://169.254.169.254/metadata/instance/compute?api-version=2019-06-04&format=json
```

Especifica los parámetros:

| Nombre | Value |
|------|-------|
| `api-version` | `2019-06-04`
| `format` | `json`

Se rechazarán las solicitudes con nombres de parámetro de consulta duplicados.

### <a name="route-parameters"></a>Parámetros de ruta

En algunos puntos de conexión que devuelven blobs JSON más grandes, se admite anexar parámetros de ruta al punto de conexión de solicitud para filtrar hasta un subconjunto de la respuesta: 

```
http://169.254.169.254/metadata/<endpoint>/[<filter parameter>/...]?<query parameters>
```
Los parámetros corresponden a los índices o claves que se utilizarían para recorrer el objeto JSON con el que interactuara con una representación analizada.

Por ejemplo, `/metatadata/instance` devuelve el objeto JSON:
```json
{
    "compute": { ... },
    "network": {
        "interface": [
            {
                "ipv4": {
                   "ipAddress": [{
                        "privateIpAddress": "10.144.133.132",
                        "publicIpAddress": ""
                    }],
                    "subnet": [{
                        "address": "10.144.133.128",
                        "prefix": "26"
                    }]
                },
                "ipv6": {
                    "ipAddress": [
                     ]
                },
                "macAddress": "0011AAFFBB22"
            },
            ...
        ]
    }
}
```

Si queremos filtrar la respuesta solo hasta la propiedad de proceso, enviaríamos la solicitud: 
```
http://169.254.169.254/metadata/instance/compute?api-version=<version>
```

Del mismo modo, si queremos filtrar hasta una propiedad anidada o un elemento de matriz específico, seguiremos anexando claves: 
```
http://169.254.169.254/metadata/instance/network/interface/0?api-version=<version>
```
filtraría hasta el primer elemento de la propiedad `Network.interface` y devolvería:

```json
{
    "ipv4": {
       "ipAddress": [{
            "privateIpAddress": "10.144.133.132",
            "publicIpAddress": ""
        }],
        "subnet": [{
            "address": "10.144.133.128",
            "prefix": "26"
        }]
    },
    "ipv6": {
        "ipAddress": [
         ]
    },
    "macAddress": "0011AAFFBB22"
}
```

> [!NOTE]
> Al filtrar hasta un nodo hoja, `format=json` no funciona. Para estas consultas, `format=text` debe especificarse explícitamente ya que el formato predeterminado es JSON.

## <a name="schema"></a>Schema

### <a name="data-format"></a>Formato de datos

De forma predeterminada, IMDS devuelve datos en formato JSON (`Content-Type: application/json`). Sin embargo, los puntos de conexión que admiten el filtrado de respuestas (consulte [Parámetros de ruta](#route-parameters)) también admiten el formato `text`.

Para acceder a un formato de respuesta que no sea el predeterminado, especifique el formato solicitado como un parámetro de cadena de consulta en la solicitud. Por ejemplo:

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

---

En las respuestas JSON, todos los primitivos serán del tipo `string` y los valores que falten o que no sean aplicables se incluirán siempre, pero se establecerán en una cadena vacía.

### <a name="versioning"></a>Control de versiones

IMDS tiene varias versiones y es obligatorio especificar la versión de la API en la solicitud HTTP. La única excepción a este requisito es el punto de conexión de [versiones](#versions), que se puede usar para recuperar dinámicamente las versiones de API disponibles.

A medida que se agreguen versiones más recientes, todavía se podrá acceder a las versiones anteriores por motivos de compatibilidad si los scripts tienen dependencias en formatos de datos específicos.

Cuando no especifique la versión, obtendrá un error con una lista de las versiones admitidas más recientes.
```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2020-10-01",
        "2020-09-01",
        "2020-07-15"
    ]
}
```

#### <a name="supported-api-versions"></a>Versiones de API admitidas
- 2017-03-01
- 2017-04-02
- 2017-08-01 
- 2017-10-01
- 2017-12-01 
- 2018-02-01
- 2018-04-02
- 2018-10-01
- 01-02-2019
- 2019-03-11
- 2019-04-30
- 2019-06-01
- 2019-06-04
- 2019-08-01
- 2019-08-15
- 2019-11-01
- 2020-06-01
- 2020-07-15
- 01-09-2020
- 2020-10-01
- 2020-12-01
- 01-01-2021

### <a name="swagger"></a>Swagger

Hay una definición completa de Swagger para IMDS disponible en https://github.com/Azure/azure-rest-api-specs/blob/master/specification/imds/data-plane/readme.md.

## <a name="regional-availability"></a>Disponibilidad regional

El servicio está **disponible con carácter general** en todas las nubes de Azure.

## <a name="root-endpoint"></a>Punto de conexión raíz

El punto de conexión raíz es `http://169.254.169.254/metadata`.

## <a name="endpoint-categories"></a>Categorías de punto de conexión

La API de IMDS contiene varias categorías de punto de conexión que representan distintos orígenes de datos, cada uno de los cuales contiene uno o varios puntos de conexión. Consulte las categorías para más información.

| Raíz de la categoría | Descripción | Versión introducida |
|---------------|-------------|--------------------|
| `/metadata/attested` | Consulte [Datos atestiguados](#attested-data) | 2018-10-01
| `/metadata/identity` | Consulte [Identidad administrada mediante IMDS](#managed-identity). | 2018-02-01
| `/metadata/instance` | Consulte [Metadatos de instancia](#instance-metadata). | 2017-04-02
| `/metadata/loadbalancer` | Consulte [Recuperación de metadatos de Load Balancer a través de IMDS](#load-balancer-metadata). | 2020-10-01
| `/metadata/scheduledevents` | Consulte [Eventos programados mediante IMDS](#scheduled-events). | 2017-08-01
| `/metadata/versions` | Consulte [Versiones](#versions). | N/D

## <a name="versions"></a>Versiones

> [!NOTE]
> Esta característica se lanzó junto con la versión 2020-10-01, que se está implementando actualmente y puede que aún no esté disponible en todas las regiones.

### <a name="list-api-versions"></a>Enumeración de las versiones de API

Devuelve el conjunto de versiones de API admitidas.

```
GET /metadata/versions
```

#### <a name="parameters"></a>Parámetros

Ninguno (este punto de conexión no tiene versión).

#### <a name="response"></a>Response

```json
{
  "apiVersions": [
    "2017-03-01",
    "2017-04-02",
    ...
  ]
}
```

## <a name="instance-metadata"></a>Metadatos de instancia

### <a name="get-vm-metadata"></a>Obtención de los metadatos de una máquina virtual

Expone los metadatos importantes de las instancias de máquina virtual, como el proceso, la red y el almacenamiento. 

```
GET /metadata/instance
```

#### <a name="parameters"></a>Parámetros

| Nombre | Obligatorio/opcional | Descripción |
|------|-------------------|-------------|
| `api-version` | Obligatorio | Versión usada para atender la solicitud.
| `format` | Opcional* | Formato de la respuesta (`json` o `text`). *Nota: Puede ser necesario al usar parámetros de solicitud.

Este punto de conexión admite el filtrado de respuestas mediante [parámetros de ruta](#route-parameters).

#### <a name="response"></a>Response

[!INCLUDE [virtual-machines-imds-full-instance-response](./virtual-machines-imds-full-instance-response.md)]

Desglose del esquema:

**Proceso**

| data | Descripción | Versión introducida |
|------|-------------|--------------------|
| `azEnvironment` | Entorno de Azure donde se está ejecutando la máquina virtual | 2018-10-01
| `customData` | Esta característica está en desuso y deshabilitada [en IMDS](#frequently-asked-questions). Se ha reemplazado por `userData` | 01-02-2019
| `evictionPolicy` | Establece cómo se expulsará una [máquina virtual de acceso puntual](../articles/virtual-machines/spot-vms.md). | 2020-12-01
| `isHostCompatibilityLayerVm` | Identifica si la máquina virtual se ejecuta en el nivel de compatibilidad del host. | 2020-06-01
| `licenseType` | Tipo de licencia para la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit). Esta opción solo está presente en las máquinas virtuales habilitadas para Ventaja híbrida de Azure. | 01-09-2020
| `location` | La región de Azure donde se ejecuta la máquina virtual | 2017-04-02
| `name` | Nombre de la máquina virtual | 2017-04-02
| `offer` | Ofrece información de la imagen de la máquina virtual y solo está presente para imágenes implementadas desde la galería de imágenes de Azure. | 2017-04-02
| `osProfile.adminUsername` | Especifica el nombre de la cuenta de administrador. | 2020-07-15
| `osProfile.computerName` | Especifica el nombre del equipo. | 2020-07-15
| `osProfile.disablePasswordAuthentication` | Especifica si se deshabilita la autenticación de contraseña. Esta opción solo está presente en las máquinas virtuales Linux. | 2020-10-01
| `osType` | Linux o Windows | 2017-04-02
| `placementGroupId` | [Grupo de selección de ubicación](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md) de su conjunto de escalado de máquina virtual | 2017-08-01
| `plan` | [Plan](/rest/api/compute/virtualmachines/createorupdate#plan) que contiene el nombre, el producto y el editor de una máquina virtual si es una imagen de Azure Marketplace | 2018-04-02
| `platformUpdateDomain` |  El [dominio de actualización](../articles/virtual-machines/availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
| `platformFaultDomain` | El [dominio de error](../articles/virtual-machines/availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
| `priority` | Prioridad de la máquina virtual. Consulte [Máquinas virtuales de acceso puntual](../articles/virtual-machines/spot-vms.md) para más información | 2020-12-01
| `provider` | Proveedor de la máquina virtual | 2018-10-01
| `publicKeys` | [Colección de claves públicas](/rest/api/compute/virtualmachines/createorupdate#sshpublickey) asignada a la máquina virtual y rutas de acceso | 2018-04-02
| `publisher` | Publicador de la imagen de VM | 2017-04-02
| `resourceGroupName` | [Grupo de recursos](../articles/azure-resource-manager/management/overview.md) para su máquina virtual | 2017-08-01
| `resourceId` | El identificador [completo](/rest/api/resources/resources/getbyid) del recurso | 2019-03-11
| `sku` | SKU específica de la imagen de VM | 2017-04-02
| `securityProfile.secureBootEnabled` | Identifica si el arranque seguro UEFI está habilitado en la máquina virtual. | 2020-06-01
| `securityProfile.virtualTpmEnabled` | Identifica si el Módulo de plataforma segura (TPM) virtual está habilitado en la máquina virtual. | 2020-06-01
| `storageProfile` | Consulte el perfil de almacenamiento a continuación. | 2019-06-01
| `subscriptionId` | Suscripción de Azure para la máquina virtual | 2017-08-01
| `tags` | [Etiquetas](../articles/azure-resource-manager/management/tag-resources.md) para su máquina virtual  | 2017-08-01
| `tagsList` | Etiquetas con formato de matriz de JSON para facilitar el análisis mediante programación  | 2019-06-04
| `userData` | Conjunto de datos especificado cuando se creó la máquina virtual para su uso durante o después del aprovisionamiento (codificado en Base64)  | 01-01-2021
| `version` | Versión de la imagen de máquina virtual | 2017-04-02
| `vmId` | [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual. | 2017-04-02
| `vmScaleSetName` | [Nombre del conjunto de escalado de máquina virtual](../articles/virtual-machine-scale-sets/overview.md) del conjunto de escalado de la máquina virtual | 2017-12-01
| `vmSize` | [Tamaño de VM](../articles/virtual-machines/sizes.md) | 2017-04-02
| `zone` | [Zona de disponibilidad](../articles/availability-zones/az-overview.md) de la máquina virtual | 2017-12-01

**Perfil de almacenamiento**

El perfil de almacenamiento de una máquina virtual se divide en tres categorías: referencia de imagen, disco del sistema operativo y discos de datos.

El objeto de referencia de imagen contiene la información siguiente sobre la imagen del sistema operativo:

| data | Descripción |
|------|-------------|
| `id` | Id. de recurso
| `offer` | Oferta de la plataforma o imagen de Marketplace
| `publisher` | Editor de la imagen
| `sku` | SKU de la imagen
| `version` | Versión de la plataforma o imagen de Marketplace

El objeto de disco del sistema operativo contiene la información siguiente sobre el disco del sistema operativo que usa la máquina virtual:

| data | Descripción |
|------|-------------|
| `caching` | Requisitos de almacenamiento en caché
| `createOption` | Información sobre cómo se ha creado la máquina virtual
| `diffDiskSettings` | Configuración de discos efímeros
| `diskSizeGB` | Tamaño del disco en GB
| `image`   | Disco duro virtual de la imagen de usuario de origen
| `lun`     | Número de unidad lógica del disco
| `managedDisk` | Parámetros de disco administrado
| `name`    | Nombre del disco
| `vhd`     | Disco duro virtual
| `writeAcceleratorEnabled` | Si writeAccelerator está habilitado o no en el disco

La matriz de discos de datos contiene una lista de discos de datos conectados a la máquina virtual. Cada objeto de disco de datos contiene la información siguiente:

data | Descripción |
-----|-------------|
| `caching` | Requisitos de almacenamiento en caché
| `createOption` | Información sobre cómo se ha creado la máquina virtual
| `diffDiskSettings` | Configuración de discos efímeros
| `diskSizeGB` | Tamaño del disco en GB
| `encryptionSettings` | Configuración de cifrado para el disco
| `image` | Disco duro virtual de la imagen de usuario de origen
| `managedDisk` | Parámetros de disco administrado
| `name` | Nombre del disco
| `osType` | Tipo de sistema operativo incluido en el disco
| `vhd` | Disco duro virtual
| `writeAcceleratorEnabled` | Si writeAccelerator está habilitado o no en el disco

**Network**

| data | Descripción | Versión introducida |
|------|-------------|--------------------|
| `ipv4.privateIpAddress` | Dirección IPv4 local de la máquina virtual | 2017-04-02
| `ipv4.publicIpAddress` | Dirección IPv4 pública de la máquina virtual | 2017-04-02
| `subnet.address` | Dirección de subred de la máquina virtual | 2017-04-02
| `subnet.prefix` | Prefijo de la subred, ejemplo, 24 | 2017-04-02
| `ipv6.ipAddress` | Dirección IPv6 local de la máquina virtual | 2017-04-02
| `macAddress` | Dirección de MAC de la VM | 2017-04-02

### <a name="get-user-data"></a>Obtención de los datos del usuario

Al crear una nueva máquina virtual, puede especificar un conjunto de datos que se usarán durante o después del aprovisionamiento de la máquina virtual y recuperarlos a través de IMDS. Para configurar los datos de usuario, use la plantilla de inicio rápido [aquí](https://aka.ms/ImdsUserDataArmTemplate). En el ejemplo siguiente se muestra cómo recuperar estos datos a través de IMDS.

> [!NOTE]
> Esta característica se lanza con la versión `2021-01-01` y depende de una actualización de la plataforma Azure, que actualmente se está implantando y es posible que aún no esté disponible en todas las regiones.

> [!NOTE]
> Aviso de seguridad: IMDS está abierto a todas las aplicaciones de la máquina virtual; no se deben colocar datos confidenciales en los datos de usuario.


#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text" | base64 --decode
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text" | base64 --decode
```

---


#### <a name="sample-1-tracking-vm-running-on-azure"></a>Muestra 1: Seguimiento de una máquina virtual que se ejecuta en Azure

Como proveedor de servicios, es posible que necesite hacer seguimiento de la cantidad de máquinas virtuales que ejecutan su software o que tenga agentes que deban hacer seguimiento de la unicidad de la máquina virtual. Para poder obtener un identificador único para una máquina virtual, use el campo `vmId` del servicio de metadatos de instancia.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

---

**Respuesta**

```
5c08b38e-4d57-4c23-ac45-aca61037f084
```

#### <a name="sample-2-placement-of-different-data-replicas"></a>Ejemplo 2: ubicación de diferentes réplicas de datos

Para ciertos escenarios, la ubicación de las distintas réplicas de datos es de máxima importancia. Por ejemplo, para la [ubicación de réplicas de HDFS](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps) o la ubicación de contenedores a través de un [orquestador](https://kubernetes.io/docs/user-guide/node-selection/), debe saber en qué `platformFaultDomain` y `platformUpdateDomain` se ejecuta la VM.
También puede usar las [zonas de disponibilidad](../articles/availability-zones/az-overview.md) para las instancias para tomar estas decisiones.
Puede consultar estos datos directamente a través de IMDS.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

---

**Respuesta**

```
0
```

#### <a name="sample-3-get-vm-tags"></a>Ejemplo 3: Obtención de etiquetas de máquina virtual

Las etiquetas de máquina virtual se incluyen en la API de instancia en el punto de conexión instance/compute/tags.
Es posible que se hayan aplicado etiquetas a las máquinas virtuales de Azure para organizarlas de forma lógica en una taxonomía. Las etiquetas asignadas a una máquina virtual se pueden recuperar mediante esta solicitud.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/tags?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

---

**Respuesta**

```
Department:IT;ReferenceNumber:123456;TestStatus:Pending
```

El campo `tags` es una cadena con las etiquetas delimitadas por puntos y coma. Esta salida puede ser un problema si se usan puntos y coma en las propias etiquetas. Si se escribe un analizador para extraer mediante programación las etiquetas, debe basarse en el campo `tagsList`. El campo `tagsList` es una matriz JSON sin delimitadores y, por tanto, es más fácil de analizar. La lista de etiquetas asignadas a una máquina virtual se puede recuperar mediante esta solicitud.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04" | jq
```

---

**Respuesta**

#### <a name="windows"></a>[Windows](#tab/windows/)

```json
{
    "value":  [
                  {
                      "name":  "Department",
                      "value":  "IT"
                  },
                  {
                      "name":  "ReferenceNumber",
                      "value":  "123456"
                  },
                  {
                      "name":  "TestStatus",
                      "value":  "Pending"
                  }
              ],
    "Count":  3
}
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```json
[
  {
    "name": "Department",
    "value": "IT"
  },
  {
    "name": "ReferenceNumber",
    "value": "123456"
  },
  {
    "name": "TestStatus",
    "value": "Pending"
  }
]
```

---


#### <a name="sample-4-get-more-information-about-the-vm-during-support-case"></a>Ejemplo 4: Obtención de más información sobre la máquina virtual durante el caso de soporte técnico

Como proveedor de servicios, es posible que reciba una llamada de soporte técnico en la que le gustaría tener más información sobre la máquina virtual. Pedirle al cliente que comparta los metadatos del equipo puede proporcionar información básica para que el profesional de soporte técnico conozca la variante de máquina virtual en Azure.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01"
```

---

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

#### <a name="windows"></a>[Windows](#tab/windows/)
```json
{
    "azEnvironment": "AZUREPUBLICCLOUD",
    "isHostCompatibilityLayerVm": "true",
    "licenseType":  "Windows_Client",
    "location": "westus",
    "name": "examplevmname",
    "offer": "WindowsServer",
    "osProfile": {
        "adminUsername": "admin",
        "computerName": "examplevmname",
        "disablePasswordAuthentication": "true"
    },
    "osType": "Windows",
    "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
    "plan": {
        "name": "planName",
        "product": "planProduct",
        "publisher": "planPublisher"
    },
    "platformFaultDomain": "36",
    "platformUpdateDomain": "42",
    "publicKeys": [{
            "keyData": "ssh-rsa 0",
            "path": "/home/user/.ssh/authorized_keys0"
        },
        {
            "keyData": "ssh-rsa 1",
            "path": "/home/user/.ssh/authorized_keys1"
        }
    ],
    "publisher": "RDFE-Test-Microsoft-Windows-Server-Group",
    "resourceGroupName": "macikgo-test-may-23",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
    "securityProfile": {
        "secureBootEnabled": "true",
        "virtualTpmEnabled": "false"
    },
    "sku": "2019-Datacenter",
    "storageProfile": {
        "dataDisks": [{
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
                "uri": ""
            },
            "lun": "0",
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }],
        "imageReference": {
            "id": "",
            "offer": "WindowsServer",
            "publisher": "MicrosoftWindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
        },
        "osDisk": {
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "diskSizeGB": "30",
            "diffDiskSettings": {
                "option": "Local"
            },
            "encryptionSettings": {
                "enabled": "false"
            },
            "image": {
                "uri": ""
            },
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampleosdiskname",
            "osType": "Windows",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "baz:bash;foo:bar",
    "version": "15.05.22",
    "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
    "vmScaleSetName": "crpteste9vflji9",
    "vmSize": "Standard_A3",
    "zone": ""
}
```

#### <a name="linux"></a>[Linux](#tab/linux/)
```json
{
    "azEnvironment": "AZUREPUBLICCLOUD",
    "isHostCompatibilityLayerVm": "true",
    "licenseType":  "Windows_Client",
    "location": "westus",
    "name": "examplevmname",
    "offer": "UbuntuServer",
    "osProfile": {
        "adminUsername": "admin",
        "computerName": "examplevmname",
        "disablePasswordAuthentication": "true"
    },
    "osType": "Linux",
    "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
    "plan": {
        "name": "planName",
        "product": "planProduct",
        "publisher": "planPublisher"
    },
    "platformFaultDomain": "36",
    "platformUpdateDomain": "42",
    "publicKeys": [{
            "keyData": "ssh-rsa 0",
            "path": "/home/user/.ssh/authorized_keys0"
        },
        {
            "keyData": "ssh-rsa 1",
            "path": "/home/user/.ssh/authorized_keys1"
        }
    ],
    "publisher": "Canonical",
    "resourceGroupName": "macikgo-test-may-23",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
    "securityProfile": {
        "secureBootEnabled": "true",
        "virtualTpmEnabled": "false"
    },
    "sku": "18.04-LTS",
    "storageProfile": {
        "dataDisks": [{
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
                "uri": ""
            },
            "lun": "0",
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }],
        "imageReference": {
            "id": "",
            "offer": "UbuntuServer",
            "publisher": "Canonical",
            "sku": "16.04.0-LTS",
            "version": "latest"
        },
        "osDisk": {
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "diskSizeGB": "30",
            "diffDiskSettings": {
                "option": "Local"
            },
            "encryptionSettings": {
                "enabled": "false"
            },
            "image": {
                "uri": ""
            },
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampleosdiskname",
            "osType": "linux",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "baz:bash;foo:bar",
    "version": "15.05.22",
    "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
    "vmScaleSetName": "crpteste9vflji9",
    "vmSize": "Standard_A3",
    "zone": ""
}
```

---

#### <a name="sample-5-get-the-azure-environment-where-the-vm-is-running"></a>Ejemplo 5: Obtención del entorno de Azure donde se ejecuta la máquina virtual

Azure tiene varias nubes soberanas, como [Azure Government](https://azure.microsoft.com/overview/clouds/government/). En ocasiones, necesitará el entorno de Azure para tomar algunas decisiones acerca del tiempo de ejecución. En el siguiente ejemplo se muestra cómo lograr este comportamiento.

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

---

**Respuesta**

```
AzurePublicCloud
```

Aquí se enumeran las nubes y los valores del entorno de Azure.

| Nube | Entorno de Azure |
|-------|-------------------|
| [Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/) | AzurePublicCloud
| [Azure Government](https://azure.microsoft.com/overview/clouds/government/) | AzureUSGovernmentCloud
| [Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/) | AzureChinaCloud
| [Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/) | AzureGermanCloud


#### <a name="sample-6-retrieve-network-information"></a>Ejemplo 6: Recuperación de la información de red

**Solicitud**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01" | ConvertTo-Json  -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01"
```

---

**Respuesta**

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}
```

#### <a name="sample-7-retrieve-public-ip-address"></a>Ejemplo 7: Recuperación de la dirección IP pública

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

---

## <a name="attested-data"></a>Datos atestiguados

### <a name="get-attested-data"></a>Obtención de los datos atestiguados

IMDS le ofrece la garantía de que los datos proporcionados proceden de Azure. Microsoft firma parte de esta información, por lo que puede confirmar que una imagen de Azure Marketplace es la que se está ejecutando en Azure.

```
GET /metadata/attested/document
```

#### <a name="parameters"></a>Parámetros

| Nombre | Obligatorio/opcional | Descripción |
|------|-------------------|-------------|
| `api-version` | Obligatorio | Versión usada para atender la solicitud.
| `nonce` | Opcionales | Cadena de 10 dígitos que actúa como valor de seguridad criptográfico (nonce). Si no se proporciona ningún valor, IMDS utiliza la marca de tiempo UTC actual.

#### <a name="response"></a>Response

```json
{
    "encoding":"pkcs7",
    "signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> [!NOTE]
> Debido al mecanismo de almacenamiento en caché de IMDS, es posible que se devuelva un valor de nonce previamente almacenado en caché.

El blob de firma es una versión con la firma [pkcs7](https://aka.ms/pkcs7) del documento. Contiene el certificado usado para firmar junto con los detalles específicos de una determinada máquina virtual.

En el caso de las máquinas virtuales creadas con Azure Resource Manager, el documento incluye `vmId`, `sku`, `nonce`, `subscriptionId` y `timeStamp` para la creación y la expiración del documento, así como la información del plan sobre la imagen. La información del plan solo se rellena para las imágenes de Azure Marketplace.

En el caso de las máquinas virtuales creadas con el modelo de implementación clásica, solo se garantiza que se rellene `vmId`. Puede extraer el certificado de la respuesta y usarlo para confirmar que la respuesta es válida y procede de Azure.

El documento descodificado contiene estos campos:

| data | Descripción | Versión introducida |
|------|-------------|--------------------|
| `licenseType` | Tipo de licencia para la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit). Esta opción solo está presente en las máquinas virtuales habilitadas para Ventaja híbrida de Azure. | 01-09-2020
| `nonce` | Una cadena que se puede proporcionar de manera opcional con la solicitud. Si no se ha proporcionado ningún elemento `nonce`, se usa la marca de tiempo de la Hora universal coordinada actual. | 2018-10-01
| `plan` | El [plan de imagen de Azure Marketplace](/rest/api/compute/virtualmachines/createorupdate#plan). Contiene el identificador de plan (nombre), la oferta o imagen de producto (producto) y el identificador de publicador (publicador). | 2018-10-01
| `timestamp.createdOn` | La marca de tiempo UTC para el momento en que se creó el documento firmado | 2018-20-01
| `timestamp.expiresOn` | La marca de tiempo UTC para el momento en que expira el documento firmado | 2018-10-01
| `vmId` | [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual. | 2018-10-01
| `subscriptionId` | Suscripción de Azure para la máquina virtual | 2019-04-30
| `sku` | SKU específica de la imagen de VM | 2019-11-01

> [!NOTE]
> En el caso de las máquinas virtuales clásicas (no de Azure Resource Manager), solo se garantiza que se rellena vmId.

Documento de ejemplo:
```json
{
   "nonce":"20201130-211924",
   "plan":{
      "name":"planName",
      "product":"planProduct",
      "publisher":"planPublisher"
   },
   "sku":"Windows-Server-2012-R2-Datacenter",
   "subscriptionId":"8d10da13-8125-4ba9-a717-bf7490507b3d",
   "timeStamp":{
      "createdOn":"11/30/20 21:19:19 -0000",
      "expiresOn":"11/30/20 21:19:24 -0000"
   },
   "vmId":"02aab8a4-74ef-476e-8182-f6d2ba4166a6"
}
```

#### <a name="sample-1-validate-that-the-vm-is-running-in-azure"></a>Muestra 1: validación de que la VM se ejecuta en Azure

Los proveedores de Azure Marketplace quieren asegurarse de que su software tiene licencia para ejecutarse solo en Azure. Si alguien copia el disco duro virtual en un entorno local, el proveedor debe ser capaz de detectarlo. A través de IMDS, estos proveedores pueden obtener datos firmados que garantizan que la respuesta es solo de Azure.

> [!NOTE]
> Tenga en cuenta que este ejemplo requiere la instalación de la utilidad JQ.

**Validación**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
# Get the signature
$attestedDoc = Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri http://169.254.169.254/metadata/attested/document?api-version=2020-09-01
# Decode the signature
$signature = [System.Convert]::FromBase64String($attestedDoc.signature)
```

Compruebe que la firma proviene de Microsoft Azure y si hay errores en la cadena de certificados.

```powershell
# Get certificate chain
$cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]($signature)
$chain = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Chain
$chain.Build($cert)
# Print the Subject of each certificate in the chain
foreach($element in $chain.ChainElements)
{
    Write-Host $element.Certificate.Subject
}

# Get the content of the signed document
Add-Type -AssemblyName System.Security
$signedCms = New-Object -TypeName System.Security.Cryptography.Pkcs.SignedCms
$signedCms.Decode($signature);
$content = [System.Text.Encoding]::UTF8.GetString($signedCms.ContentInfo.Content)
Write-Host "Attested data: " $content
$json = $content | ConvertFrom-Json
# Do additional validation here
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
# Get the signature
curl --silent -H Metadata:True --noproxy "*" "http://169.254.169.254/metadata/attested/document?api-version=2020-09-01" | jq -r '.["signature"]' > signature
# Decode the signature
base64 -d signature > decodedsignature
# Get PKCS7 format
openssl pkcs7 -in decodedsignature -inform DER -out sign.pk7
# Get Public key out of pkc7
openssl pkcs7 -in decodedsignature -inform DER  -print_certs -out signer.pem
# Get the intermediate certificate
curl -s -o intermediate.cer "$(openssl x509 -in signer.pem -text -noout | grep " CA Issuers -" | awk -FURI: '{print $2}')"
openssl x509 -inform der -in intermediate.cer -out intermediate.pem
# Verify the contents
openssl smime -verify -in sign.pk7 -inform pem -noverify
```

**Resultados**

```json
Verification successful
{
  "nonce": "20181128-001617",
  "plan":
    {
      "name": "",
      "product": "",
      "publisher": ""
    },
  "timeStamp":
    {
      "createdOn": "11/28/18 00:16:17 -0000",
      "expiresOn": "11/28/18 06:16:17 -0000"
    },
  "vmId": "d3e0e374-fda6-4649-bbc9-7f20dc379f34",
  "licenseType": "Windows_Client",  
  "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
  "sku": "RS3-Pro"
}
```

Compruebe que la firma proviene de Microsoft Azure y si hay errores en la cadena de certificados.

```bash
# Verify the subject name for the main certificate
openssl x509 -noout -subject -in signer.pem
# Verify the issuer for the main certificate
openssl x509 -noout -issuer -in signer.pem
#Validate the subject name for intermediate certificate
openssl x509 -noout -subject -in intermediate.pem
# Verify the issuer for the intermediate certificate
openssl x509 -noout -issuer -in intermediate.pem
# Verify the certificate chain, for Azure China 21Vianet the intermediate certificate will be from DigiCert Global Root CA
openssl verify -verbose -CAfile /etc/ssl/certs/Baltimore_CyberTrust_Root.pem -untrusted intermediate.pem signer.pem
```

---

> [!NOTE]
> Debido al mecanismo de almacenamiento en caché de IMDS, es posible que se devuelva un valor `nonce` previamente almacenado en caché.

Se puede comparar el elemento `nonce` del documento firmado si proporcionó un parámetro `nonce` en la solicitud inicial.

> [!NOTE]
> El certificado para la nube pública y cada nube soberana será distinto.

| Nube | Certificado |
|-------|-------------|
| [Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/) | *.metadata.azure.com
| [Azure Government](https://azure.microsoft.com/overview/clouds/government/) | *.metadata.azure.us
| [Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/) | *.metadata.azure.cn
| [Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/) | *.metadata.microsoftazure.de

> [!NOTE]
> Es posible que los certificados no tengan una coincidencia exacta de `metadata.azure.com` para la nube pública. Por lo tanto, la validación de la certificación debe permitir un nombre común de cualquier subdominio `.metadata.azure.com`.

En aquellos casos en los que el certificado intermedio no se puede descargar debido a las restricciones de red durante la validación, es posible anclarlo. Azure sustituye los certificados, ya que es una práctica estándar de PKI. Debe actualizar los certificados anclados cuando se produzca la sustitución. Cada vez que planee un cambio para actualizar el certificado intermedio, se actualizará el blog de Azure y se notificará a los clientes de Azure. 

Puede encontrar los certificados intermedios en el [repositorio de PKI](https://www.microsoft.com/pki/mscorp/cps/default.htm). Los certificados intermedios para cada una de las regiones pueden ser diferentes.

> [!NOTE]
> El certificado intermedio para Azure China 21Vianet será de la entidad de certificación raíz global de DigiCert en lugar de Baltimore.
Además, si había anclado los certificados intermedios para Azure China como parte del cambio de la entidad de la cadena raíz, los certificados intermedios tendrán que actualizarse.


## <a name="managed-identity"></a>Identidad administrada

Se puede habilitar una identidad administrada que haya asignado el sistema en la VM. También puede asignar una o varias identidades administradas que haya asignado el usuario a la VM.
A continuación, puede solicitar tokens para identidades administradas desde IMDS. Estos tokens se pueden usar para autenticarse con otros servicios de Azure, como Azure Key Vault.

Para ver pasos detallados sobre cómo habilitar esta característica, consulte cómo [adquirir un token de acceso](../articles/active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).

## <a name="load-balancer-metadata"></a>Metadatos de Load Balancer
Al colocar instancias de máquina virtual o de conjunto de máquinas virtuales detrás de una instancia de Azure Standard Load Balancer, puede usar IMDS para recuperar los metadatos relacionados con el equilibrador de carga y las instancias. Para más información, consulte [Recuperación de información del equilibrador de carga](../articles/load-balancer/instance-metadata-service-load-balancer.md).

## <a name="scheduled-events"></a>Eventos programados
Puede obtener el estado de los eventos programados mediante IMDS. A continuación, el usuario puede especificar un conjunto de acciones para que se ejecuten en estos eventos. Para más información, consulte [Eventos programados para Linux](../articles/virtual-machines/linux/scheduled-events.md) o [Eventos programados para Windows](../articles/virtual-machines/windows/scheduled-events.md).


## <a name="sample-code-in-different-languages"></a>Código de ejemplo en diferentes lenguajes

En la tabla siguiente se muestran ejemplos de llamada a IMDS mediante el uso de diferentes idiomas en la VM:

| Idioma | Ejemplo |
|----------|---------|
| Bash | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.sh
| C# | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
| Go | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
| Java | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
| NodeJS | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
| Perl | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
| PowerShell | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
| Puppet | https://github.com/keirans/azuremetadata
| Python | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
| Ruby | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb

## <a name="errors-and-debugging"></a>Errores y depuración

Si no se encuentra un elemento de datos o hay una solicitud con formato incorrecto, el servicio de metadatos de instancia devuelve errores HTTP estándar. Por ejemplo:

| Código de estado HTTP | Motivo |
|------------------|--------|
| `200 OK` | La solicitud fue correcta.
| `400 Bad Request` | Falta el encabezado `Metadata: true` o el parámetro `format=json` al consultar un nodo hoja
| `404 Not Found` | El elemento solicitado no existe
| `405 Method Not Allowed` | El método HTTP (verbo) no se admite en el punto de conexión.
| `410 Gone` | Reintente después de un tiempo durante un máximo de 70 segundos
| `429 Too Many Requests` | Se ha superado el [límite de frecuencia](#rate-limiting) de la API
| `500 Service Error` | Vuelva a intentarlo más tarde

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

- Obtengo el error `400 Bad Request, Required metadata header not specified`. ¿Qué significa?
  - IMDS requiere que el encabezado `Metadata: true` se transmita en la solicitud. Transmitir este encabezado en la llamada de REST le permite obtener acceso a IMDS.

- ¿Por qué no recibo información de proceso de mi máquina virtual?
  - Actualmente IMDS solo admite instancias creadas con Azure Resource Manager.

- He creado mi VM mediante Azure Resource Manager hace algún tiempo. ¿Por qué no veo la información de metadatos de proceso?
  - Si creó la VM después de septiembre del 2016, agregue una [etiqueta](../articles/azure-resource-manager/management/tag-resources.md) para empezar a ver los metadatos del proceso. Si creó la VM antes de septiembre de 2016, agregue extensiones o discos de datos en la instancia de VM (o quítelos) para actualizar los metadatos.

- ¿Los datos de usuario son los mismos que los datos personalizados?
  - Los datos de usuario ofrecen una funcionalidad similar a la de los datos personalizados, lo que le permite pasar sus propios metadatos a la instancia de máquina virtual. La diferencia es que los datos de usuario se recuperan a través de IMDS y son persistentes en toda la duración de la instancia de máquina virtual. La característica de datos personalizados existente seguirá funcionando como se describe en [este artículo](https://docs.microsoft.com/azure/virtual-machines/custom-data). Sin embargo, solo puede obtener datos personalizados a través de la carpeta del sistema local, no a través de IMDS.

- ¿Por qué no veo todos los datos rellenados para una nueva versión?
  - Si creó la VM después de septiembre del 2016, agregue una [etiqueta](../articles/azure-resource-manager/management/tag-resources.md) para empezar a ver los metadatos del proceso. Si creó la VM antes de septiembre de 2016, agregue extensiones o discos de datos en la instancia de VM (o quítelos) para actualizar los metadatos.

- ¿Por qué recibo el error `500 Internal Server Error` o `410 Resource Gone`?
  - Vuelva a intentar la solicitud. Para obtener más información, consulte [Administración de errores transitorios](/azure/architecture/best-practices/transient-faults). Si el problema persiste, cree un problema de soporte técnico en Azure Portal para la VM.

- ¿Esto funciona en las instancias del conjunto de escalado de máquinas virtuales?
  - Sí, IMDS está disponible para las instancias del conjunto de escalado de máquinas virtuales.

- Actualicé mis etiquetas en el conjunto de escalado de máquinas virtuales, pero no aparecen en las instancias, a diferencia de las VM de instancia única. ¿Estoy haciendo algo mal?
  - Actualmente, las etiquetas para los conjuntos de escalado de máquinas virtuales solo se muestran a la VM durante el reinicio, el restablecimiento de una imagen o el cambio de un disco en la instancia.

- ¿Por qué no veo la información de la SKU de la máquina virtual en los detalles de `instance/compute`?
  - En el caso de las imágenes personalizadas creadas desde Azure Marketplace, la plataforma de Azure no conserva la información de la SKU para la imagen personalizada y los detalles de las máquinas virtuales creadas a partir de la imagen personalizada. Esto es así por diseño y, por lo tanto, no se muestra en los detalles de la máquina virtual `instance/compute`.

- ¿Por qué se agotó el tiempo de espera de la solicitud de mi llamada al servicio?
  - Las llamadas de metadatos se deben hacer desde la dirección IP principal asignada a la tarjeta de red principal de la máquina virtual. Además, si ha cambiado las rutas, debe haber una ruta para la dirección 169.254.169.254/32 en la tabla de rutas local de la VM.

    ### <a name="windows"></a>[Windows](#tab/windows/)

    1. Vuelque la tabla de rutas local y busque la entrada de IMDS. Por ejemplo:
        ```console
        > route print
        IPv4 Route Table
        ===========================================================================
        Active Routes:
        Network Destination        Netmask          Gateway       Interface  Metric
                0.0.0.0          0.0.0.0      172.16.69.1      172.16.69.7     10
                127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
                127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
        127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
            168.63.129.16  255.255.255.255      172.16.69.1      172.16.69.7     11
        169.254.169.254  255.255.255.255      172.16.69.1      172.16.69.7     11
        ... (continues) ...
        ```
    1. compruebe que existe una ruta para `169.254.169.254` y anote la interfaz de red correspondiente (por ejemplo, `172.16.69.7`).
    1. Vuelque la configuración de la interfaz, busque aquella interfaz que se corresponda con la que se menciona en la tabla de rutas y anote la dirección MAC (física).
        ```console
        > ipconfig /all
        ... (continues) ...
        Ethernet adapter Ethernet:

        Connection-specific DNS Suffix  . : xic3mnxjiefupcwr1mcs1rjiqa.cx.internal.cloudapp.net
        Description . . . . . . . . . . . : Microsoft Hyper-V Network Adapter
        Physical Address. . . . . . . . . : 00-0D-3A-E5-1C-C0
        DHCP Enabled. . . . . . . . . . . : Yes
        Autoconfiguration Enabled . . . . : Yes
        Link-local IPv6 Address . . . . . : fe80::3166:ce5a:2bd5:a6d1%3(Preferred)
        IPv4 Address. . . . . . . . . . . : 172.16.69.7(Preferred)
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        ... (continues) ...
        ```
    1. Confirme que la interfaz corresponde a la NIC principal y la dirección IP principal de la máquina virtual. Para encontrar la NIC y la dirección IP principales, consulte la configuración de red en Azure Portal o búsquela con la CLI de Azure. Anote las direcciones IP privadas (y la dirección MAC, si usa la CLI). Aquí tiene un ejemplo de la CLI de PowerShell:
        ```powershell
        $ResourceGroup = '<Resource_Group>'
        $VmName = '<VM_Name>'
        $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
        foreach($NicName in $NicNames)
        {
            $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
            Write-Host $NicName, $Nic.primary, $Nic.macAddress
        }
        # Output: wintest767 True 00-0D-3A-E5-1C-C0
        ```
    1. Si no coinciden, actualice la tabla de rutas para que la NIC y la dirección IP principales estén dirigidas.

    ### <a name="linux"></a>[Linux](#tab/linux/)

    1. Vuelque la tabla de rutas local con un comando como `netstat -r` y busque la entrada IMDS (por ejemplo):
        ```console
        ~$ netstat -r
        Kernel IP routing table
        Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
        default         _gateway        0.0.0.0         UG        0 0          0 eth0
        168.63.129.16   _gateway        255.255.255.255 UGH       0 0          0 eth0
        169.254.169.254 _gateway        255.255.255.255 UGH       0 0          0 eth0
        172.16.69.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
        ```
    1. Compruebe que existe una ruta para `169.254.169.254` y anote la interfaz de red correspondiente (por ejemplo, `eth0`).
    1. Vuelque la configuración de la interfaz para la interfaz correspondiente en la tabla de rutas (tenga en cuenta que el nombre exacto del archivo de configuración puede variar).
        ```console
        ~$ cat /etc/netplan/50-cloud-init.yaml
        network:
        ethernets:
            eth0:
                dhcp4: true
                dhcp4-overrides:
                    route-metric: 100
                dhcp6: false
                match:
                    macaddress: 00:0d:3a:e4:c7:2e
                set-name: eth0
        version: 2
        ```
    1. Si usa una dirección IP dinámica, anote la dirección MAC. Si usa una dirección IP estática, puede anotar las direcciones IP o la dirección MAC que aparecen.
    1. Confirme que la interfaz corresponde a la NIC principal y la dirección IP principal de la máquina virtual. Para encontrar la NIC y la dirección IP principales, consulte la configuración de red en Azure Portal o búsquela con la CLI de Azure. Anote las direcciones IP privadas (y la dirección MAC, si usa la CLI). Aquí tiene un ejemplo de la CLI de PowerShell:
        ```powershell
        $ResourceGroup = '<Resource_Group>'
        $VmName = '<VM_Name>'
        $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
        foreach($NicName in $NicNames)
        {
            $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
            Write-Host $NicName, $Nic.primary, $Nic.macAddress
        }
        # Output: ipexample606 True 00-0D-3A-E4-C7-2E
        ```
    1. Si no coinciden, actualice la tabla de rutas para que la NIC/dirección IP principal estén dirigidas.

    ---

- Clústeres de conmutación por error de Windows Server
  - Cuando se consulta IMDS con los clústeres de conmutación por error, a veces es necesario agregar una ruta a la tabla de rutas. A continuación, se indica cómo puede hacerlo.

    1. Abra un símbolo del sistema con privilegios de administrador.

    1. Ejecute el siguiente comando y anote la dirección de la interfaz de red de destino (`0.0.0.0`) en la tabla de rutas IPv4.

    ```bat
    route print
    ```

    > [!NOTE]
    > La salida de ejemplo siguiente es de una VM de Windows Server que tiene un clúster de conmutación por error habilitado. Para simplificar las cosas, la salida solo contiene la tabla de rutas IPv4.

    ```
    IPv4 Route Table
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
            0.0.0.0          0.0.0.0         10.0.1.1        10.0.1.10    266
            10.0.1.0  255.255.255.192         On-link         10.0.1.10    266
            10.0.1.10  255.255.255.255         On-link         10.0.1.10    266
            10.0.1.15  255.255.255.255         On-link         10.0.1.10    266
            10.0.1.63  255.255.255.255         On-link         10.0.1.10    266
            127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
            127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
    127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
        169.254.0.0      255.255.0.0         On-link     169.254.1.156    271
        169.254.1.156  255.255.255.255         On-link     169.254.1.156    271
    169.254.255.255  255.255.255.255         On-link     169.254.1.156    271
            224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
            224.0.0.0        240.0.0.0         On-link     169.254.1.156    271
    255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    255.255.255.255  255.255.255.255         On-link     169.254.1.156    271
    255.255.255.255  255.255.255.255         On-link         10.0.1.10    266
    ```

    Ejecute el siguiente comando y use la dirección de la interfaz de red de destino (`0.0.0.0`) que es (`10.0.1.10`) en el ejemplo.

    ```bat
    route add 169.254.169.254/32 10.0.1.10 metric 1 -p
    ```

## <a name="support"></a>Soporte técnico

Si no puede obtener una respuesta de metadatos después de varios intentos, puede crear un problema de soporte técnico en Azure Portal.

## <a name="product-feedback"></a>Comentarios sobre el producto

Puede proporcionar comentarios sobre el producto e ideas en nuestro canal de la fuente de usuario en Virtual Machines > Instance Metadata Service [aquí](https://feedback.azure.com/forums/216843-virtual-machines?category_id=394627).

## <a name="next-steps"></a>Pasos siguientes

- [Obtener un token de acceso para la VM](../articles/active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)

- [Eventos programados para Linux](../articles/virtual-machines/linux/scheduled-events.md)

- [Eventos programados para Windows](../articles/virtual-machines/windows/scheduled-events.md)
