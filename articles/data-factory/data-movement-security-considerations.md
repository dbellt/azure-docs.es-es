---
title: Consideraciones sobre la seguridad
description: En este artículo se describe la infraestructura de seguridad básica que utilizan los servicios de movimiento de datos de Azure Data Factory para ayudar a proteger los datos.
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/03/2021
ms.openlocfilehash: 54c96aced03853b8a3f78ff0f348eeb6459afccc
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108760331"
---
# <a name="security-considerations-for-data-movement-in-azure-data-factory"></a>Consideraciones de seguridad para el movimiento de datos en Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
>
> * [Versión 1](v1/data-factory-data-movement-security-considerations.md)
> * [Versión actual](data-movement-security-considerations.md)

 [!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe la infraestructura de seguridad básica que utilizan los servicios de movimiento de datos en Azure Data Factory para ayudar a proteger los datos. Los recursos de administración de Data Factory están integrados en la infraestructura de seguridad de Azure y aplican todas las medidas de seguridad que ofrece Azure.

En una solución de Data Factory, se crean una o varias [canalizaciones](concepts-pipelines-activities.md)de datos. Una canalización es una agrupación lógica de actividades que realizan una tarea. Estas canalizaciones residen en la región donde se creó la factoría de datos.

Aunque Data Factory solo está disponible en algunas regiones, el servicio de movimiento de datos está [disponible globalmente](concepts-integration-runtime.md#integration-runtime-location) para garantizar el cumplimiento de datos, la eficiencia y menores costos de salida de la red.

Azure Data Factory, que incluye Azure Integration Runtime y el entorno de ejecución de integración autohospedado, no almacena ningún dato temporal, datos de caché ni registro, excepto las credenciales de servicio vinculadas de los almacenes de datos en la nube, que se cifran mediante certificados. Con Data Factory, puede crear flujos de trabajo controlados por datos para orquestar el movimiento de los datos entre los [almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats) y organizar el procesamiento de los datos mediante [servicios de proceso](compute-linked-services.md) en otras regiones o en entornos locales. También puede supervisar y administrar flujos de trabajo mediante SDK y Azure Monitor.

Data Factory tiene certificación para:

| **[Certificación CSA STAR](https://www.microsoft.com/trustcenter/compliance/csa-star-certification)** |
| :----------------------------------------------------------- |
| **[ISO 20000-1:2011](https://www.microsoft.com/trustcenter/Compliance/ISO-20000-1)** |
| **[ISO 22301:2012](/compliance/regulatory/offering-iso-22301)** |
| **[ISO 27001:2013](https://www.microsoft.com/trustcenter/compliance/iso-iec-27001)** |
| **[ISO 27017:2015](https://www.microsoft.com/trustcenter/compliance/iso-iec-27017)** |
| **[ISO 27018:2014](https://www.microsoft.com/trustcenter/compliance/iso-iec-27018)** |
| **[ISO 9001:2015](https://www.microsoft.com/trustcenter/compliance/iso-9001)** |
| **[SOC 1, 2, 3](https://www.microsoft.com/trustcenter/compliance/soc)** |
| **[HIPAA BAA](/compliance/regulatory/offering-hipaa-hitech)** |
| **[HITRUST](/compliance/regulatory/offering-hitrust)** |

Si está interesado en el cumplimiento de Azure y en cómo Azure protege su propia infraestructura, visite el [Centro de confianza de Microsoft](https://microsoft.com/trustcenter/default.aspx). Para consultar la lista más reciente con todas las ofertas de cumplimiento normativo de Azure, visite https://aka.ms/AzureCompliance.

En este artículo, revisamos las consideraciones de seguridad en los dos escenarios de movimiento de datos siguientes:

- **Escenario de nube**: en este escenario, el origen y el destino son públicamente accesibles a través de Internet. Por ejemplo, los servicios de almacenamiento administrado en la nube, como Azure Storage, Azure Synapse Analytics, Azure SQL Database, Azure Data Lake Store, Amazon S3, Amazon Redshift, los servicios SaaS como Salesforce y los protocolos web, como FTP y OData. Encuentre una lista completa de orígenes de datos admitidos en [formatos y almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
- **Escenario híbrido**: en este escenario, el origen o el destino están detrás de un firewall o dentro de una red corporativa local. O bien el almacén de datos está en una red privada o en una red virtual (que suele ser el origen) y no se puede acceder a él públicamente. Los servidores de base de datos hospedados en máquinas virtuales también se incluyen en este escenario.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="cloud-scenarios"></a>Escenarios de nube

### <a name="securing-data-store-credentials"></a>Protección de las credenciales del almacén de datos

- **Almacenamiento de credenciales cifradas en un almacén administrado de Azure Data Factory**. Data Factory ayuda a proteger las credenciales del almacén de datos cifrándolas con certificados administrados por Microsoft. Estos certificados se alternan cada dos años (esto incluye la renovación del certificado y la migración de las credenciales). Para más información sobre la seguridad de Azure Storage, consulte [Introducción a la seguridad de Azure Storage](../storage/blobs/security-recommendations.md).
- **Almacenamiento de credenciales en Azure Key Vault** También puede almacenar la credencial del almacén de datos en [Azure Key Vault](https://azure.microsoft.com/services/key-vault/). Data Factory recupera la credencial durante la ejecución de una actividad. Para obtener más información, consulte el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md).

### <a name="data-encryption-in-transit"></a>Cifrado de datos en tránsito
Si el almacén de datos en la nube es compatible con HTTPS o TLS, todas las transferencias de datos entre los servicios de movimiento de datos de Data Factory y un almacén de datos en la nube se realizan a través del canal seguro HTTPS o TLS.

> [!NOTE]
> Todas las conexiones a Azure SQL Database y Azure Synapse Analytics requieren cifrado (SSL/TLS) que haya datos en tránsito hacia y desde la base de datos. Al crear una canalización con JSON, agregue la propiedad encryption y establézcala en **true** en la cadena de conexión. Para Azure Storage, puede usar **HTTPS** en la cadena de conexión.

> [!NOTE]
> Para habilitar el cifrado en tránsito al mover los datos desde Oracle, aplique alguna de las opciones siguientes:
>
> 1. En el servidor de Oracle, vaya a Oracle Advanced Security (OAS) y realice la configuración de cifrado, que admite el cifrado Triple-DES (3DES) y el Estándar de cifrado avanzado (AES); consulte [aquí](https://docs.oracle.com/cd/E11882_01/network.112/e40393/asointro.htm#i1008759) los detalles. ADF negocia automáticamente el método de cifrado para usar el que configura en OAS al establecer la conexión con Oracle.
> 2. En ADF, puede agregar EncryptionMethod=1 en la cadena de conexión (en el servicio vinculado). En este caso, se usará SSL/TLS como método de cifrado. Para usarlo, debe deshabilitar la configuración de cifrado no SSL en OAS en el lado servidor de Oracle para evitar conflictos de cifrado.

> [!NOTE]
> La versión de TLS usada es la 1.2.

### <a name="data-encryption-at-rest"></a>Cifrado de datos en reposo

Algunos almacenes de datos admiten el cifrado de datos en reposo. Se recomienda habilitar el mecanismo de cifrado de datos para estos almacenes. 

#### <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

El Cifrado de datos transparente (TDE) de Azure Synapse Analytics ayuda a protegerse frente a la amenaza de actividad malintencionada al realizar el cifrado y descifrado en tiempo real de los datos en reposo. Este comportamiento es transparente para el cliente. Para obtener más información, consulte [Protección de una base de datos en Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md).

#### <a name="azure-sql-database"></a>Azure SQL Database

Azure SQL Database admite también el Cifrado de datos transparente (TDE), que ayuda a proteger frente a la amenaza de actividad malintencionada al realizar el cifrado y descifrado en tiempo real de los datos sin que haya que efectuar cambios en la aplicación. Este comportamiento es transparente para el cliente. Para más información, consulte [Cifrado de datos transparente para SQL Database y Data Warehouse](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql).

#### <a name="azure-data-lake-store"></a>Azure Data Lake Store

Azure Data Lake Store también ofrece el cifrado de los datos que se almacenan en la cuenta. Cuando se habilita, Data Lake Store cifrará automáticamente los datos antes de la persistencia y los descifrará antes de la recuperación, por lo que resulta un proceso completamente transparente para el cliente que accede a los datos. Para más información, consulte [Seguridad en Azure Data Lake Store](../data-lake-store/data-lake-store-security-overview.md). 

#### <a name="azure-blob-storage-and-azure-table-storage"></a>Azure Blob Storage y Azure Table Storage

Azure Blob Storage y Azure Table Storage admiten el cifrado del servicio Storage (SSE), que cifra automáticamente los datos antes de su persistencia en el almacenamiento y los descifra antes de su recuperación. Para más información, consulte [Cifrado del servicio Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md).

#### <a name="amazon-s3"></a>Amazon S3

Amazon S3 admite el cifrado de cliente y servidor para datos en reposo. Para más información, consulte [Protección de datos del cliente mediante cifrado](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html).

#### <a name="amazon-redshift"></a>Amazon Redshift

Amazon Redshift admite cifrado de clúster para datos en reposo. Para más información, consulte [Amazon Redshift Database Encryption](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html) (Cifrado de base de datos de Amazon Redshift). 

#### <a name="salesforce"></a>Salesforce

Salesforce admite Shield Platform Encryption, que permite el cifrado de todos los archivos, datos adjuntos y campos personalizados. Para más información, consulte [Understanding the Web Server OAuth Authentication Flow](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_web_server_oauth_flow.htm) (Descripción del flujo de autenticación OAuth de servidor web).  

## <a name="hybrid-scenarios"></a>Escenarios híbridos

Los escenarios híbridos necesitan que el entorno de ejecución de integración autohospedado se instale en una red local o en una virtual (Azure), o bien dentro de una nube privada virtual (Amazon). El entorno de ejecución de integración autohospedado debe poder tener acceso a los almacenes de datos locales. Para más información acerca del entorno de ejecución de integración autohospedado, consulte [Creación y configuración de una instancia de Integration Runtime autohospedado](./create-self-hosted-integration-runtime.md). 

![canales del entorno de ejecución de integración autohospedado](media/data-movement-security-considerations/data-management-gateway-channels.png)

El canal del comandos permite la comunicación entre los servicios de movimiento de datos de Data Factory y el entorno de ejecución de integración autohospedado. La comunicación contiene información relacionada con la actividad. El canal de datos se usa para transferir datos entre los almacenes de datos locales y los almacenes de datos en la nube.    

### <a name="on-premises-data-store-credentials"></a>Credenciales de los almacenes de datos locales

Las credenciales se pueden almacenar en Data Factory o [Data Factory puede hacer referencia a ellas](store-credentials-in-key-vault.md) durante el tiempo de ejecución desde Azure Key Vault. Si almacena las credenciales en Data Factory, siempre se almacenan cifrada en entorno de ejecución de integración autohospedado. 

   - **Almacenar credenciales localmente**. Si usa directamente el cmdlet **Set-AzDataFactoryV2LinkedService** con las cadenas de conexión y las credenciales insertadas en JSON, el servicio vinculado se cifra y almacena en el entorno de ejecución de integración autohospedado.  En este caso, las credenciales atraviesan el servicio de back-end de Azure, que es muy seguro, hasta la máquina de integración autohospedada, donde finalmente se cifran y se almacenan. El entorno de ejecución de integración autohospedado utiliza Windows [DPAPI](/previous-versions/ms995355(v=msdn.10)) para cifrar los datos confidenciales y la información de credenciales.

   - **Almacenamiento de credenciales en Azure Key Vault** También puede almacenar la credencial del almacén de datos en [Azure Key Vault](https://azure.microsoft.com/services/key-vault/). Data Factory recupera la credencial durante la ejecución de una actividad. Para obtener más información, consulte el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md).

   - **Almacene las credenciales localmente sin transferirlas mediante el back-end de Azure al entorno de ejecución de integración autohospedado**. Si desea cifrar las credenciales y almacenarlas localmente en el entorno de ejecución de integración autohospedado sin hacer que fluyan en el back-end de Data Factory, siga los pasos descritos en [Cifrado de credenciales de almacenes de datos locales en Azure Data Factory](encrypt-credentials-self-hosted-integration-runtime.md). Todos los conectores admiten esta opción. El entorno de ejecución de integración autohospedado utiliza Windows [DPAPI](/previous-versions/ms995355(v=msdn.10)) para cifrar los datos confidenciales y la información de credenciales. 

   - Use el cmdlet **New-AzDataFactoryV2LinkedServiceEncryptedCredential** para cifrar las credenciales del servicio vinculado e información confidencial en el servicio vinculado. Después, puede usar el código JSON devuelto (con el elemento **EncryptedCredential** en la cadena de conexión) para crear un servicio vinculado mediante el cmdlet **Set-AzDataFactoryV2LinkedService**.  

#### <a name="ports-used-when-encrypting-linked-service-on-self-hosted-integration-runtime"></a>Puertos utilizados durante el cifrado del servicio vinculado en Integration Runtime autohospedado

De manera predeterminada, cuando el acceso remoto desde la intranet está habilitado, PowerShell usa el puerto 8060 en la máquina con el entorno de ejecución de integración autohospedado para una comunicación segura. Si es necesario, este puerto se puede cambiar desde el administrador de configuración de Integration Runtime en la pestaña Configuración:

:::image type="content" source="media/data-movement-security-considerations/integration-runtime-configuration-manager-settings.png" alt-text="Pestaña de configuración del administrador de configuración de Integration Runtime":::

:::image type="content" source="media/data-movement-security-considerations/https-port-for-gateway.png" alt-text="Puerto HTTPS para la puerta de enlace":::

### <a name="encryption-in-transit"></a>Cifrado en tránsito

Todas las transferencias de datos se realizan a través del canal seguro HTTPS y TLS sobre TCP para evitar ataques tipo "Man in the middle" durante la comunicación con los servicios de Azure.

También puede usar una [conexión VPN de IPSec](../vpn-gateway/vpn-gateway-about-vpn-devices.md) o [Azure ExpressRoute](../expressroute/expressroute-introduction.md) para proteger aún más el canal de comunicación entre la red local y Azure.

Azure Virtual Network es una representación lógica de la red en la nube. Puede conectar una red local a la red virtual mediante la configuración de una conexión VPN de IPSec (de sitio a sitio) o ExpressRoute (Emparejamiento privado)

En la tabla siguiente, se resumen las recomendaciones de configuración de red y del entorno de ejecución de integración autohospedado en función de diferentes combinaciones de ubicaciones de origen y de destino para el movimiento de datos híbridos.

| Source      | Destination                              | Network configuration (Configuración de red)                    | Configuración de Integration Runtime                |
| ----------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| Local | Máquinas virtuales y servicios en la nube implementados en redes virtuales | VPN de IPSec (de punto a sitio o de sitio a sitio) | El entorno de ejecución de integración autohospedado se debe instalar en una máquina virtual de Azure en una red virtual.  |
| Local | Máquinas virtuales y servicios en la nube implementados en redes virtuales | ExpressRoute (Emparejamiento privado)           | El entorno de ejecución de integración autohospedado se debe instalar en una máquina virtual de Azure en una red virtual.  |
| Local | Servicios basados en Azure que tienen un punto de conexión público | ExpressRoute (emparejamiento de Microsoft)            | El entorno de ejecución de integración autohospedado se puede instalar de forma local o en una máquina virtual de Azure. |

Las siguientes imágenes muestran el uso del entorno de ejecución de integración autohospedado para mover datos entre una base de datos local y servicios de Azure mediante ExpressRoute e IPSec VPN (con Azure Virtual Network):

#### <a name="express-route"></a>ExpressRoute

![Usar ExpressRoute con puerta de enlace](media/data-movement-security-considerations/express-route-for-gateway.png) 

#### <a name="ipsec-vpn"></a>VPN de IPSec

![Conexión VPN de IPSec con la puerta de enlace](media/data-movement-security-considerations/ipsec-vpn-for-gateway.png)

### <a name="firewall-configurations-and-allow-list-setting-up-for-ip-addresses"></a> Configuraciones de firewall y configuración de direcciones IP permitidas

> [!NOTE]
> Puede que tenga que administrar puertos o configurar la lista de admitidos en los dominios a nivel del firewall corporativo, en función de lo que requiera cada origen de datos. En esta tabla, Azure SQL Database, Azure Synapse Analytics y Azure Data Lake Store solo se usan a modo de ejemplo.

> [!NOTE]
> Para obtener detalles sobre las estrategias de acceso a datos a través de Azure Data Factory, consulte [este artículo](./data-access-strategies.md#data-access-strategies-through-azure-data-factory).

#### <a name="firewall-requirements-for-on-premisesprivate-network"></a>Requisitos de firewall para redes locales o privadas

En una empresa, se ejecuta un firewall corporativo en el enrutador central de la organización. El Firewall de Windows se ejecuta como demonio en la máquina local con Integration Runtime autohospedado instalado. 

En la tabla siguiente se proporcionan el puerto de salida y los requisitos de dominio para el firewall corporativo:

[!INCLUDE [domain-and-outbound-port-requirements](includes/domain-and-outbound-port-requirements.md)]

> [!NOTE]
> Puede que tenga que administrar puertos o configurar la lista de admitidos en los dominios a nivel del firewall corporativo, en función de lo que requiera cada origen de datos. En esta tabla, Azure SQL Database, Azure Synapse Analytics y Azure Data Lake Store solo se usan a modo de ejemplo.   

En la tabla siguiente, se proporcionan los requisitos del puerto de entrada para el Firewall de Windows.

| Puertos de entrada | Descripción                              |
| ------------- | ---------------------------------------- |
| 8060 (TCP)    | Requerido por el cmdlet de cifrado de PowerShell como se describe en el [Cifrado de credenciales de almacenes de datos locales en Azure Data Factory](encrypt-credentials-self-hosted-integration-runtime.md) o en la aplicación de administrador de credenciales para establecer de forma segura credenciales para almacenes de datos locales en el entorno de ejecución de integración. |

![Requisitos de puerto de la puerta de enlace](media/data-movement-security-considerations/gateway-port-requirements.png) 

#### <a name="ip-configurations-and-allow-list-setting-up-in-data-stores"></a>Configuraciones de IP y configuración de lista de permitidos en almacenes de datos

Algunos almacenes de datos de la nube también requieren que permita la dirección IP de la máquina que accede a ellos. Asegúrese de que la dirección IP de la máquina con el entorno de ejecución de integración autohospedado tiene permiso en o está configurada en el firewall correctamente.

Los siguientes almacenes de datos en la nube requieren que permita la dirección IP de la máquina el entorno de ejecución de integración autohospedado. De forma predeterminada, es posible que algunos de estos almacenes de datos no necesiten lista de admitidas.

* [Azure SQL Database](../azure-sql/database/firewall-configure.md)
* [Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md)
* [Azure Data Lake Store](../data-lake-store/data-lake-store-secure-data.md#set-ip-address-range-for-data-access)
* [Azure Cosmos DB](../cosmos-db/how-to-configure-firewall.md)
* [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) 

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿El entorno de ejecución de integración autohospedado puede compartirse entre diferentes factorías de datos?**

Sí. Más detalles [aquí](https://azure.microsoft.com/blog/sharing-a-self-hosted-integration-runtime-infrastructure-with-multiple-data-factories/).

**¿Cuáles son los requisitos de puerto para que el entorno de ejecución de integración autohospedado funcione?**

Integration Runtime autohospedado establece conexiones basadas en HTTP para acceder a Internet. El puerto de salida 443 debe estar abierto para que el entorno de ejecución de integración autohospedado establezca la conexión. Abra el puerto de entrada 8060 solo en la máquina (no en el firewall corporativo) para la aplicación de administración de credenciales. Si se utiliza Azure SQL Database o Azure Synapse Analytics como origen o destino, tendrá que abrir también el puerto 1433. Para más información, consulte la sección [Configuraciones de firewall y configuración de la lista de permitidos para direcciones IP](#firewall-configurations-and-allow-list-setting-up-for-ip-addresses).

## <a name="next-steps"></a>Pasos siguientes

Para información sobre el rendimiento de la actividad de copia de Azure Data Factory, consulte la [Guía de optimización y rendimiento de la actividad de copia](copy-activity-performance.md).
