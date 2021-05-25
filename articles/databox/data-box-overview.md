---
title: Introducción a Microsoft Azure Data Box | Microsoft Docs en datos
description: Describe Azure Data Box, una solución en la nube que permite transferir grandes cantidades de datos en Azure
services: databox
documentationcenter: NA
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: overview
ms.date: 04/19/2021
ms.author: alkohli
ms.openlocfilehash: 6da5dfcded067420fd6d43f4a5b2841da060898a
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738129"
---
# <a name="what-is-azure-data-box"></a>¿Qué es Azure Data Box?

La solución en la nube Microsoft Azure Data Box le permite enviar terabytes de datos dentro y fuera de Azure de forma rápida, económica y confiable. La transferencia de datos segura se acelera mediante el envío de un dispositivo de almacenamiento de Data Box de propiedad. Cada dispositivo de almacenamiento tiene una capacidad de almacenamiento máximo utilizable de 80 TB y se transporta al centro de datos a través de un operador regional. El dispositivo tiene un sólido uso de mayúsculas y minúsculas para proteger los datos durante el tránsito.

Puede pedir el dispositivo Data Box a través de Azure Portal para importar o exportar datos desde Azure. Una vez recibido el dispositivo, puede configurarlo rápidamente mediante la interfaz de usuario web local. En función de si va a importar o exportar datos, copie los datos de los servidores en el dispositivo, o del dispositivo en los servidores, y vuelva a enviar el dispositivo a Azure. Si se importan datos en Azure, en el centro de datos de Azure, los datos se cargan automáticamente desde el dispositivo hasta Azure. El servicio de Data Box se encarga de realizar el seguimiento de todo el proceso en Azure Portal.

## <a name="use-cases"></a>Casos de uso

Data Box es ideal para transferir tamaños de datos con más de 40 TB en escenarios sin conectividad de red limitada. El movimiento de datos puede ser único, periódico o una transferencia de datos masiva inicial seguida de transferencias periódicas. 

Estos son los distintos escenarios donde se puede usar Data Box para importar datos en Azure.

 - **Migración única**: cuando se mueve gran cantidad de datos locales a Azure. 
     - Traslade una biblioteca multimedia de cintas sin conexión a Azure para crear una biblioteca multimedia en línea.
     - Migre la granja de máquinas virtuales, SQL Server y las aplicaciones a Azure.
     - Traslade los datos históricos a Azure para un análisis exhaustivo y generar informes con HDInsight.

 - **Transferencia masiva inicial**: cuando se realiza una transferencia masiva inicial con Data Box (inicialización) seguida de transferencias incrementales a través de la red. 
     - Por ejemplo, los asociados de soluciones de copia de seguridad, como Commvault y Data Box, se usan para mover la copia de seguridad histórica de gran tamaño inicial a Azure. Una vez completado el proceso, los datos incrementales se transfieren a través de la red a Azure Storage.

- **Cargas periódicas**: cuando se genera periódicamente una gran cantidad de datos y es necesario moverlos a Azure. Por ejemplo, en la exploración de energía, donde el contenido de vídeo se genera en plataformas petrolíferas y parques eólicos. 

Estos son los distintos escenarios donde se puede usar Data Box para exportar datos a Azure.

- **Recuperación ante desastres**: cuando se restaura una copia de los datos de Azure en una red local. En un escenario de recuperación ante desastres habitual, se exporta una gran cantidad de datos de Azure se exporta a Data Box. Microsoft luego los envía a Data Box y, en poco tiempo, los datos se restauran en un entorno local.

- **Requisitos de seguridad**: cuando necesita poder exportar datos de Azure debido a los requisitos de seguridad o de la administración pública. Por ejemplo, Azure Storage está disponible en las nubes US Secret y Top Secret, y se puede usar Data Box para exportar datos fuera de Azure. 

- **Migración de vuelta al entorno local o a otro proveedor de servicios en la nube**: cuando quiera mover todos los datos de vuelta al entorno local o a otro proveedor de servicios en la nube, exporte los datos a través de Data Box para migrar las cargas de trabajo.


## <a name="benefits"></a>Ventajas

Data Box está pensado para mover grandes cantidades de datos a Azure sin que afecte casi a la red. La solución tiene las siguientes ventajas:

- **Velocidad**: Data Box usa interfaces de red de 1 o 10 Gbps para mover introducir o sacar de Azure hasta 80 TB de datos.

- **Protección**: Data Box tiene protecciones de seguridad integradas para el dispositivo, los datos y el servicio.
  - El dispositivo tiene un sólido uso de mayúsculas y minúsculas protegido por tornillos resistentes a alteraciones y adhesivos de alteración evidente. 
  - Los datos en el dispositivo se protegen con un cifrado AES de 256 bits en todo momento.
  - El dispositivo solo se puede desbloquear con una contraseña proporcionada en Azure Portal.
  - El servicio está protegido por las características de seguridad de Azure.
  - Una vez que los datos del pedido de importación se cargan en Azure, los discos del dispositivo se limpian, según las normas NIST 800-88r1. Si el pedido es de exportación, los discos se borran una vez que el dispositivo llega al centro de datos de Azure.
    
    Para obtener más información, vaya a [Azure Data Box security and data protection](data-box-security.md) (Protección de datos y seguridad de Azure Data Box).

## <a name="features-and-specifications"></a>Características y especificaciones

El dispositivo Data Box tiene las siguientes características en esta versión.

| Especificaciones                                          | Descripción              |
|---------------------------------------------------------|--------------------------|
| Peso                                                  | < 50 libras                |
| Dimensions                                              | Dispositivo: Ancho: 309 mm Alto: 430,4 mm Fondo: 502 mm |            
| Espacio en bastidor                                              | 7 U cuando se coloca en el bastidor en su lado (no se puede montar en bastidor)|
| Se necesitan cables                                         | 1 cable de alimentación (incluido) <br> 2 cables RJ45 <br> 2 cables de cobre X SFP + Twinax|
| Capacidad de almacenamiento                                        | Un dispositivo de 100 TB que tenga 80 TB, de capacidad utilizable tras aplicar la protección de RAID 5|
| Potencia eléctrica                                            | La unidad de fuente de alimentación tiene una potencia nominal de 700 w. <br> Normalmente, la unidad utiliza 375 w.|
| Interfaces de red                                      | Dos interfaces de 1 GbE: MGMT y DATA 3. <br> MGMT: para la administración, no es configurable por el usuario y se usar para la configuración inicial <br> Data3: para los datos, es configurable por el usuario y es dinámico de forma predeterminada <br> MGMT y DATA 3 también pueden funcionar como 10 GbE <br> Dos interfaces de 10 GbE: DATA 1 y DATA 2 <br> Ambas son para los datos, se pueden configurar como estáticas o dinámicas (valor predeterminado) |
| Transferencia de datos                                      | Se admiten tanto importaciones como exportaciones.  |
| Transferencia de datos multimedia                                     | Cable de cobre RJ45 SFP+ y Ethernet de 10 GbE  |
| Seguridad                                                | Uso resistente de mayúsculas y minúsculas con tornillos personalizados a prueba de alteraciones <br> Adhesivos de alteración evidente colocados en la parte inferior del dispositivo|
| Velocidad de transferencia de datos                                      | Hasta 80 TB en un día a través de una interfaz de red de 10 GbE        |
| Administración                                              | Interfaz de usuario de web local: configuración inicial y única <br> Azure Portal: administración diaria de dispositivos        |

## <a name="data-box-components"></a>Componentes de Data Box

Data Box incluye los siguientes componentes:

* **Dispositivo Data Box**: un dispositivo físico que le proporciona almacenamiento principal, administra la comunicación con el almacenamiento en la nube y ayuda a garantizar la seguridad y confidencialidad de todos los datos almacenados en el dispositivo. El dispositivo Data Box tiene una capacidad de almacenamiento utilizable de 80 TB. 

    ![Plano frontal y trasero de Data Box](media/data-box-overview/data-box-combined.png)

    
* **Servicio Data Box**: es una extensión de Azure Portal que le permite administrar un dispositivo Data Box desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el servicio Data Box para realizar la administración diaria del dispositivo Data Box. Las tareas del servicio incluyen cómo crear y administrar pedidos, ver y administrar alertas y administrar recursos compartidos.  

    ![Servicio Data Box en Azure Portal](media/data-box-overview/data-box-service.png)

    Para obtener más información, consulte [Use the Data Box service to administer your Data Box device](data-box-portal-ui-admin.md) (Uso del servicio Data Box para administrar su dispositivo Data Box).

* **Interfaz de usuario web local**: es una interfaz de usuario basada en web que se usa para configurar el dispositivo; de esta manera, se podrá conectar a la red local y registrar el dispositivo con el servicio Data Box. Use también la interfaz de usuario web local para apagar y reiniciar el dispositivo Data Box, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft para realizar una solicitud de servicio.

    ![Interfaz de usuario web local de Data Box](media/data-box-overview/data-box-local-web-ui.png)

    Actualmente, la interfaz de usuario web local del dispositivo admite los siguientes idiomas con sus códigos de idioma correspondientes:

    | Idioma             | Código | Idioma                | Código   | Idioma                | Código         |
    |----------------------|------|-------------------------|--------|-------------------------|--------------|
    | Inglés {predeterminado}    | en   |  Checo                  | cs     | Alemán                  | de           |
    | Español              | es   | Francés                  | fr     | Húngaro               | hu           |
    | Italiano              | it   | Japonés                | ja     | Coreano                  | ko           |
    | Neerlandés                | nl   | Polaco                  | pl     | Portugués (Brasil)     | pt-br        |
    | Portugués (Portugal)| pt-pt| Ruso                 | ru     | Sueco                 | sv           |
    | Turco              | tr   | Chino (simplificado)    | zh-hans|    |       |    

    Para más información acerca de cómo usar la interfaz de usuario basada en web, consulte [Use the web-based UI to administer your Data Box](data-box-portal-ui-admin.md) (Uso de la interfaz de usuario web para administrar Data Box).

## <a name="the-workflow"></a>El flujo de trabajo

Un flujo de importación habitual incluye los siguientes pasos:

1. **Solicitar**: cree un pedido en Azure Portal, proporcione la información de envío y la cuenta de almacenamiento de Azure de destino para los datos. Si el dispositivo está disponible, Azure lo prepara y lo envía con un identificador de seguimiento del envío.

2. **Recibir**: una vez entregado el dispositivo, conéctelo a la red y enciéndalo con los cables especificados. Active el dispositivo y conéctese a él. Configure la red del dispositivo y monte los recursos compartidos en el equipo host desde donde desea copiar los datos.

3. **Copiar los datos**: copie los datos en los recursos compartidos de Data Box.

4. **Devolver**: prepare, desactive y devuelva el dispositivo a los centros de datos de Azure.

5. **Cargar**: los datos se copian automáticamente del dispositivo a Azure. Los discos del dispositivo se borran de forma segura según las directrices del Instituto Nacional de Normas y Tecnología (NIST).

Durante este proceso, se le notificará por correo electrónico de todos los cambios de estado. Para más información sobre el flujo detallado, consulte [Cómo implementar instancias de Data Box en Azure Portal](data-box-deploy-ordered.md).


Un flujo de exportación habitual incluye los siguientes pasos:

1. **Pedido**: cree un pedido de exportación en Azure Portal, especifique la información de envío y la cuenta de Azure Storage de origen de los datos. Si el dispositivo está disponible, Azure prepara un dispositivo. Los datos se copian de la cuenta de Azure Storage a Data Box. Una vez completada la copia de datos, Microsoft envía el dispositivo con un identificador de seguimiento de envío.

2. **Recibir**: una vez entregado el dispositivo, conéctelo a la red y enciéndalo con los cables especificados. Active el dispositivo y conéctese a él. Configure la red del dispositivo y monte los recursos compartidos en el equipo host en el que quiere copiar los datos.

3. **Copia de los datos**: copie los datos desde los recursos compartidos de Data Box hasta los servidores de datos locales.

4. **Devolver**: prepare, desactive y devuelva el dispositivo a los centros de datos de Azure.

5. **Borrado de datos**: los discos del dispositivo se borran de forma segura según las directrices del Instituto Nacional de Normas y Tecnología (NIST).

Durante este proceso, se le avisará por correo electrónico de todos los cambios de estado. Para más información sobre el flujo detallado, consulte [Cómo implementar instancias de Data Box en Azure Portal](data-box-deploy-export-ordered.md).

## <a name="region-availability"></a>Disponibilidad en regiones

Data Box puede transferir datos en función de la región en la que se implementa el servicio, el país o región al que se envía el dispositivo y la cuenta de Azure Storage de destino a la que se transfieren los datos.

### <a name="for-import"></a>Para realizar la importación

- **Disponibilidad del servicio**: cuando se use Data Box para pedidos de importación o exportación, puede obtener información sobre la disponibilidad por región en [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all).

    En el caso de los pedidos de exportación, Data Box también se puede implementar en la nube de Azure Government. Para más información, consulte [¿Qué es Azure Government?](../azure-government/documentation-government-welcome.md). 

- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure donde está disponible el servicio.


## <a name="next-steps"></a>Pasos siguientes

- Revise [los requisitos del sistema Data Box](data-box-system-requirements.md).
- Información acerca de los [límites de Data Box](data-box-limits.md).
- Implemente rápidamente [Azure Data Box](data-box-quickstart-portal.md) en Azure Portal.
