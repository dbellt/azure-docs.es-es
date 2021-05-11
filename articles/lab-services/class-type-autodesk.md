---
title: Configuración de un laboratorio con Autodesk mediante Azure Lab Services
description: Aprenda a configurar laboratorios para enseñar clases de ingeniería con Autodesk.
author: nicolela
ms.topic: article
ms.date: 04/21/2021
ms.author: nicolela
ms.openlocfilehash: 6cc983907349ca8eab0731b18c18d526f2b75ba5
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108281120"
---
# <a name="set-up-labs-for-autodesk"></a>Configuración de laboratorios para Autodesk

En este artículo se describe cómo configurar el software Autodesk Inventor y Autodesk Revit para clases de ingeniería:
- El [diseño asistido por PC (CAD)](https://www.autodesk.com/products/inventor/new-features) y la [fabricación asistida por PC (CAM) Inventor](https://www.autodesk.com/products/inventor-cam/overview) proporcionan modelado 3D y se usan en el diseño de ingeniería.
- [Revit](https://www.autodesk.com/products/revit/overview) se usa en el diseño de arquitectura para el modelado de información de construcción (BIM) en 3D.

Autodesk se usa habitualmente tanto en universidades como en educación primaria y secundaria.  Por ejemplo, en primaria y secundaria, AutoDesk se incluye en el plan de estudios [Project Lead the Way (PLTW)](./class-type-pltw.md).

## <a name="lab-configuration"></a>Configuración del laboratorio

Para configurar este laboratorio, para empezar, necesita una suscripción a Azure y una cuenta de laboratorio. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Una vez que obtenga una suscripción a Azure, puede crear una nueva cuenta de laboratorio en Azure Lab Services. Para más información sobre la creación de una nueva cuenta de laboratorio, consulte el tutorial sobre la [configuración de una cuenta de laboratorio](./tutorial-setup-lab-account.md). También puede usar una cuenta de laboratorio existente.

### <a name="lab-account-settings"></a>Configuración de la cuenta de laboratorio

Habilite la configuración de la cuenta de laboratorio tal y como se describe en la tabla siguiente. Para obtener más información sobre cómo habilitar imágenes de Azure Marketplace, consulte [Especificación de las imágenes de Azure Marketplace disponibles para los creadores de laboratorios](./specify-marketplace-images.md).

| Configuración de la cuenta de laboratorio | Instructions |
| -------------------- | ----- |
| Imagen de Marketplace | Habilite la imagen de Windows 10 para usarla en la cuenta de laboratorio. |

### <a name="lab-settings"></a>Configuración del laboratorio
El tamaño de la máquina virtual (VM) que se recomienda depende de los tipos de cargas de trabajo de las que los alumnos tienen que encargarse.  Se recomienda usar el tamaño de GPU pequeña (visualización).

| Configuración del laboratorio | Valor y descripción |
| ------------ | ------------------ |
| Tamaño de la máquina virtual | **GPU pequeña (visualización)**<br>Es el más adecuado para la visualización, la realización de streaming, los juegos y la codificación de forma remota que usan marcos como OpenGL y DirectX. | 

> [!NOTE]
> El tamaño de máquina virtual de **GPU pequeña (visualización)** está configurado para permitir una experiencia de gráficos de alto rendimiento. Para obtener más información sobre este tamaño de máquina virtual, consulte el artículo sobre la [configuración de un laboratorio con GPU](./how-to-setup-lab-gpu.md).

### <a name="license-server"></a>Servidor de licencias
Tendrá que acceder a un servidor de licencias si tiene previsto usar el modelo de licencias de red de Autodesk.  Lea el artículo de Autodesk sobre [administración de licencias de red](https://knowledge.autodesk.com/customer-service/network-license-administration/network-deployment/preparing-for-deployment/determining-installation-type) para obtener más información.

Para usar las licencias de red con el software de Autodesk, [AutoDesk proporciona pasos detallados](https://knowledge.autodesk.com/customer-service/network-license-administration/install-and-configure-network-license) para instalar el administrador de licencias de Autodesk en el servidor de licencias.  Este servidor de licencias se encuentra normalmente en la red local u hospedado en una máquina virtual de Azure en Azure Virtual Network.

Una vez configurado el servidor de licencias, tendrá que emparejar la [red virtual](./how-to-connect-peer-virtual-network.md) con la [cuenta de laboratorio](./tutorial-setup-lab-account.md). *Antes* debe crear el laboratorio para que las máquinas virtuales de laboratorio puedan acceder al servidor de licencias y viceversa.

Los archivos de licencia generados por Autodesk insertan la dirección MAC del servidor de licencias.  Si decide hospedar el servidor de licencias mediante una VM de Azure, es importante que se asegure de que la dirección MAC del servidor de licencias no cambie. Si la dirección MAC cambia, deberá volver a generar los archivos de licencia. Siga estas sugerencias para evitar que la dirección MAC cambie:

- [Establezca una dirección IP privada estática y una dirección MAC](./how-to-create-a-lab-with-shared-resource.md#static-private-ip-and-mac-address) para la VM de Azure que hospeda el servidor de licencias.
- Asegúrese de configurar la red virtual del servidor de licencias y la cuenta del laboratorio en una región o ubicación que tenga suficiente capacidad de VM para no tener que trasladar estos recursos a otra región o ubicación más adelante.

Para obtener más información, vea [Configuración de un servidor de licencias como recurso compartido](./how-to-create-a-lab-with-shared-resource.md).

> [!WARNING]
> No olvide [emparejar la red virtual](./how-to-connect-peer-virtual-network.md) de la cuenta de laboratorio con la red virtual del servidor de licencias **antes** de crear el laboratorio.

### <a name="template-machine"></a>Máquina de plantilla
Los pasos de esta sección muestran cómo configurar la VM de plantilla:

1. Inicie la VM de plantilla y conéctese a la máquina.

1. Descargue e instale Inventor y Inventor siguiendo las [instrucciones de AutoDesk](https://knowledge.autodesk.com/customer-service/download-install/install-software).  Cuando se le solicite, especifique el nombre de equipo del servidor de licencias.

1.  Por último, publique la máquina de plantilla para crear las VM de los alumnos.

## <a name="cost"></a>Coste
Veamos un ejemplo de una posible estimación de costos para esta clase.  Esta estimación no incluye el costo de ejecutar un servidor de licencias. Supongamos que tiene una clase de 25 alumnos, cada uno de los cuales tiene 20 horas de clase programadas.  Cada alumno tiene también 10 horas de cuota adicionales para deberes o asignaciones fuera de las horas de clase programadas.  El tamaño de la máquina virtual que hemos elegido es el de **GPU pequeña (visualización)** , que tiene 160 unidades de laboratorio.

- 25 alumnos &times; (20 horas programadas + 10 horas de cuota) &times; 160 unidades de laboratorio &times; 0,01 USD por hora = 1200,00 USD

> [!IMPORTANT] 
> La estimación de costos solo se utiliza con fines de ejemplo.  Para obtener información sobre los precios actuales, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="next-steps"></a>Pasos siguientes

Cuando configure el laboratorio, consulte los siguientes artículos:

- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de cuotas](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users) 
