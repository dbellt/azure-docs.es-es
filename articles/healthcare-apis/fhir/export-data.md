---
title: Ejecución de la exportación mediante la invocación del comando $export en Azure API for FHIR
description: En este artículo se describe cómo exportar datos de FHIR mediante $export
author: caitlinv39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 5/17/2021
ms.author: cavoeg
ms.openlocfilehash: 50f79d8b73b6c716e14504d6d763d900a7bed488
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110078669"
---
# <a name="how-to-export-fhir-data"></a>Exportación de datos de FHIR


La característica de exportación masiva permite exportar datos desde el servidor de FHIR según la [especificación FHIR](https://hl7.org/fhir/uv/bulkdata/export/index.html). 

Antes de $export, querrá asegurarse de que el Azure API for FHIR está configurado para usarlo. Para configurar las opciones de exportación y crear una cuenta de almacenamiento de Azure, consulte la [página de configuración de los datos de exportación](configure-export-data.md).

## <a name="using-export-command"></a>Uso del comando $export

Después de configurar Azure API for FHIR para la exportación, puede usar el comando $export para exportar los datos fuera del servicio. Los datos se almacenarán en la cuenta de almacenamiento que especificó al configurar la exportación. Para más información sobre cómo invocar el comando $export en el servidor de FHIR, lea la documentación sobre la [especificación de $export del estándar FHIR de HL7](https://hl7.org/Fhir/uv/bulkdata/export/index.html).


**Trabajos bloqueados en mal estado**

En algunas situaciones, existe la posibilidad de que un trabajo se atasca en un estado no bueno. Esto puede ocurrir especialmente si los permisos de la cuenta de almacenamiento no se han configurado correctamente. Una manera de validar si la exportación se realiza correctamente es comprobar la cuenta de almacenamiento para ver si están presentes los archivos de contenedor correspondientes (es decir, ndjson). Si no están presentes y no hay ningún otro trabajo de exportación en ejecución, existe la posibilidad de que el trabajo actual esté bloqueado en un estado no bueno. Debe cancelar el trabajo de exportación mediante el envío de una solicitud de cancelación e intentar volver a hacer cola del trabajo. El tiempo de ejecución predeterminado para una exportación en mal estado es de 10 minutos antes de que se detenga y se mueva a un nuevo trabajo o vuelva a intentar la exportación. 

Azure API for FHIR admite $export en los siguientes niveles:
* [Sistema](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---system-level-export): `GET https://<<FHIR service base URL>>/$export>>`
* [Paciente](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---all-patients): `GET https://<<FHIR service base URL>>/Patient/$export>>`
* [Grupo de pacientes*:](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---group-of-patients) Azure API for FHIR exporta todos los recursos relacionados, pero no exporta las características del grupo: `GET https://<<FHIR service base URL>>/Group/[ID]/$export>>`

Cuando se exportan los datos, se crea un archivo independiente para cada tipo de recurso. Para asegurarse de que los archivos exportados no se vuelven demasiado grandes. Creamos un nuevo archivo después de que el tamaño de un único archivo exportado sea superior a 64 MB. El resultado es que puede obtener varios archivos para cada tipo de recurso, que se enumerarán (es decir, Patient-1.ndjson, Patient-2.ndjson). 


> [!Note] 
> `Patient/$export` y `Group/[ID]/$export` pueden exportar recursos duplicados si el recurso está en un compartimiento de más de un recurso o se encuentra en varios grupos.

Además, se admite la comprobación del estado de exportación a través de la dirección URL devuelta por el encabezado de ubicación durante la puesta en cola, junto con la cancelación del trabajo de exportación real.

### <a name="exporting-fhir-data-to-adls-gen2"></a>Exportación de datos de FHIR a ADLS Gen2

Actualmente se admiten $export para las cuentas de almacenamiento habilitadas para ADLS Gen2, con la siguiente limitación:

- El usuario no puede aprovechar los [espacios](../../storage/blobs/data-lake-storage-namespace.md)de nombres jerárquicos, pero no hay una manera de dirigir la exportación a un subdirectorio específico dentro del contenedor. Solo se ofrece la posibilidad de destinar a un contenedor específico (donde crearemos una carpeta para cada exportación).
- Una vez completada la exportación, nunca se exporta nada de nuevo a esa carpeta, ya que las exportaciones posteriores al mismo contenedor estarán dentro de una carpeta recién creada.


## <a name="settings-and-parameters"></a>Configuración y parámetros

### <a name="headers"></a>Encabezados
Hay dos parámetros de encabezado necesarios que se deben establecer para los trabajos de $export. Los valores se definen mediante la [especificación de $export](https://hl7.org/Fhir/uv/bulkdata/export/index.html#headers) actual.
* **Accept** (Aceptar): application/fhir+json.
* **Prefer** (Preferir): respond-async.

### <a name="query-parameters"></a>Parámetros de consulta
Azure API for FHIR admite los siguientes parámetros de consulta. Todos estos parámetros son opcionales:

|Parámetro de consulta        | ¿Se define mediante la especificación de FHIR?    |  Descripción|
|------------------------|---|------------|
| \_outputFormat | Sí | Actualmente admite tres valores que se alinean con la especificación de FHIR: application/fhir+ndjson, application/ndjson o just ndjson. Todos los trabajos de exportación devolverán `ndjson` y el valor pasado no tiene ningún efecto sobre el comportamiento del código. |
| \_since | Sí | Permite exportar solo los recursos que se han modificado desde el momento indicado. |
| \_type | Sí | Permite especificar los tipos de recursos que se van a incluir. Por ejemplo, \_type=Patient devolvería solo los recursos de pacientes.|
| \_typefilter | Sí | Para solicitar un filtrado más preciso, puede usar \_typefilter junto con el parámetro \_type. El valor del parámetro _typeFilter es una lista separada por comas de consultas de FHIR que restringen aún más los resultados. |
| \_container | No |  Especifica el contenedor en la cuenta de almacenamiento configurada donde se deben exportar los datos. Si se especifica un contenedor, los datos se exportarán a este en una nueva carpeta con el nombre. Si no se especifica, se exportarán a un nuevo contenedor con la marca de tiempo y el identificador de trabajo. |

> [!Note]
> Solo las cuentas de almacenamiento de la misma suscripción que para Azure API for FHIR pueden registrarse como destino para $export operaciones.

## <a name="secure-export-to-azure-storage"></a>Exportación segura a Azure Storage

Azure API for FHIR admite una operación de exportación segura. Elija una de las dos opciones siguientes:

* Permitir Azure API for FHIR como un servicio de confianza de Microsoft para acceder a la cuenta de almacenamiento de Azure.
 
* Permitir que direcciones IP específicas asociadas a Azure API for FHIR accedan a la cuenta de Almacenamiento de Azure. Esta opción proporciona dos configuraciones diferentes en función de si la cuenta de almacenamiento está en la misma ubicación que o está en una ubicación diferente de la del Azure API for FHIR.

### <a name="allowing-azure-api-for-fhir-as-a-microsoft-trusted-service"></a>Permitir Azure API for FHIR como un servicio de confianza de Microsoft

Seleccione una cuenta de almacenamiento en la Azure Portal y, a continuación, seleccione la **hoja** Redes. Seleccione **Redes seleccionadas en** la pestaña **Firewalls y redes** virtuales.

> [!IMPORTANT]
> Asegúrese de que ha concedido permiso de acceso a la cuenta de almacenamiento para Azure API for FHIR mediante su identidad administrada. Para más información, consulte [Configuración de la configuración de exportación y configuración de la cuenta de almacenamiento.](https://docs.microsoft.com/azure/healthcare-apis/fhir/configure-export-data)

  :::image type="content" source="media/export-data/storage-networking.png" alt-text="Configuración de redes de Azure Storage." lightbox="media/export-data/storage-networking.png":::

En la **sección Excepciones,** seleccione la casilla Permitir que servicios Microsoft de confianza **acceda a esta cuenta de** almacenamiento y guarde la configuración. 

:::image type="content" source="media/export-data/exceptions.png" alt-text="Permitir que los servicios Microsoft de confianza accedan a esta cuenta de almacenamiento.":::

Ya está listo para exportar datos de FHIR a la cuenta de almacenamiento de forma segura. Tenga en cuenta que la cuenta de almacenamiento está en redes seleccionadas y no es accesible públicamente. Para acceder a los archivos, puede habilitar y usar puntos de conexión privados para la cuenta de almacenamiento, o bien habilitar todas las redes para la cuenta de almacenamiento durante un breve período de tiempo.

> [!IMPORTANT]
> La interfaz de usuario se actualizará más adelante para que pueda seleccionar el tipo de recurso para Azure API for FHIR una instancia de servicio específica.

### <a name="allowing-specific-ip-addresses-for-the-azure-storage-account-in-a-different-region"></a>Permitir direcciones IP específicas para la cuenta de Almacenamiento de Azure en una región diferente

Seleccione **Redes** de la cuenta de Almacenamiento de Azure en el portal. 
   
Seleccione **Redes seleccionadas**. En la sección Firewall, especifique la dirección IP en el **cuadro Intervalo de** direcciones. Agregue intervalos IP para permitir el acceso desde Internet o las redes locales. Puede encontrar la dirección IP en la tabla siguiente para la región de Azure en la que se Azure API for FHIR servicio.

|**Región de Azure**         |**Dirección IP pública** |
|:----------------------|:-------------------|
| Este de Australia       | 20.53.44.80       |
| Centro de Canadá       | 20.48.192.84      |
| Centro de EE. UU.           | 52.182.208.31     |
| Este de EE. UU.              | 20.62.128.148     |
| Este de EE. UU. 2            | 20.49.102.228     |
| EUAP de Este de EE. UU. 2       | 20.39.26.254      |
| Norte de Alemania        | 51.116.51.33      |
| Centro-oeste de Alemania | 51.116.146.216    |
| Japón Oriental           | 20.191.160.26     |
| Centro de Corea del Sur        | 20.41.69.51       |
| Centro-Norte de EE. UU     | 20.49.114.188     |
| Norte de Europa         | 52.146.131.52     |
| Norte de Sudáfrica   | 102.133.220.197   |
| Centro-sur de EE. UU.     | 13.73.254.220     |
| Sudeste de Asia       | 23.98.108.42      |
| Norte de Suiza    | 51.107.60.95      |
| Sur de Reino Unido             | 51.104.30.170     |
| Oeste de Reino Unido              | 51.137.164.94     |
| Centro-Oeste de EE. UU.      | 52.150.156.44     |
| Oeste de Europa          | 20.61.98.66       |
| Oeste de EE. UU. 2            | 40.64.135.77      |

> [!NOTE]
> Los pasos anteriores son similares a los pasos de configuración descritos en el documento Conversión de datos a FHIR (versión preliminar). Para más información, consulte [Host and use templates (Hospedar y usar plantillas).](https://docs.microsoft.com/azure/healthcare-apis/fhir/convert-data#host-and-use-templates)

### <a name="allowing-specific-ip-addresses-for-the-azure-storage-account-in-the-same-region"></a>Permitir direcciones IP específicas para la cuenta de Almacenamiento de Azure en la misma región

El proceso de configuración es el mismo que el anterior, salvo que en su lugar se usa un intervalo de direcciones IP específico en formato CIDR, 100.64.0.0/10. La razón por la que se debe especificar el intervalo de direcciones IP, que incluye 100.64.0.0 a 100.127.255.255, es que la dirección IP real utilizada por el servicio varía, aunque lo hará dentro del intervalo, para cada solicitud de $export.

> [!Note] 
> En su lugar, es posible que se use una dirección IP privada dentro del intervalo 10.0.2.0/24. En ese caso, la $export operación no se realizará correctamente. Puede volver $export la solicitud, pero no hay ninguna garantía de que la próxima vez se usará una dirección IP dentro del intervalo de 100.64.0.0/10. Ese es el comportamiento de red conocido por diseño. La alternativa es configurar la cuenta de almacenamiento en una región diferente.
    
## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a exportar recursos de FHIR mediante el comando $export. A continuación, para obtener información sobre cómo exportar datos no identificados, consulte:
 
>[!div class="nextstepaction"]
>[Exportación de datos sin identificación](de-identified-export.md)
