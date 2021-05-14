---
title: Consideraciones de seguridad | Microsoft Docs
description: En este tema se proporcionan instrucciones generales para proteger SQL Server cuando se ejecuta en una máquina virtual de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: d710c296-e490-43e7-8ca9-8932586b71da
ms.service: virtual-machines-sql
ms.subservice: security
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/23/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: ab51a22abd5dac99a1a1bd3be5324e5d57c8e12e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145616"
---
# <a name="security-considerations-for-sql-server-on-azure-virtual-machines"></a>Consideraciones de seguridad para SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este tema se incluyen instrucciones de seguridad generales que ayudan a establecer el acceso seguro a instancias de SQL Server en una máquina virtual de Azure.

Azure cumple diversos reglamentos y estándares del sector que permiten compilar una solución compatible con SQL Server que se ejecuta en una máquina virtual. Para obtener información acerca del cumplimiento normativo de Azure, consulte [Centro de confianza de Azure](https://azure.microsoft.com/support/trust-center/).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="control-access-to-the-sql-virtual-machine"></a>Control del acceso a la máquina virtual de SQL

Cuando se crea la máquina virtual de SQL Server, tenga en cuenta cómo controlar con cuidado quién tiene acceso a la máquina y a SQL Server. En general, siempre debe seguir estos pasos:

- Restringir el acceso a SQL Server solo a las aplicaciones y los clientes que lo necesiten.
- Seguir las prácticas recomendadas para administrar las contraseñas y cuentas de usuario.

En las secciones siguientes se proporcionan sugerencias relacionadas con estos puntos.

## <a name="secure-connections"></a>Conexiones seguras

Cuando se crea una máquina virtual de SQL Server con una imagen de la galería, la opción **Conectividad de SQL Server** le permite elegir entre **Local (dentro de la máquina virtual)** , **Privada (dentro de la red virtual)** o **Pública (Internet)** .

![Conectividad de SQL Server](./media/security-considerations-best-practices/sql-vm-connectivity-option.png)

Para obtener una mayor seguridad, elija la opción más restrictiva para su escenario. Por ejemplo, si está ejecutando una aplicación que tiene acceso a SQL Server en la misma máquina virtual, entonces **Local** es la opción más segura. Si está ejecutando una aplicación de Azure que requiere acceso a SQL Server, **Privada** protege la comunicación con SQL Server solo dentro de la [red virtual de Azure](../../../virtual-network/virtual-networks-overview.md) especificada. Si necesita acceso **Público** (Internet) a la máquina virtual de SQL Server, asegúrese de seguir otras prácticas recomendadas en este tema para reducir el área expuesta a los ataques.

Las opciones seleccionadas en el portal usan reglas de seguridad de entrada en el [grupo de seguridad de red](../../../active-directory/identity-protection/concept-identity-protection-security-overview.md) (NSG) de las máquinas virtuales para permitir o denegar el tráfico de red a la máquina virtual. Puede modificar o crear nuevas reglas NSG de entrada para permitir el tráfico al puerto de SQL Server (el predeterminado es 1433). También puede especificar direcciones IP concretas que tengan permiso para comunicarse a través de este puerto.

![Reglas del grupo de seguridad de red](./media/security-considerations-best-practices/sql-vm-network-security-group-rules.png)

Además de las reglas NSG para restringir el tráfico de red, también puede usar el Firewall de Windows en la máquina virtual.

Si está usando puntos de conexión con el modelo de implementación clásica, quite de la máquina virtual todos los puntos de conexión que no use. Para obtener instrucciones sobre el uso de ACL con puntos de conexión, consulte [Administrar la ACL en un punto de conexión](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints#manage-the-acl-on-an-endpoint). Esto no es necesario en máquinas virtuales que usan Azure Resource Manager.

Por último, considere la posibilidad de habilitar conexiones cifradas para la instancia del motor de base de datos de SQL Server en la máquina virtual de Azure. Configure la instancia de SQL Server con un certificado firmado. Para más información, consulte [Habilitar conexiones cifradas en el motor de base de datos](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine) y [Sintaxis de cadena de conexión](/dotnet/framework/data/adonet/connection-string-syntax).

## <a name="encryption"></a>Cifrado

Los discos administrados ofrecen cifrado del lado servidor y Azure Disk Encryption. El [cifrado del lado servidor](../../../virtual-machines/disk-encryption.md) proporciona cifrado en reposo y protege los datos con el fin de cumplir con los compromisos de cumplimiento y seguridad de su organización. [Azure Disk Encryption](../../../security/fundamentals/azure-disk-encryption-vms-vmss.md) usa la tecnología BitLocker o DM-Crypt y se integra con Azure Key Vault para cifrar los discos de datos y del sistema operativo. 

## <a name="use-a-non-default-port"></a>Usar un puerto no predeterminado

De forma predeterminada, SQL Server escucha en un puerto conocido, 1433. Para aumentar la seguridad, configure SQL Server para que escuche en un puerto no predeterminado, como 1401. Si se aprovisiona una imagen de la galería de SQL Server en Azure Portal, puede especificar este puerto en la hoja **Configuración de SQL Server**.

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Para configurar esto después del aprovisionamiento, tiene dos opciones:

- Para las máquinas virtuales de Resource Manager, puede seleccionar **Seguridad** en el [recurso máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). Esto proporciona una opción para cambiar el puerto.

  ![Cambiar el puerto TCP en el portal](./media/security-considerations-best-practices/sql-vm-change-tcp-port.png)

- Para las máquinas virtuales clásicas o las máquinas virtuales de SQL Server que no se aprovisionaron con el portal, puede configurar manualmente el puerto mediante la conexión de forma remota a la máquina virtual. Para obtener los pasos de configuración, vea [Configure a Server to Listen on a Specific TCP Port](/sql/database-engine/configure-windows/configure-a-server-to-listen-on-a-specific-tcp-port) (Configurar un servidor para que escuche en un puerto TCP específico). Si usa esta técnica manual, también debe agregar una regla de Firewall de Windows para permitir el tráfico entrante en ese puerto TCP.

> [!IMPORTANT]
> La especificación de un puerto no predeterminado es una buena idea si el puerto de SQL Server está abierto para las conexiones públicas a Internet.

Cuando SQL Server escucha en un puerto no predeterminado, debe especificar el puerto al conectarse. Por ejemplo, considere un escenario donde la dirección IP del servidor es 13.55.255.255 y SQL Server está escuchando en el puerto 1401. Para conectarse a SQL Server, debe especificar `13.55.255.255,1401` en la cadena de conexión.

## <a name="manage-accounts"></a>Administrar cuentas

No querrá que los atacantes adivinen fácilmente los nombres de las cuentas o las contraseñas. Use las sugerencias siguientes como ayuda para:

- Cree una cuenta de administrador local única que no se llame **Administrador**.

- Use contraseñas seguras complejas para todas sus cuentas. Para más información sobre cómo crear una contraseña segura, vea el artículo [Crear una contraseña segura](https://support.microsoft.com/account-billing/how-to-create-a-strong-password-for-your-microsoft-account-f67e4ddd-0dbe-cd75-cebe-0cfda3cf7386).

- De forma predeterminada, Azure selecciona la Autenticación de Windows durante la instalación de la máquina virtual de SQL Server. Por lo tanto, el inicio de sesión de **SA** está deshabilitado y el programa de instalación asigna una contraseña. Se recomienda no usar ni habilitar el inicio de sesión de **SA**. Si debe tener un inicio de sesión de SQL, use una de las estrategias siguientes:

  - Cree una cuenta de SQL con un nombre único que sea miembro de **sysadmin**. Puede hacerlo desde el portal si habilita **Autenticación de SQL** durante el aprovisionamiento.

    > [!TIP] 
    > Si no habilita la autenticación de SQL durante el aprovisionamiento, debe cambiar manualmente el modo de autenticación a **Modo de autenticación de Windows y SQL Server**. Para obtener más información, consulte [Cambiar el modo de autenticación del servidor](/sql/database-engine/configure-windows/change-server-authentication-mode).

  - Si tiene que usar el inicio de sesión de **SA**, habilite el inicio de sesión después del aprovisionamiento y asigne una nueva contraseña segura.

## <a name="additional-best-practices"></a>Procedimientos recomendados adicionales

Además de los procedimientos descritos en este tema, le recomendamos que revise e implemente los procedimientos recomendados de seguridad, entre los que destacamos tanto los tradicionales que se relacionan con la seguridad local como los de seguridad de las máquinas virtuales. 

Para obtener más información sobre los procedimientos de seguridad locales, consulte [Consideraciones de seguridad para una instalación de SQL Server](/sql/sql-server/install/security-considerations-for-a-sql-server-installation) y [Centro de seguridad](/sql/relational-databases/security/security-center-for-sql-server-database-engine-and-azure-sql-database). 

Para obtener más información sobre la seguridad de las máquinas virtuales, consulte la [información general sobre la seguridad de las máquinas virtuales](../../../security/fundamentals/virtual-machines-overview.md).


## <a name="next-steps"></a>Pasos siguientes

Si también está interesado en los procedimientos recomendados de rendimiento, consulte [Directrices de rendimiento para SQL Server en Azure Virtual Machines](./performance-guidelines-best-practices-checklist.md).

Para ver otros temas sobre la ejecución de SQL Server en Azure Virtual Machines, consulte [Información general sobre SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).