---
title: archivo de inclusión
description: archivo de inclusión
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 08/01/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 45bdd569741dc13181bcaf9e8587a02b3d02c621
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "68728858"
---
1. Inicie sesión en la [consola Firebase](https://firebase.google.com/console/). Si aún no tiene uno, cree un nuevo proyecto de Firebase.
2. Después de crear el proyecto, seleccione **Add Firebase to your Android app** (Añade Firebase a tu aplicación de Android). 

    ![Agregar Firebase a una aplicación de Android](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)

3. En la página **Add Firebase to your Android app** (Agregar Firebase a la aplicación Android), haga lo siguiente: 
    1. En **Android package name** (Nombre del paquete Android), escriba un nombre para el paquete. Por ejemplo: `tutorials.tutoria1.xamarinfcmapp`. 

        ![Especifique el nombre del paquete.](./media/notification-hubs-enable-firebase-cloud-messaging/specify-package-name-fcm-settings.png)
    2. Seleccione **Registrar aplicación**.  
    1. Seleccione **Download google-services.json** (Descargar google-services.json). A continuación, guarde el archivo en la carpeta del proyecto y seleccione **Siguiente**. Si aún no ha creado el proyecto de Visual Studio, puede realizar este paso después de crear el proyecto. 

        ![Descargue el archivo google-services.json.](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-service-button.png)
    6. Seleccione **Next** (Siguiente). 
    7. Seleccione **Skip this step** (Omitir este paso). 

        ![Omita el último paso.](./media/notification-hubs-enable-firebase-cloud-messaging/skip-this-step.png)
8. En la consola Firebase, seleccione el icono de la rueda dentada del proyecto. Luego, seleccione **Project Settings** (Configuración del proyecto).

    ![Seleccionar Project Settings (Configuración del proyecto)](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)
4. Si no ha descargado el archivo **google-services.json**, puede hacerlo en esta página. 

    ![Descargue el archivo google-services.json de la pestaña General.](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-services-json-general-page.png)
1. Cambie a la pestaña **Cloud Messaging** de la parte superior. Copie y guarde la **clave del servidor** para su uso posterior. Este valor se utilizará para configurar el centro de notificaciones.

    ![Copia de la clave del servidor](./media/notification-hubs-enable-firebase-cloud-messaging/server-key.png)
