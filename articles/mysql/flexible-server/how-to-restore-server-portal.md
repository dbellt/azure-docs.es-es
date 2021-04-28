---
title: Restauración de un servidor flexible de Azure Database for MySQL con Azure Portal
description: En este artículo se explica cómo realizar operaciones de restauración en el servidor flexible de Azure Database for MySQL mediante Azure Portal.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 04/01/2021
ms.openlocfilehash: 962a2cbdbcc238517616c9ade235eed9b8cae6f7
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502052"
---
# <a name="point-in-time-restore-of-a-azure-database-for-mysql---flexible-server-preview-using-azure-portal"></a>Restauración a un momento dado de Azure Database for MySQL: servidor flexible (versión preliminar) con Azure Portal


> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

En este artículo se proporciona un procedimiento detallado para llevar a cabo recuperaciones a un momento dado en servidores flexibles mediante copias de seguridad.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Debe tener Azure Database for MySQL: servidor flexible.

## <a name="restore-to-the-latest-restore-point"></a>Recuperación al punto de restauración más reciente

Siga estos pasos para restaurar un servidor flexible utilizando su copia de seguridad más antigua.

1.  En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible cuya copia de seguridad quiera restaurar.

2.  En el panel izquierdo, haga clic en **Información general**.

3.  En la página Información general, haga clic en **Restaurar**.

4.  Se mostrará la página de restauración con una opción para elegir entre el **punto de restauración más reciente** y uno personalizado.

5.  Seleccione el **punto de restauración más reciente**.

6.  Indique un nombre de servidor nuevo en el campo **Restaurar en el servidor nuevo**.

    :::image type="content" source="./media/concept-backup-restore/restore-blade-latest.png" alt-text="Punto de restauración más antiguo":::

8.  Haga clic en **Aceptar**.

9.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.

## <a name="restoring-to-a-custom-restore-point"></a>Restauración a un punto de restauración personalizado

Siga estos pasos para restaurar un servidor flexible mediante la copia de seguridad más antigua.

1.  En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible cuya copia de seguridad quiera restaurar.

2.  En la página Información general, haga clic en **Restaurar**.

3.  Se mostrará la página de restauración con una opción para elegir entre el primer punto de restauración y uno personalizado.

4.  Elija un **punto de restauración personalizado**.

5.  Seleccione la fecha y la hora.

6.  Indique un nombre de servidor nuevo en el campo **Restaurar en el servidor nuevo**.

6.  Indique un nombre de servidor nuevo en el campo **Restaurar en el servidor nuevo**.

    :::image type="content" source="./media/concept-backup-restore/restore-blade-custom.png" alt-text="ver información general":::

7.  Haga clic en **Aceptar**.

8.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.


## <a name="perform-post-restore-tasks"></a>Tareas posteriores a la restauración
Una vez finalizada la restauración, deberá realizar las siguientes tareas para que los usuarios y las aplicaciones vuelvan a conectarse:

- Si el nuevo servidor está destinado a reemplazar al original, redirija a los clientes y las aplicaciones cliente al nuevo servidor.
- Asegúrese de que haya vigentes reglas de red virtual adecuadas para que los usuarios se conecten. Estas reglas no se copian desde el servidor original.
- No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
- Configure las alertas según corresponda para el servidor recién restaurado.


## <a name="next-steps"></a>Pasos siguientes
Más información sobre la [continuidad empresarial](concepts-business-continuity.md).
