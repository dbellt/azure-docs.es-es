---
title: Implementación de SAP S/4 HANA o BW/4 HANA en una máquina virtual de Azure | Microsoft Docs
description: Implementación de SAP S/4HANA o BW/4HANA en una máquina virtual de Azure
ms.service: virtual-machines-sap
author: hobru
manager: maordish
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/15/2016
ms.author: hobruche
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e9cbb7e03d77507e4671e2e10665bc245a095b0a
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755401"
---
# <a name="deploy-sap-s4hana-or-bw4hana-on-azure"></a>Implementación de SAP S/4HANA o BW/4HANA en Azure
En este artículo se describe cómo implementar S/4HANA en Azure por medio de SAP Cloud Appliance Library (SAP CAL) 3.0. Para implementar otras soluciones basadas en SAP HANA, por ejemplo, BW/4HANA, siga los mismos pasos.

> [!NOTE]
> Para más información sobre SAP CAL, vaya al sitio web de [SAP Cloud Appliance Library](https://cal.sap.com/). SAP también tiene un blog acerca de [SAP Cloud Appliance Library 3.0](https://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience).
> 
> [!NOTE]
> A partir del 29 de mayo de 2017 puede usar el modelo de implementación mediante Azure Resource Manager, además del modelo de implementación clásico, que se usa menos, para implementar SAP CAL. Se recomienda usar el nuevo modelo de implementación mediante Resource Manager, en lugar del modelo de implementación clásico.

## <a name="step-by-step-process-to-deploy-the-solution"></a>Proceso paso a paso para implementar la solución

La siguiente secuencia de capturas de pantalla muestra cómo implementar S/4HANA en Azure mediante la solución SAP CAL. El proceso funciona del mismo modo para otras soluciones como BW/4HANA.

La página **Soluciones** muestra algunas de las soluciones basadas en SAP CAL HANA disponibles en Azure. **SAP S/4HANA 1610 FPS01, aplicación completamente activada** está en la fila central:

![Soluciones de SAP CAL](./media/cal-s4h/s4h-pic-1c.png)

### <a name="create-an-account-in-the-sap-cal"></a>Creación de una cuenta en SAP CAL
1. La primera vez que inicie sesión en SAP CAL, utilice S-User de SAP, o cualquier otro usuario registrado de SAP. A continuación, defina una cuenta de SAP CAL que SAP CAL use para implementar aplicaciones en Azure. En la definición de la cuenta, debe:

    a. Seleccione el modelo de implementación en Azure (Resource Manager o clásico).

    b. Especifique la suscripción a Azure. Una cuenta de SAP CAL no se puede asignar a más de una suscripción. Si necesita más de una suscripción, debe crear otra cuenta de SAP CAL.

    c. Conceda el permiso de SAP CAL para realizar la implementación en la suscripción de Azure.

   > [!NOTE]
   >  Los pasos siguientes muestran cómo crear una cuenta de SAP CAL para las implementaciones mediante Resource Manager. Si ya tiene una cuenta de SAP CAL vinculada al modelo de implementación clásico, *tendrá que* seguir estos pasos para crear una cuenta de SAP CAL nueva. La nueva cuenta de SAP CAL se tiene que implementar mediante el modelo de Resource Manager.

1. Cree una nueva cuenta de SAP CAL. La página **Cuentas** muestra tres opciones de Azure: 

    a. **Microsoft Azure (classic)** es el modelo de implementación clásico y ya no se suele usar.

    b. **Microsoft Azure** es el nuevo modelo de implementación mediante Resource Manager.

    c. **Windows Azure operado por 21Vianet** es una opción en China que usa el modelo de implementación clásica.

    Para implementar mediante el modelo de Resource Manager, seleccione **Microsoft Azure**.

    ![Detalles de la cuenta de SAP CAL](./media/cal-s4h/s4h-pic-2a.png)

1. En el campo **Suscription ID**, especifique el identificador de suscripción de Azure, que se puede encontrar en Azure Portal.

   ![Cuentas de SAP CAL](./media/cal-s4h/s4h-pic3c.png)

1. Para autorizar que SAP CAL se implemente en la suscripción de Azure que ha definido, haga clic en **Authorize** (Autorizar). La siguiente página aparecerá en la pestaña del explorador:

   ![Inicio de sesión de servicios en la nube de Internet Explorer](./media/cal-s4h/s4h-pic4c.png)

1. Si se enumera más de un usuario, elija la cuenta de Microsoft vinculada para ser el coadministrador de la suscripción de Azure que ha seleccionado. La siguiente página aparecerá en la pestaña del explorador:

   ![Confirmación de servicios en la nube de Internet Explorer](./media/cal-s4h/s4h-pic5a.png)

1. Haga clic en **Aceptar**. Si la autorización es correcta, vuelve a mostrarse la definición de la cuenta de SAP CAL. Poco después, un mensaje confirma que el proceso de autorización se ha realizado correctamente.

1. Para asignar la cuenta de SAP CAL recién creada al usuario, escriba en el cuadro de texto de la derecha su **identificador de usuario** y haga clic en **Agregar**.

   ![Asociación de cuenta a usuario](./media/cal-s4h/s4h-pic8a.png)

1. Para asociar una cuenta con el usuario que utiliza para iniciar sesión en SAP CAL, haga clic en **Review** (Revisar). 
 
1. Para crear la asociación entre el usuario y la cuenta de SAP CAL recién creada, haga clic en **Create** (Crear).

   ![Asociación de usuario a cuenta de SAP CAL](./media/cal-s4h/s4h-pic9b.png)

Ha creado correctamente una cuenta de SAP CAL que puede:

- Usar el modelo de implementación mediante Resource Manager.
- Implementar sistemas de SAP en su suscripción a Azure.

Ya puede empezar a implementar S/4HANA en su suscripción de usuario en Azure.

> [!NOTE]
> Antes de continuar, determine si dispone de las cuotas de unidades vCPU de Azure para máquinas virtuales de la serie H de Azure. En este momento, SAP CAL usa máquinas virtuales de la serie H de Azure para implementar algunas de las soluciones basadas en SAP HANA. Es posible que su suscripción a Azure no tenga ninguna cuota de unidades vCPU para la serie H. Si es así, deberá ponerse en contacto con el soporte técnico de Azure para obtener una cuota de al menos 16 unidades vCPU de la serie H.
> 
> [!NOTE]
> Al implementar una solución de Azure en SAP CAL, es posible que solo pueda elegir una región de Azure. Para implementar en otras regiones de Azure distintas a la que sugiere SAP CAL, debe adquirir una suscripción de CAL desde SAP. Es posible que también necesite abrir un mensaje con SAP para habilitar su cuenta de CAL para entregar en regiones de Azure distintas de las que se sugirieron inicialmente.

### <a name="deploy-a-solution"></a>Implementación de una solución

Vamos a implementar una solución desde la página **Soluciones** de SAP CAL. SAP CAL tiene dos secuencias para implementar:

- Una secuencia básica que utiliza una página para definir el sistema que se va a implementar
- Una secuencia avanzada que le ofrece varias opciones en relación al tamaño de las máquinas virtuales 

Aquí se muestra la ruta de acceso básica a la implementación.

1. En la página **Detalles de la cuenta**, debe:

    a. Seleccionar una cuenta de SAP CAL. (Utilice una cuenta asociada para implementar con el modelo de implementación de Resource Manager).

    b. Especificar en **Name** el nombre de la instancia.

    c. Seleccionar una **región** de Azure. SAP CAL sugiere una región. Si necesita otra región de Azure y no tiene una suscripción de SAP CAL, deberá solicitar una suscripción de CAL con SAP.

    d. Escriba una **contraseña** maestra para la solución que contenga ocho o nueve caracteres. La contraseña se usa para los administradores de los distintos componentes.

   ![SAP CAL Basic Mode: Create Instance](./media/cal-s4h/s4h-pic10a.png)

1. Haga clic en **Crear** y, en el cuadro de mensaje que aparece, haga clic en **Aceptar**.

   ![Tamaños de máquinas virtuales que admite SAP CAL](./media/cal-s4h/s4h-pic10b.png)

1. En el cuadro de diálogo **Clave privada**, haga clic en **Almacenar** para almacenar la clave privada en SAP CAL. Para usar la protección con contraseña para la clave privada, haga clic en **Descargar**. 

   ![Clave privada de SAP CAL](./media/cal-s4h/s4h-pic10c.png)

1. Lea el mensaje de **advertencia** de SAP CAL y haga clic en **Aceptar**.

   ![Advertencia de SAP CAL](./media/cal-s4h/s4h-pic10d.png)

    Ahora se realizará la implementación. Después de un tiempo, y en función del tamaño y la complejidad de la solución (SAP CAL ofrece una estimación), el estado pasa a mostrarse como activo y listo para usarse.

1. Para buscar las máquinas virtuales que se recopilan con los demás recursos asociados de un grupo de recursos, vaya a Azure Portal: 

   ![Objetos de SAP CAL implementados en el nuevo portal](./media/cal-s4h/sapcaldeplyment_portalview.png)

1. En el portal de SAP CAL, el estado aparece como **Activo**. Para conectarse a la solución, haga clic en **Conectar**. Se implementan diferentes opciones para conectarse a los distintos componentes en esta solución.

   ![Instancias de SAP CAL](./media/cal-s4h/active_solution.png)

1. Para poder usar una de las opciones para conectarse a los sistemas implementados, haga clic en **Guía de introducción**. 

   ![Conectarse a la instancia](./media/cal-s4h/connect_to_solution.png)

    La documentación asigna los nombres de los usuarios en cada uno de los métodos de conectividad. Las contraseñas de dichos usuarios se establecen en la contraseña maestra que se definió al comienzo del proceso de implementación. En la documentación, otros usuarios más funcionales se enumeran junto con sus contraseñas, que se pueden usar para iniciar sesión en el sistema implementado. 

    Por ejemplo, si utiliza la GUI de SAP que está preinstalada en la máquina del Escritorio remoto de Windows, puede que el sistema S/4 sea parecido al siguiente:

   ![SM50 en la GUI de SAP preinstalada](./media/cal-s4h/gui_sm50.png)

    O bien, si utiliza DBACockpit, puede que la instancia se parezca a la siguiente:

   ![SM50 en la GUI de SAP de DBACockpit](./media/cal-s4h/dbacockpit.png)

En unas horas, una aplicación SAP S/4 correcta se implementa en Azure.

Si ha adquirido una suscripción a SAP CAL, SAP admite totalmente implementaciones a través de SAP CAL en Azure. La cola de compatibilidad es BC-VCM-CAL.







