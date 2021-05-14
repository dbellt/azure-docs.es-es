---
title: Habilitación de la virtualización anidada en una plantilla de máquina virtual en Azure Lab Services | Microsoft Docs
description: En este artículo aprenderá cómo configurar la virtualización anidada en una máquina de plantilla en Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: f1f981602adcf90762258a2921b996507f56f3a2
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125578"
---
# <a name="enable-nested-virtualization-on-a-template-virtual-machine-in-azure-lab-services"></a>Habilitación de la virtualización anidada en una plantilla de máquina virtual en Azure Lab Services

Actualmente, Azure Lab Services le permite configurar una máquina virtual de una plantilla en un laboratorio y realizar una copia única disponible para cada uno de los usuarios. Si usted es un formador que hace clases de redes, seguridad o TI, puede que necesite proporcionar a cada uno de sus alumnos un entorno en el que varias máquinas virtuales puedan comunicarse entre sí a través de una red.

La virtualización anidada le permite crear un entorno de varias máquinas virtuales dentro de una plantilla de máquina virtual de un laboratorio. La publicación de la plantilla proporcionará a cada usuario del laboratorio una máquina virtual configurada con varias máquinas virtuales dentro.  En este artículo se explica cómo configurar la virtualización anidada en una máquina de pantilla en Azure Lab Services.

## <a name="what-is-nested-virtualization"></a>¿Qué es la virtualización anidada?

La virtualización anidada le permite crear máquinas virtuales dentro de una máquina virtual. La virtualización anidada se realiza a través de Hyper-V y solo está disponible en las máquinas virtuales de Windows.

Para más información sobre la virtualización anidada, vea los siguientes artículos:

- [Virtualización anidada en Azure](https://azure.microsoft.com/blog/nested-virtualization-in-azure/)
- [Habilitación de la virtualización anidada en una máquina virtual de Azure](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)

## <a name="considerations"></a>Consideraciones

Antes de configurar un laboratorio con virtualización anidada, estas son algunas cosas que debe tener en cuenta.

- Al crear un laboratorio, seleccione los tamaños **Medio (virtualización anidada)** o **Grande (virtualización anidada)** como tamaño de la máquina virtual. Estos tamaños de máquina virtual admiten la virtualización anidada.
- Elija un tamaño que proporcione un buen rendimiento para las máquinas virtuales de host y de cliente.  Recuerde que, al usar la virtualización, el tamaño que elija debe ser adecuado no solo para una máquina, sino también para el host y las máquinas de Hyper-V que se ejecuten simultáneamente.
- Las máquinas virtuales de cliente no tendrán acceso a los recursos de Azure como, por ejemplo, a los servidores DNS de la red virtual de Azure.
- La máquina virtual del host requiere configuración para permitir que la máquina cliente tenga conectividad a Internet.
- Las máquinas virtuales de cliente tienen licencia como máquinas independientes. Consulte [Licencias de Microsoft](https://www.microsoft.com/licensing/default) para más información sobre las licencias de los productos y sistemas operativos de Microsoft. Compruebe los contratos de licencia de cualquier otro software que se esté usando antes de configurar la plantilla de máquina virtual.

## <a name="enable-nested-virtualization-on-a-template-vm"></a>Habilitación de la virtualización anidada en una máquina virtual de plantilla

En este artículo se supone que ha creado una cuenta de laboratorio y un laboratorio.  Para más información sobre la creación de una nueva cuenta de laboratorio, consulte el [tutorial para configurar una cuenta de laboratorio](tutorial-setup-lab-account.md). Para más información sobre cómo crear un laboratorio, consulte el [tutorial Configuración de un laboratorio educativo](tutorial-setup-classroom-lab.md).

>[!IMPORTANT]
>Seleccione **Grande (virtualización anidada)** o **Medio (virtualización anidada)** como tamaño de la máquina virtual al crear el laboratorio.  De lo contrario, la virtualización anidada no funcionará.  

Para conectarse a la máquina de plantilla, consulte [Creación y administración de plantillas educativas en Azure Lab Services](how-to-create-manage-template.md).

Para habilitar la virtualización anidada, hay que realizar algunas tareas.  

- **Habilitar el rol de Hyper-V**. El rol de Hyper-V debe estar habilitado para la creación y ejecución de máquinas virtuales de Hyper-V en la máquina virtual de Lab Services.
- **Habilitar DHCP**.  Cuando la máquina virtual de Lab Services tiene el rol DHCP habilitado, se puede asignar automáticamente una dirección IP a las máquinas virtuales de Hyper-V.
- **Crear una red NAT para máquinas virtuales de Hyper-V**.  La red NAT está configurada para permitir que las máquinas virtuales de Hyper-V tengan acceso a Internet.  Las máquinas virtuales de Hyper-V pueden comunicarse entre sí.

>[!NOTE]
>La red NAT creada en la máquina virtual de Lab Services permitirá que una máquina virtual de Hyper-V tenga acceso a Internet y a otras máquinas virtuales de Hyper-V en la misma máquina virtual de Lab Services.  La máquina virtual de Hyper-V no podrá tener acceso a los recursos de Azure, como los servidores DNS, en la red virtual de Azure.

La realización de las tareas indicadas anteriormente puede realizarse mediante un script o con las herramientas de Windows.  Lea las secciones siguientes para obtener más detalles.

### <a name="using-script-to-enable-nested-virtualization"></a>Uso de un script para habilitar la virtualización anidada

Si quiere usar la configuración automatizada para la virtualización anidada con Windows Server 2016 o Windows Server 2019, consulte [Habilitación de la virtualización anidada en una máquina virtual de plantilla en Azure Lab Services mediante un script](how-to-enable-nested-virtualization-template-vm-using-script.md). Usará scripts de [Hyper-V de Lab Services](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/HyperV) para instalar el rol de Hyper-V.  Los scripts también configurarán las funciones de red para que las máquinas virtuales de Hyper-V puedan acceder a Internet.

### <a name="using-windows-tools-to-enable-nested-virtualization"></a>Uso de las herramientas de Windows para habilitar la virtualización anidada

Si quiere configurar la virtualización anidada para Windows Server 2016 o Windows Server 2019 con los roles y herramientas administrativas de Windows, consulte [Habilitación de la virtualización anidada en una máquina virtual de plantilla en Azure Lab Services de forma manual](how-to-enable-nested-virtualization-template-vm-ui.md).  Las instrucciones también explican cómo configurar las funciones de red para que las máquinas virtuales de Hyper-V puedan acceder a Internet.