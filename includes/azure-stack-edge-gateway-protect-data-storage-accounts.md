---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/08/2021
ms.author: alkohli
ms.openlocfilehash: 4740ce4bd59598747cdd9c2147fbbd460b6e648e
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107285216"
---
Su dispositivo está asociado a una cuenta de almacenamiento que se utiliza como destino para los datos en Azure. El acceso a la cuenta de almacenamiento se controla mediante la suscripción y las dos claves de acceso de almacenamiento de 512 bits asociadas a esa cuenta de almacenamiento.

Una de las claves se utiliza para la autenticación cuando el dispositivo Azure Stack Edge accede a la cuenta de almacenamiento. La otra clave se mantiene en reserva, por lo que puede rotar periódicamente las claves.

Por motivos de seguridad, muchos centros de datos requieren que las claves se roten. Se recomienda seguir estos procedimientos recomendados para la rotación de claves:

- La clave de la cuenta de almacenamiento es similar a la contraseña raíz de la cuenta de almacenamiento. Proteja cuidadosamente la clave de cuenta. No distribuya la contraseña a otros usuarios, no la codifique ni la guarde en cualquier lugar en texto sin formato que sea accesible para otros.
- [Vuelva a generar la clave de cuenta](../articles/storage/common/storage-account-keys-manage.md#manually-rotate-access-keys) mediante Azure Portal si cree podría estar en peligro.
- Periódicamente, el administrador de Azure debe cambiar o volver a generar la clave principal o secundaria mediante la sección Storage de Azure Portal para acceder directamente a la cuenta de almacenamiento.
- Puede usar también su propia clave de cifrado para proteger los datos de la cuenta de almacenamiento. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. Para más información sobre cómo proteger los datos, consulte [Habilitación de claves administradas por el cliente para una cuenta de almacenamiento](../articles/storage/common/customer-managed-keys-overview.md#enable-customer-managed-keys-for-a-storage-account).
