---
title: Introducción a la copia de seguridad operativa para blobs de Azure
description: Obtenga más información sobre la copia de seguridad operativa para blobs de Azure.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: 5d6113108a41430e9d864ff05095be4d440989ee
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108767246"
---
# <a name="overview-of-operational-backup-for-azure-blobs"></a>Introducción a la copia de seguridad operativa para blobs de Azure

La copia de seguridad operativa de blobs es una solución de protección de datos local administrada que le permite proteger los blobs en bloques de diversos escenarios en los que se hayan perdido datos; por ejemplo, si se producen daños, si se realizan eliminaciones de blobs y la eliminación accidental de cuentas de almacenamiento. Los datos se almacenan localmente dentro de la propia cuenta de almacenamiento de origen y se pueden recuperar a un momento determinado en el tiempo cuando sea necesario. Por tanto, proporciona un medio sencillo, seguro y rentable para proteger sus blobs.

La copia de seguridad operativa de blobs se integra con el [Centro de copias de seguridad](backup-center-overview.md), entre otras funcionalidades de administración de Azure Backup, que permite controlar, supervisar, usar y analizar las copias de seguridad a escala desde un solo panel.

## <a name="how-operational-backup-works"></a>Funcionamiento de la copia de seguridad operativa

La copia de seguridad operativa de blobs es una solución de **copia de seguridad local**. Por lo tanto, los datos de la copia de seguridad no se transfieren al almacén de Azure Backup, sino que se almacenan en la propia cuenta de almacenamiento de origen. Sin embargo, el almacén de Azure Backup sigue siendo la unidad de administración de las copias de seguridad. Además, se trata de una solución de **copia de seguridad continua**, lo que significa que no es necesario programar ninguna copia de seguridad y que todos los cambios se conservarán y restaurarán al estado que haya en un momento determinado.

La copia de seguridad operativa utiliza funcionalidades de la plataforma de blobs para proteger los datos y permitir la recuperación cuando sea necesario:

- **Restauración a un momento dado**: la [restauración de blobs a un momento dado](../storage/blobs/point-in-time-restore-overview.md) permite restaurar los datos de blobs a un estado anterior. A su vez, esto usa las funciones de **eliminación temporal**, **fuente de cambios** y **control de versiones de blobs** para conservar los datos durante el tiempo especificado. La copia de seguridad operativa se encarga de habilitar la restauración a un momento dado, así como las funcionalidades subyacentes, para que los datos se conserven durante el tiempo especificado.

- **Bloqueo de la eliminación**: el bloqueo de la eliminación impide que usuarios no autorizados eliminen la cuenta de almacenamiento o que se elimine accidentalmente. Cuando se configura la copia de seguridad operativa, también aplica automáticamente un bloqueo de la eliminación para reducir las posibilidades de pérdida de datos debida a la eliminación de la cuenta de almacenamiento.

Consulte la [tabla de compatibilidad](blob-backup-support-matrix.md) para obtener información sobre las limitaciones de la solución actual.

### <a name="protection"></a>Protección

La copia de seguridad operativa se configura y se administra en el nivel de la **cuenta de almacenamiento**, y se aplica a todos los blobs en bloques de ella. La copia de seguridad operativa usa una **directiva de copia de seguridad** para administrar la duración de la retención de los datos de copia de seguridad (incluidas las versiones anteriores y los blobs eliminados). De esta manera, se define el período durante el cual se podrán restaurar los datos. La directiva de copia de seguridad puede tener una retención máxima de 360 días, o un número equivalente de semanas completas (51) o meses (11).

Cuando se configura la copia de seguridad de una cuenta de almacenamiento y se le asigna una directiva de copia de seguridad con una retención de "n" días, las propiedades subyacentes se establecen tal y como se describe a continuación. Puede ver estas propiedades en la pestaña **Protección de datos** de Blob Service, en la cuenta de almacenamiento.

- **Restauración a un momento dado**: establézcalo en "n" días, tal y como se define en la directiva de copia de seguridad. Si la cuenta de almacenamiento ya tenía habilitada la restauración a un momento dado con una retención de, por ejemplo, "x" días antes de configurar la copia de seguridad, la duración de la restauración a un momento dado se establecerá en el mayor de los dos valores, que es max(n,x). Si ya había habilitado la restauración a un momento dado y había especificado una retención mayor que la de la directiva de copia de seguridad, no se hará ningún cambio.

- **Eliminación temporal**: establézcalo en "n+5" días, es decir, cinco días más que la duración especificada en la directiva de copia de seguridad. Si la cuenta de almacenamiento que se está configurando para la copia de seguridad operativa ya tiene habilitada la eliminación temporal con una retención de, por ejemplo, "y" días, la retención de eliminación temporal se establecerá en el máximo de los dos valores, es decir, max(n+5,y). Si ya había habilitado la eliminación temporal y había especificado una retención mayor que la de la directiva de copia de seguridad, no se hará ningún cambio.

- **Control de versiones de blobs y fuente de cambios de blob:** el control de versiones y la fuente de cambios están habilitados para las cuentas de almacenamiento configuradas para copia de seguridad operativa.

- **Bloqueo de la eliminación:** la configuración de la copia de seguridad operativa en una cuenta de almacenamiento también aplica un bloqueo de la eliminación a la cuenta de almacenamiento. El bloqueo de la eliminación que la copia de seguridad aplica se puede ver en la pestaña **Bloqueos** de la cuenta de almacenamiento.

Para que la copia de seguridad habilite estas propiedades en las cuentas de almacenamiento que se van a proteger, se debe conceder al almacén de copia de seguridad el rol **Colaborador de copias de seguridad de la cuenta de almacenamiento** en las cuentas de almacenamiento correspondientes.

>[!NOTE]
>La copia de seguridad operativa solo admite operaciones en blobs en bloques, y las operaciones en contenedores no se pueden restaurar. Si elimina un contenedor de la cuenta de almacenamiento mediante una llamada a la operación **Eliminar contenedor**, ese contenedor no se puede restaurar con una operación de restauración. Se recomienda habilitar la eliminación temporal para mejorar la protección y recuperación de los datos.

### <a name="management"></a>Administración

Una vez que haya habilitado la copia de seguridad en una cuenta de almacenamiento, se creará la instancia de copia de seguridad correspondiente a la cuenta de almacenamiento en el almacén de copias de seguridad. Puede realizar todas las operaciones de copia de seguridad de una cuenta de almacenamiento, como iniciar restauraciones, supervisar, detener la protección, etc., mediante su instancia de copia de seguridad correspondiente.

La copia de seguridad operativa también se integra directamente en el Centro de copia de seguridad para ayudarle a administrar de forma centralizada la protección de todas las cuentas de almacenamiento, además de todas las otras cargas de trabajo compatibles con copias de seguridad. El Centro de copia de seguridad es un panel único para todos sus requisitos de copia de seguridad: supervisar los trabajos y el estado de las copias de seguridad y restauraciones, garantizar el cumplimiento y la gobernanza, analizar el uso de las copias de seguridad y ejecución de operaciones relacionadas con la copia de seguridad y restauración de los datos.

### <a name="restore"></a>Restauración

Puede restaurar los datos de cualquier punto en el tiempo para el cual exista un punto de recuperación. Los puntos de recuperación se crean cuando una cuenta de almacenamiento se encuentra en estado protegido, y se pueden usar para restaurar datos siempre que se encuentren dentro del período de retención definido por la directiva de copia de seguridad (y por la funcionalidad de restauración a un momento dado de Blob Service en la cuenta de almacenamiento). La copia de seguridad operativa usa la restauración a un momento dado de blobs para restaurar los datos de un punto de recuperación.

La copia de seguridad operativa ofrece la opción de restaurar todos los blobs en bloques de la cuenta de almacenamiento, examinar y restaurar contenedores específicos, o usar coincidencias de prefijos para restaurar un subconjunto de blobs. Las restauraciones solo se pueden realizar en la cuenta de almacenamiento de origen.

## <a name="pricing"></a>Precios

No se incurrirá en cargos de administración o tarifas por instancia cuando se use la copia de seguridad operativa para blobs. Sin embargo, incurrirá en cargos por los siguientes motivos:

- Las restauraciones se realizan mediante la restauración de blobs a un momento dado e incurren en cargos en función de la cantidad de datos procesados. Para más información, consulte los [precios de la restauración a un momento dado](../storage/blobs/point-in-time-restore-overview.md#pricing-and-billing).

- Retención de los datos debida a las funciones de [eliminación temporal de blobs](../storage/blobs/soft-delete-blob-overview.md), [compatibilidad con la fuente de cambios en Azure Blob Storage](../storage/blobs/storage-blob-change-feed.md) y [control de versiones de blobs](../storage/blobs/versioning-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración y administración de copias de seguridad de blobs de Azure](blob-backup-configure-manage.md)