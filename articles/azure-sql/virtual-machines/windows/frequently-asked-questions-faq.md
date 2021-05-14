---
title: Preguntas más frecuentes sobre SQL Server en Windows Virtual Machines de Azure | Microsoft Docs
description: Este artículo brinda respuestas a las preguntas más frecuentes sobre cómo ejecutar SQL Server en Máquinas virtuales de Azure.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: virtual-machines-sql
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/05/2019
ms.author: mathoma
ms.openlocfilehash: 02bb1f539369cf72a5d5b6503a3584069589b19e
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727355"
---
# <a name="frequently-asked-questions-for-sql-server-on-azure-vms"></a>Preguntas más frecuentes sobre SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](frequently-asked-questions-faq.md)
> * [Linux](../linux/frequently-asked-questions-faq.md)

En este artículo se ofrecen respuestas a algunas de las preguntas más comunes sobre la ejecución de [SQL Server en Microsoft Azure Virtual Machines (VM)](https://azure.microsoft.com/services/virtual-machines/sql-server/).

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="images"></a><a id="images"></a> Imágenes

1. **¿Qué imágenes de la galería de máquinas virtuales de SQL Server están disponibles?** 

   Azure mantiene imágenes de máquinas virtuales para todas las versiones principales de SQL Server admitidas en todas las ediciones para Windows y Linux. Para obtener más información, consulte la lista completa de [imágenes de VM de Windows](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo) e [imágenes de VM de Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md#create).

1. **¿Están actualizadas las imágenes existentes de la galería de máquinas virtuales de SQL Server?**

   Cada dos meses, las imágenes de SQL Server de la galería de máquinas virtuales se actualizan con las últimas actualizaciones de Windows y Linux. En el caso de las imágenes de Windows, esto incluye cualquier actualización que se marque como importante en Windows Update, incluidas las actualizaciones de seguridad importantes de SQL Server y los Service Pack. En las imágenes de Linux, esto incluye las últimas actualizaciones del sistema. Las actualizaciones acumulativas de SQL Server se tratan de forma diferente para Linux y Windows. En Linux, las actualizaciones acumulativas de SQL Server también se incluyen en la actualización. Pero en este momento, las máquinas virtuales de Windows no se actualizan con las actualizaciones acumulativas de SQL Server o Windows Server.

1. **¿Se pueden quitar las imágenes de máquinas virtuales de SQL Server de la galería?**

   Sí. Azure solo mantiene una imagen por edición y versión principal. Por ejemplo, cuando se lanza un nuevo Service Pack de SQL Server, Azure agrega una nueva imagen a la galería para dicho Service Pack. La imagen SQL Server para el Service Pack anterior se quita inmediatamente de Azure Portal. Sin embargo, todavía está disponible desde PowerShell para su aprovisionamiento durante los tres meses siguientes. Transcurridos tres meses, la imagen del Service Pack anterior dejará de estar disponible. Esta directiva de eliminación también se aplicaría si una versión de SQL Server no recibe soporte técnico cuando llega al final de su ciclo de vida.


1. **¿Es posible implementar una imagen anterior de SQL Server que no sea visible en Azure Portal?**

   Sí, mediante PowerShell. Para obtener más información sobre cómo implementar VM con SQL Server mediante PowerShell, consulte [Aprovisionamiento de máquinas virtuales de SQL Server con Azure PowerShell](create-sql-vm-powershell.md).
   
1. **¿Se puede crear una imagen de SQL Server de Azure Marketplace generalizada de mi VM con SQL Server y usarla para implementar las VM?**

   Sí, pero debe [registrar las VM con SQL Server con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para administrarlas en el portal y poder usar características como la aplicación automática de revisiones y las copias de seguridad automáticas. Al registrar con la extensión, también deberá especificar el tipo de licencia para cada VM con SQL Server.

1. **¿Cómo puedo generalizar SQL Server en VM de Azure y usarlo para implementar nuevas VM?**

   Puede implementar una máquina virtual de Windows Server (sin SQL Server instalado) y usar el proceso [sysprep de SQL](/sql/database-engine/install-windows/install-sql-server-using-sysprep) para generalizar SQL Server en VM de Azure (Windows) con los medios de instalación de SQL Server. Los clientes que tengan [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot%3aprimaryr3) pueden obtener el soporte de instalación en el [centro de licencias por volumen](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Los clientes que no tengan Software Assurance pueden usar el soporte de instalación de una imagen de VM con SQL Server de Azure Marketplace que tenga la edición que deseen.

   Como alternativa, use una de las imágenes de SQL Server de Azure Marketplace para generalizar SQL Server en las VM de Azure. Tenga en cuenta que debe eliminar la siguiente clave del Registro en la imagen de origen antes de crear su propia imagen. Si no lo hace, se puede producir un sobredimensionamiento de la carpeta de arranque del programa de instalación de SQL Server o un estado de error en la extensión de IaaS de SQL.

   Ruta de acceso a la clave del Registro:  
   `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\SysPrepExternal\Specialize`

   > [!NOTE]
   > Se recomienda que todas las VM con SQL Server en Azure, incluidas las implementadas a partir de imágenes generalizadas personalizadas, estén [registradas con la extensión Agente de IaaS de SQL](./sql-agent-extension-manually-register-single-vm.md?tabs=azure-cli%252cbash) para satisfacer los requisitos de cumplimiento y para usar características opcionales como la aplicación automatizada de revisiones y las copias de seguridad automáticas. La extensión también le permite [especificar el tipo de licencia](./licensing-model-azure-hybrid-benefit-ahb-change.md?tabs=azure-portal) para cada VM con SQL Server.

1. **¿Puedo usar mi propio disco duro virtual para implementar una máquina virtual con SQL Server?**

   Sí, pero debe [registrar las VM con SQL Server con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para administrarlas en el portal y poder usar características como la aplicación automática de revisiones y las copias de seguridad automáticas.

1. **¿Es posible configurar configuraciones que no aparecen en la galería de máquinas virtuales (por ejemplo, Windows 2008 R2 + SQL Server 2012)?**

   No. En el caso de las imágenes de la galería de máquinas virtuales que incluyen SQL Server, debe seleccionar una de las imágenes que se proporcionan a través de Azure Portal o mediante [PowerShell](create-sql-vm-powershell.md). No obstante, tiene la posibilidad de implementar una máquina virtual Windows e instalar automáticamente SQL Server en ella. Debe [registrar la VM con SQL Server con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para administrarla en Azure Portal y poder usar características como la aplicación automática de revisiones y las copias de seguridad automáticas. 


## <a name="creation"></a>Creación

1. **¿Cómo puedo crear una máquina virtual de Azure con SQL Server?**

   El método más simple es crear una máquina virtual que incluya SQL Server. Para ver un tutorial sobre cómo suscribirse a Azure y crear una máquina virtual con SQL Server desde el portal, consulte [Aprovisionamiento de una máquina virtual con SQL Server en Azure Portal](create-sql-vm-portal.md). Puede seleccionar una imagen de máquina virtual que use licencias de SQL Server de pago por segundo o puede utilizar una imagen que le permita traer su propia licencia de SQL Server. También tiene la opción de instalar manualmente SQL Server en una máquina virtual con una edición de licencia libre (Developer o Express) o reutilizando una licencia local. Asegúrese de [registrar su VM con SQL Server con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para administrarla en el portal y poder usar características como la aplicación automática de revisiones y las copias de seguridad automáticas. Si trae su propia licencia, debe disponer de [Movilidad de Licencias a través de Software Assurance en Azure](https://azure.microsoft.com/pricing/license-mobility/). Para más información, vea [Pricing guidance for SQL Server Azure VMs](pricing-guidance.md) (Orientación de precios de máquinas virtuales de SQL Server Azure).

1. **¿Cómo puedo migrar mi base de datos local de SQL Server a la nube?**

   En primer lugar, cree una máquina virtual de Azure con una instancia de SQL Server. Luego, migre las bases de datos locales a esa instancia. Para conocer las estrategias de migración de datos, consulte [Migración de una Base de datos SQL Server a SQL Server en una máquina virtual de Azure](migrate-to-vm-from-sql-server.md).

## <a name="licensing"></a>Licencias

1. **¿Cómo puedo instalar mi copia de SQL Server con licencia en una máquina virtual de Azure?**

   Existen tres formas de hacerlo. Si tiene un Contrato Enterprise (EA), puede aprovisionar una de las [imágenes de máquina virtual que admita licencias](sql-server-on-azure-vm-iaas-what-is-overview.md#BYOL), lo que también se conoce como "traiga su propia licencia" (BYOL). Si tiene [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-default), puede habilitar la [Ventaja híbrida de Azure](licensing-model-azure-hybrid-benefit-ahb-change.md) en una imagen de pago por uso existente. También puede copiar los medios de instalación de SQL Server en una máquina virtual de Windows Server y, luego, instalar SQL Server en la máquina virtual. Asegúrese de registrar la máquina virtual con SQL Server con la [extensión](sql-agent-extension-manually-register-single-vm.md) para usar características como la administración de portal, las copias de seguridad automáticas y la aplicación automatizada de revisiones. 


1. **¿Necesita un cliente licencias de acceso de cliente (CAL) de SQL Server para conectarse a una imagen de pago por uso de SQL Server que se ejecuta en Azure Virtual Machines?**

   No. Los clientes necesitan licencias CAL cuando usan "traiga su propia licencia" y mueven su servidor SA o máquina virtual CAL de SQL Server a máquinas virtuales de Azure. 

1. **¿Puedo cambiar una máquina virtual para usar mi propia licencia de SQL Server si se ha creado desde una de las imágenes de la galería de pago por uso?**

   Sí. Puede cambiar fácilmente una imagen de la galería de pago por uso a traiga su propia licencia (BYOL) habilitando la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/faq/).  Para más información, consulte [Modificación del modelo de licencia para una máquina virtual de SQL Server en Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). Actualmente, este recurso solo está disponible para los clientes de la nube pública y la nube de Azure Government.


1. **¿El cambio de modelos de licencia requerirá un tiempo de inactividad para SQL Server?**

   No. El [cambio del modelo de licencias](licensing-model-azure-hybrid-benefit-ahb-change.md) no requiere ningún tiempo de inactividad para SQL Server, ya que el cambio es efectivo de inmediato y no requiere un reinicio de la VM. Sin embargo, para registrar la VM con SQL Server con la extensión Agente de IaaS de SQL, la [extensión IaaS de SQL](sql-server-iaas-agent-extension-automate-management.md) es un requisito previo y la instalación de la extensión IaaS de SQL en el modo _completo_ reinicia el servicio SQL Server. Por lo tanto, si es necesario instalar la extensión IaaS de SQL, instálela en modo _ligero_ para una funcionalidad limitada o instálela en modo _completo_ durante una ventana de mantenimiento. La extensión IaaS de SQL instalada en modo _ligero_ se puede actualizar al modo _completo_ en cualquier momento, pero requiere el reinicio del servicio SQL Server. 
   
1. **¿Es posible cambiar los modelos de licencias en una VM con SQL Server implementada con el modelo clásico?**

   No. No es posible cambiar los modelos de licencias en una VM clásica. Puede migrar la máquina virtual al modelo de Azure Resource Manager y registrarla con la extensión Agente de IaaS de SQL. Una vez que la máquina virtual está registrada con la extensión Agente de IaaS de SQL, los cambios en el modelo de licencias estarán disponibles en la máquina virtual.

1. **¿Puedo usar Azure Portal para administrar varias instancias en la misma máquina virtual?**

   No. La administración desde el portal es una característica proporcionada por la extensión Agente de IaaS de SQL, que se basa en la extensión Agente de IaaS de SQL Server. Como tal, se aplican las mismas limitaciones a esta extensión que a la otra. El portal solo puede administrar una instancia predeterminada o una instancia con nombre, siempre y cuando se haya configurado correctamente. Para más información sobre estas limitaciones, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). 

1. **¿Las suscripciones de CSP pueden activar la Ventaja híbrida de Azure?**

   Sí, Ventaja híbrida de Azure está disponible para suscripciones de CSP. En primer lugar, los clientes de CSP deben implementar una imagen de pago por uso y, después, [cambiar el modelo de licencias](licensing-model-azure-hybrid-benefit-ahb-change.md) a traiga su propia licencia.
   
 
1. **¿Tengo que pagar para obtener una licencia de SQL Server en una máquina virtual de Azure si solo se usa para conmutación por error o en modo en espera?**

   Para tener una licencia pasiva gratuita para un grupo de disponibilidad secundario en espera o una instancia de clúster de conmutación por error, debe cumplir todos los criterios siguientes, tal como se describe en los [Términos de licencia del producto](https://www.microsoft.com/licensing/product-licensing/products):

   1. Tiene [movilidad de licencias](https://www.microsoft.com/licensing/licensing-programs/software-assurance-license-mobility?activetab=software-assurance-license-mobility-pivot:primaryr2) a través de [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?activetab=software-assurance-default-pivot%3aprimaryr3). 
   1. La instancia de SQL Server pasiva no proporciona datos de SQL Server a los clientes ni ejecuta cargas de trabajo de SQL Server activas. Solo se utiliza para sincronizar con el servidor principal y mantener la base de datos pasiva en un estado de espera semiactiva. Si está proporcionando datos, como informes a clientes que ejecutan cargas de trabajo de SQL Server activas, o si realiza cualquier trabajo, que no se especifique en los términos del producto, debe ser una instancia de SQL Server con licencia de pago. La siguiente actividad se permite en la instancia secundaria: comprobaciones de coherencia de base de datos o CHECKDB, copias de seguridad completas, copias de seguridad de registros de transacciones y supervisión de datos de uso de recursos. También puede ejecutar la instancia principal y la instancia de recuperación ante desastres correspondiente de forma simultánea durante breves períodos de pruebas de recuperación ante desastres cada 90 días. 
   1. La licencia activa de SQL Server está incluida en Software Assurance y permite **una** instancia de SQL Server secundaria pasiva, con hasta la misma cantidad de proceso que el servidor activo con licencia únicamente. 
   1. La VM de SQL Server secundaria emplea la licencia de [recuperación ante desastres](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) en Azure Portal.
   
1. **¿Qué se considera una instancia pasiva?**

   La instancia de SQL Server pasiva no proporciona datos de SQL Server a los clientes ni ejecuta cargas de trabajo de SQL Server activas. Solo se utiliza para sincronizar con el servidor principal y mantener la base de datos pasiva en un estado de espera semiactiva. Si está proporcionando datos, como informes a clientes que ejecutan cargas de trabajo de SQL Server activas, o si realiza cualquier trabajo, que no se especifique en los términos del producto, debe ser una instancia de SQL Server con licencia de pago. La siguiente actividad se permite en la instancia secundaria: comprobaciones de coherencia de base de datos o CHECKDB, copias de seguridad completas, copias de seguridad de registros de transacciones y supervisión de datos de uso de recursos. También puede ejecutar la instancia principal y la instancia de recuperación ante desastres correspondiente de forma simultánea durante breves períodos de pruebas de recuperación ante desastres cada 90 días.
   

1. **¿Qué escenarios pueden aprovechar la ventaja de recuperación ante desastres (DR)?**

   En la [guía de licencias](https://aka.ms/sql2019licenseguide) se proporcionan escenarios en los que se puede aprovechar la ventaja de recuperación ante desastres. Consulte los términos del producto y póngase en contacto con sus contactos de licencias o con el administrador de cuentas para más información.

1. **¿Qué suscripciones admiten la ventaja de recuperación ante desastres (DR)?**

   Los programas completos que ofrecen derechos de suscripción equivalentes de Software Assurance como una ventaja fija admiten la ventaja de la recuperación ante desastres. Esto incluye, entre otros, Open Value (OV), suscripción de Open Value (OVS), Contrato Enterprise (EA), suscripción del Contrato Enterprise (EAS) y Server and Cloud Enrollment (SCE). Consulte los [términos del producto](https://www.microsoft.com/licensing/product-licensing/products) y hable con los contactos de licencias o con el administrador de cuenta para más información. 

   
 ## <a name="extension"></a>Comprobación de actualización

1. **¿El registro de mi VM con la nueva extensión Agente de IaaS de SQL implicará costos adicionales?**

   No. La extensión Agente de IaaS de SQL simplemente hace posible una capacidad de administración adicional para SQL Server en la VM de Azure sin cargos adicionales. 

1. **¿Está disponible la extensión Agente de IaaS de SQL para todos los clientes?**
 
   Sí, siempre que se haya implementado la máquina virtual con SQL Server en la nube pública con el modelo de Resource Manager y no con el modelo clásico. Todos los demás clientes pueden registrarse con la nueva extensión Agente de IaaS de SQL. Sin embargo, solo los clientes con la ventaja de [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?activetab=software-assurance-default-pivot%3aprimaryr3) pueden usar su propia licencia activando la [Ventaja híbrida de Azure (AHB)](https://azure.microsoft.com/pricing/hybrid-benefit/) en una máquina virtual con SQL Server. 

1. **¿Qué ocurre con el recurso ( _Microsoft.SqlVirtualMachine_) de la extensión si se mueve o se quita el recurso de la máquina virtual?** 

   Cuando el recurso Microsoft.Compute/VirtualMachine se coloca o se mueve, se envía una notificación al recurso Microsoft.SqlVirtualMachine asociado para replicar de forma asincrónica la operación.

1. **¿Qué ocurre con la máquina virtual si el recurso (_Microsoft.SqlVirtualMachine_) de la extensión se elimina?**

    El recurso Microsoft.Compute/VirtualMachine no se ve afectado cuando se coloca el recurso Microsoft.SqlVirtualMachine. Sin embargo, los cambios en las licencias devolverán el valor predeterminado al origen de la imagen original. 

1. **¿Es posible registrar VM de SQL Server implementadas automáticamente con la extensión Agente de IaaS de SQL?**

    Sí. Si ha implementado SQL Server desde sus propios medios e instalado la extensión IaaS de SQL, puede registrar la VM con SQL Server con la extensión para obtener las ventajas de capacidad de administración proporcionadas por la extensión IaaS de SQL.    


## <a name="administration"></a>Administración

1. **¿Puedo instalar una segunda instancia de SQL Server en la misma máquina virtual? ¿Puedo cambiar las características instaladas de la instancia predeterminada?**

   Sí. Los medios de instalación de SQL Server están ubicados en una carpeta en la unidad **C** . Ejecute **Setup.exe** desde esa ubicación para agregar instancias de SQL Server nuevas o para cambiar otras características instaladas de SQL Server en la máquina. Tenga en cuenta que algunas características, como Copia de seguridad automatizada, Aplicación de revisión automatizada e integración de Azure Key Vault, solo funcionan con la instancia predeterminada o con una instancia con nombre que se haya configurado correctamente (consulte la pregunta 3). Los clientes que usan el modelo de licencias de [Software Assurance a través de la Ventaja híbrida de Azure](licensing-model-azure-hybrid-benefit-ahb-change.md) o de **pago por uso** pueden instalar varias instancias de SQL Server en la máquina virtual sin incurrir en costos adicionales de licencias. Las instancias de SQL Server adicionales pueden agotar los recursos del sistema a menos que se configuren correctamente. 

1. **¿Cuál es el número máximo de instancias de una máquina virtual?**
   Las versiones de SQL Server 2012 a SQL Server 2019 pueden admitir [50](/sql/sql-server/editions-and-components-of-sql-server-version-15#RDBMSSP) instancias en un servidor independiente. Este es el mismo límite, independientemente de que sea en el entorno local de Azure. Consulte las [prácticas recomendadas](performance-guidelines-best-practices.md#multiple-instances) para aprender a preparar mejor su entorno. 

1. **¿Puedo desinstalar la instancia predeterminada de SQL Server?**

   Sí, pero debe tener en cuenta algunas consideraciones. En primer lugar, es posible que sigan aplicándose cargos por facturación asociada con SQL Server en función del modelo de licencia de la máquina virtual. En segundo lugar, tal como se ha mencionado en la respuesta anterior, hay características que dependen de la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). Si desinstala la instancia predeterminada sin haber eliminado la extensión IaaS, la extensión continuará buscando la instancia predeterminada y puede generar errores de registro de eventos. Estos errores proceden de los dos orígenes siguientes: **Administración de credenciales de Microsoft SQL Server** y **Agente de IaaS de Microsoft SQL Server**. Es posible que uno de los errores sea similar al siguiente:

      Se ha producido un error relacionado con la red o específico de la instancia al establecer una conexión en SQL Server. No se encontró el servidor o no era accesible.

   Si decide desinstalar la instancia predeterminada, desinstale también la [Extensión del Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). 

1. **¿Puedo usar una instancia con nombre de SQL Server con la extensión IaaS?**
   
   Sí, en caso de que la instancia con nombre sea la única instancia de SQL Server y que la instancia predeterminada original se haya [desinstalado correctamente](sql-server-iaas-agent-extension-automate-management.md#named-instance-support). Si no hay ninguna instancia predeterminada y hay varias instancias con nombre en una única máquina virtual con SQL Server, no se podrá instalar la extensión Agente de IaaS de SQL Server.  

1. **¿Puedo quitar SQL Server y la facturación de licencias asociada de una VM con SQL Server?**

   Sí, pero deberá realizar pasos adicionales para evitar que se le cobre por su instancia de SQL Server, como se describe en la [Guía de precios](pricing-guidance.md). Si quiere quitar por completo la instancia de SQL Server, puede migrar a otra máquina virtual de Azure sin SQL Server preinstalado y eliminar la VM con SQL Server actual. Si quiere conservar la máquina virtual, pero detener la facturación de SQL Server, siga estos pasos: 

   1. Haga una copia de seguridad de todos los datos, incluidas las bases de datos del sistema, si es necesario. 
   1. Desinstale SQL Server completamente, incluida la extensión IaaS de SQL (si existe).
   1. Instale la [edición de SQL Express](https://www.microsoft.com/sql-server/sql-server-downloads) gratuita.
   1. Regístrese con la extensión Agente de IaaS de SQL en [modo ligero](sql-agent-extension-manually-register-single-vm.md).
   1. [Cambie la edición de SQL Server](change-sql-server-edition.md#change-edition-in-portal) en [Azure Portal](https://portal.azure.com) a Express para detener la facturación.  
   1. Deshabilite el servicio SQL Server Express deshabilitando el inicio del servicio (este paso es opcional). 

1. **¿Puedo usar Azure Portal para administrar varias instancias en la misma máquina virtual?**

   No. La administración desde el portal se proporciona por la extensión Agente de IaaS de SQL, que se basa en la extensión Agente de IaaS de SQL Server. Como tal, se aplican las mismas limitaciones a esta extensión que a la otra. El portal solo puede administrar una instancia predeterminada o una instancia con nombre, siempre y cuando se haya configurado correctamente. Para obtener más información, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). 


## <a name="updating-and-patching"></a>Actualizaciones y revisiones

1. **¿Cómo cambio a una versión o edición de SQL Server diferente en una VM de Azure?**

   Para cambiar su versión o edición de SQL Server, los clientes pueden utilizar los medios de instalación que contengan la versión o edición de SQL Server que deseen. Una vez que se ha cambiado la edición, utilice Azure Portal para modificar la propiedad edition de la máquina virtual para que refleje con precisión la facturación de la máquina virtual. Para más información, consulte cómo [cambiar la edición de una máquina virtual de SQL Server](change-sql-server-edition.md). No hay ninguna diferencia de facturación para las distintas versiones de SQL Server, por lo que una vez que se ha cambiado la versión de SQL Server, no es necesario realizar ninguna otra acción.

1. **¿Dónde puedo obtener el soporte de instalación para cambiar la edición o la versión de SQL Server?**

   Los clientes que tengan [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) pueden obtener el soporte de instalación en el [centro de licencias por volumen](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Los clientes que no tengan Software Assurance pueden usar el soporte de instalación de una imagen de VM con SQL Server de Marketplace que tenga la edición que deseen.
   
1. **¿Cómo se aplican las actualizaciones y los Service Packs en una máquina virtual de SQL Server?**

   Las máquinas virtuales brindan control sobre la máquina virtual, incluido el momento en que se aplican las actualizaciones y la manera de hacerlas. En el caso del sistema operativo, puede aplicar manualmente las actualizaciones de Windows, o bien puede habilitar un servicio de programación llamado [Aplicación de revisiones automatizada](automated-patching.md). Aplicación de revisión automatizada instala las actualizaciones marcadas como importantes, incluidas las actualizaciones de SQL Server que tienen esa categoría. Otras actualizaciones opcionales a SQL Server se deben instalar manualmente.

1. **¿Puedo actualizar mi instancia de SQL Server 2008/2008 R2 después de registrarla con la extensión Agente de IaaS de SQL?**

   Si el sistema operativo es Windows Server 2008 R2 o una versión posterior, sí. Puede usar cualquier soporte de instalación para actualizar la versión y edición de SQL Server y, a continuación, puede actualizar el [modo de la extensión IaaS de SQL](sql-server-iaas-agent-extension-automate-management.md#management-modes)) de _sin agente_ a _completo_. Esto le dará acceso a todas las ventajas de la extensión IaaS de SQL, como la capacidad de administración de portal, las copias de seguridad automáticas y la aplicación de revisiones automatizada. Si la versión del sistema operativo es Windows Server 2008, solo se admite el modo NoAgent. 

1. **¿Cómo puedo obtener actualizaciones de seguridad ampliadas gratuitas para el fin del soporte de mis instancias de SQL Server 2008 y SQL Server 2008 R2?**

   Para obtener [actualizaciones de seguridad ampliadas gratuitas](sql-server-2008-extend-end-of-support.md), mueva su instancia de SQL Server tal cual a una máquina virtual de Azure. Para obtener más información, vea las [opciones de fin del soporte](/sql/sql-server/end-of-support/sql-server-end-of-life-overview). 
  
   

## <a name="general"></a>General

1. **¿Se admiten las instancias del clúster de conmutación por error (FCI) de SQL Server con máquinas virtuales de Azure?**

   Sí. Puede instalar una instancia del clúster de conmutación por error mediante [recursos compartidos de archivos Premium (PFS)](failover-cluster-instance-premium-file-share-manually-configure.md) o de [espacios de almacenamiento directo (S2D)](failover-cluster-instance-storage-spaces-direct-manually-configure.md) para el subsistema de almacenamiento. Los recursos compartidos de archivos Premium proporcionan funcionalidades de IOPS y rendimiento para responder a las necesidades de varias cargas de trabajo. Sin embargo, en el caso de cargas de trabajo con un uso intensivo de E/S, considere el uso de espacios de almacenamiento directo basado en discos o ultradiscos Premium administrados. También puede usar soluciones de agrupación en clústeres o almacenamiento de terceros como se describe en [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](business-continuity-high-availability-disaster-recovery-hadr-overview.md#azure-only-high-availability-solutions).

   > [!IMPORTANT]
   > En este momento, la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md) _completa_ no es compatible con FCI de SQL Server en Azure. Recomendamos que desinstale la extensión _completa_ de las máquinas virtuales que participan en la FCI y que, en su lugar, instale la extensión en modo _ligero_. Esta extensión admite características, como las copias de seguridad automáticas y la aplicación de revisiones automatizada y algunas características del portal de SQL Server. Estas características no funcionarán para las máquinas virtuales con SQL Server después de desinstalar el agente _completo_.

1. **¿Cuál es la diferencia entre las máquinas virtuales con SQL Server y el servicio SQL Database?**

   De manera conceptual, ejecutar SQL Server en una máquina virtual de Azure no es diferente a ejecutar SQL Server en un centro de datos remoto. En cambio, [Azure SQL Database](../../database/sql-database-paas-overview.md) ofrece base de datos como servicio. Con SQL Database, no se tiene acceso a las máquinas que hospedan las bases de datos. Para una comparación completa, consulte [Choose a cloud SQL Server option: Azure SQL (PaaS) Database or SQL Server on Azure VMs (IaaS)](../../azure-sql-iaas-vs-paas-what-is-overview.md) (Selección de una opción Azure SQL en la nube: Azure SQL (PaaS) Database o SQL Server en máquinas virtuales de Azure (IaaS).

1. **¿Cómo instalo herramientas de datos de SQL en mi máquina virtual de Azure?**

    Descargue e instale las herramientas de datos de SQL desde [Microsoft SQL Server Data Tools - Business Intelligence para Visual Studio 2013](https://www.microsoft.com/download/details.aspx?id=42313).

1. **¿Las transacciones distribuidas con MSDTC se admiten en las VM con SQL Server?**
   
    Sí. El DTC local es compatible con SQL Server 2016 SP2 y versiones posteriores. Sin embargo, las aplicaciones deben probarse cuando se utilicen los grupos de disponibilidad Always On, ya que las transacciones en proceso durante una conmutación por error producirán un error y se deberán intentar de nuevo. El DTC en clúster está disponible a partir de Windows Server 2019. 
    
1. **¿La máquina virtual de Azure SQL mueve o almacena datos de clientes fuera de la región?**

   No. De hecho, la máquina virtual de Azure SQL y la extensión del agente de IaaS de SQL no almacenan datos de clientes.

## <a name="sql-server-iaas-agent-extension"></a>Extensión Agente de IaaS de SQL Server

1. **¿Debo registrar mi máquina virtual con SQL Server aprovisionada a partir de una imagen de SQL Server en Azure Marketplace?**

   No. Microsoft registra automáticamente las máquinas virtuales aprovisionadas a partir de imágenes de SQL Server en Azure Marketplace. El registro con la extensión solo es necesario si la máquina virtual *no* se aprovisionó a partir de las imágenes de SQL Server en Azure Marketplace y SQL Server se instaló automáticamente.

1. **¿Está disponible la extensión del Agente de IaaS de SQL para todos los clientes?** 

   Sí. Los clientes deben registrar sus máquinas virtuales con SQL Server con la extensión si no usaron una imagen de SQL Server de Azure Marketplace y, en su lugar, se instaló automáticamente SQL Server o si usan su propio VHD personalizado. Las máquinas virtuales que poseen todo tipo de suscripciones (directa, Contrato Enterprise y proveedor de soluciones en la nube) se pueden registrar con la extensión del Agente de IaaS de SQL.

1. **¿Cuál es el modo de administración predeterminado al registrarse con la extensión del Agente de IaaS de SQL?**

   El modo de administración predeterminado al registrarse con la extensión del Agente de IaaS de SQL es *ligero*. Si no se establece la propiedad de administración de SQL Server al registrarse con la extensión, se establecerá el modo de administración ligero y el servicio SQL Server no se reiniciará. Se recomienda registrar la máquina virtual con la extensión del Agente de IaaS de SQL primero en modo ligero y, posteriormente, actualizar al modo completo durante una ventana de mantenimiento. Del mismo modo, la administración predeterminada también es ligera cuando se usa la [característica de registro automático](sql-agent-extension-automatic-registration-all-vms.md).

1. **¿Cuáles son los requisitos previos para registrarse con la extensión del Agente de IaaS de SQL?**

   No existe ningún requisito previo para registrarse con la extensión del Agente de IaaS de SQL aparte de tener instalado SQL Server en la máquina virtual. Tenga en cuenta que si la extensión del Agente de IaaS de SQL se instala en modo completo, el servicio SQL Server se reiniciará, por lo que se recomienda hacerlo durante una ventana de mantenimiento.

1. **¿Al registrarse con la extensión del Agente de IaaS de SQL se instala un agente en mi máquina virtual?**

   Sí, al registrarse con la extensión del Agente de IaaS de SQL en modo de administración completa se instala un agente en la máquina virtual. No si se realiza el registro en modo ligero o NoAgent. 

   Al registrarse con la extensión del Agente de IaaS de SQL en modo ligero, solo se copian los *archivos binarios* de la extensión en la máquina virtual, no se instala el agente. Estos archivos binarios se utilizan posteriormente para instalar el agente cuando el modo de administración se actualiza a completo.


1. **¿Al registrarse con la extensión del Agente de IaaS de SQL se reinicia SQL Server en la máquina virtual?**

   Depende del modo especificado durante el registro. Si se especifica el modo ligero o NoAgent, el servicio SQL Server no se reiniciará. Sin embargo, si se especifica el modo de administración como completo, el servicio SQL Server se reiniciará. La característica de registro automático registra las VM con SQL Server en modo ligero, a menos que la versión de Windows Server sea 2008, en cuyo caso la VM con SQL Server se registrará en el modo NoAgent. 

1. **¿Cuál es la diferencia entre los modos de administración ligero y NoAgent al registrarse con la extensión del Agente de IaaS de SQL?** 

   El modo de administración NoAgent es el único modo de administración disponible para SQL Server 2008 y SQL Server 2008 R2 en Windows Server 2008. En todas las versiones posteriores de Windows Server, los dos modos de administración disponibles son ligero y completo. 

   El modo NoAgent requiere que el cliente establezca las propiedades de la versión y edición de SQL Server. El modo ligero consulta a la máquina virtual para encontrar la versión y la edición de la instancia de SQL Server.

1. **¿Puedo registrarme con la extensión del Agente de IaaS de SQL sin especificar el tipo de licencia de SQL Server?**

   No. El tipo de licencia de SQL Server no es una propiedad opcional cuando se registra con la extensión del Agente de IaaS de SQL. Al registrarse con la extensión del Agente de IaaS de SQL en todos los modos de administración (NoAgent, ligero y completo), tiene que establecer el tipo de licencia de SQL Server como pago por uso o Ventaja híbrida de Azure. Si tiene instalada alguna de las versiones gratuitas de SQL Server, como Developer o Evaluation Edition, debe registrarse con las licencias de pago por uso. Ventaja híbrida de Azure solo está disponible para las versiones de pago de SQL Server, como Enterprise y Standard Edition.

1. **¿Se puede actualizar la extensión de IaaS de SQL Server del modo NoAgent al modo completo?**

   No. La actualización del modo de administración a completo o ligero no está disponible para el modo NoAgent. Se trata de una limitación técnica de Windows Server 2008. Tendrá que actualizar primero el sistema operativo a Windows Server 2008 R2 o posterior y, a continuación, podrá actualizar al modo de administración completa. 

1. **¿Se puede actualizar la extensión IaaS de SQL Server del modo ligero al modo completo?**

   Sí. La actualización del modo de administración de ligero a completo se admite a través de Azure PowerShell o Azure Portal. Esto desencadenará un reinicio del servicio SQL Server.

1. **¿Se puede cambiar la extensión de IaaS de SQL Server del modo completo a los modos inferiores de administración NoAgent o ligero?**

   No. No se admite cambiar a una versión inferior del modo de administración de la extensión IaaS de SQL Server. El modo de administración no se puede cambiar a una versión inferior, del modo completo al modo ligero o NoAgent, ni del modo ligero al modo NoAgent. 

   Para cambiar el modo de administración de completo a otro, [anule el registro](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) de la VM con SQL Server de la extensión del Agente de IaaS de SQL. Para ello, quite el _recurso_ de la máquina virtual con SQL y vuelva a registrarla con esta extensión en otro modo de administración.

1. **¿Puedo registrarme con la extensión del Agente de IaaS de SQL desde Azure Portal?**

   No. El registro con la extensión del Agente de IaaS de SQL no está disponible en Azure Portal. El registro con la extensión del Agente de IaaS de SQL solo se admite con la CLI de Azure o Azure PowerShell. 

1. **¿Puedo registrar una máquina virtual con la extensión del Agente de IaaS de SQL antes de instalar SQL Server?**

   No. Una máquina virtual debe tener al menos una instancia de SQL Server (Motor de base de datos) para registrarse correctamente con la extensión del Agente de IaaS de SQL. Si no hay ninguna instancia de SQL Server en la máquina virtual, el nuevo recurso Microsoft.SqlVirtualMachine tendrá un estado de error.

1. **¿Puedo registrar una máquina virtual con la extensión del Agente de IaaS de SQL si hay varias instancias de SQL Server?**

   Sí, siempre que haya una instancia predeterminada en la VM. La extensión del Agente de IaaS de SQL solo registrará una instancia de SQL Server (Motor de base de datos). En caso de varias instancias, registrará la instancia de SQL Server predeterminada.

1. **¿Se puede registrar una instancia de clúster de conmutación por error de SQL Server con la extensión del Agente de IaaS de SQL?**

   Sí. Las instancias de clúster de conmutación por error de SQL Server en una máquina virtual de Azure se pueden registrar con la extensión del Agente de IaaS de SQL en modo ligero. Sin embargo, las instancias de clúster de conmutación por error de SQL Server no se pueden actualizar al modo de administración completa.

1. **¿Puedo registrar mi máquina virtual con la extensión del Agente de IaaS de SQL si hay configurado un grupo de disponibilidad AlwaysOn?**

   Sí. No hay ninguna restricción por registrar una instancia de SQL Server de una máquina virtual de Azure con la extensión del Agente de IaaS de SQL si participa en una configuración de grupo de disponibilidad AlwaysOn.

1. **¿Cuál es el costo por registrarse con la extensión del Agente de IaaS de SQL o por actualizar al modo de administración completa?**

   Ninguno. No hay ninguna tarifa asociada al registro con la extensión del Agente de IaaS de SQL ni al uso de ninguno de los tres modos de administración. La administración de la VM con SQL Server con la extensión es completamente gratuita. 

1. **¿Como afecta el uso de los distintos modos de administración al rendimiento?**

   El uso de los modos de manejabilidad *NoAgent* y *ligero* no tiene ningún impacto en el rendimiento. Si se usa el modo de manejabilidad *completa* desde dos servicios instalados en el sistema operativo, el impacto es mínimo. Estos se pueden supervisar mediante el administrador de tareas y se muestran en la consola de Servicios de Windows integrada. 

   Los dos nombres de servicio son:
   - `SqlIaaSExtensionQuery` (nombre para mostrar - `Microsoft SQL Server IaaS Query Service`)
   - `SQLIaaSExtension` (nombre para mostrar - `Microsoft SQL Server IaaS Agent`)

1. **¿Cómo se quita la extensión?**

   Para quitar la extensión, [anule el registro](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) de la VM con SQL Server de ella. 

## <a name="resources"></a>Recursos

**Máquinas virtuales Windows**:

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Aprovisionamiento de SQL Server en una VM Windows](create-sql-vm-portal.md)
* [Migración de una base de datos a SQL Server en una máquina virtual de Azure](migrate-to-vm-from-sql-server.md)
* [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](business-continuity-high-availability-disaster-recovery-hadr-overview.md)
* [Directrices de rendimiento para SQL Server en Azure Virtual Machines](performance-guidelines-best-practices.md)
* [Estrategias de desarrollo y patrones de aplicación de SQL Server en Azure Virtual Machines](application-patterns-development-strategies.md)

**Máquinas virtuales Linux**:

* [Introducción a SQL Server en máquinas virtuales Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
* [Aprovisionamiento de SQL Server en una VM Linux](../linux/sql-vm-create-portal-quickstart.md)
* [Preguntas más frecuentes (Linux)](../linux/frequently-asked-questions-faq.md)
* [Documentación de SQL Server en Linux](/sql/linux/sql-server-linux-overview)
