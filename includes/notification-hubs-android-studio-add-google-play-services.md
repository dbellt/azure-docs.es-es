---
title: archivo de inclusión
description: archivo de inclusión
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 02/05/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: cd202c98ed605209f5600965ecdb6c0b4c03c17e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67509126"
---
1. En Android Studio, seleccione **Tools** (Herramientas) en el menú y, luego, seleccione **SDK Manager**. 
2. Busque la versión de destino de Android SDK se usa en el proyecto. Luego, seleccione **Show Package Details** (Mostrar detalles del paquete). 

    ![Android SDK Manager: selección de la versión de destino](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)
3. Seleccione **Google APIs** (API de Google), si aún no está instalado este componente.

    ![Android SDK Manager: Google APIs seleccionado](./media/notification-hubs-android-studio-add-google-play-services/googole-apis-selected.png)
4. Cambie a la pestaña **SDK Tools**. Si no ha instalado todavía Google Play Services, seleccione **Google Play Services**, tal como se muestra en la imagen siguiente. A continuación, seleccione **Apply** (Aplicar) para realizar la instalación. Tome nota de la ruta de acceso del SDK para usarla en el paso siguiente.

    ![Android SDK Manager: Google Play Services seleccionado](./media/notification-hubs-android-studio-add-google-play-services/google-play-services-selected.png)
3. Si ve el cuadro de diálogo **Confirm Change** (Confirmar cambio), seleccione **OK** (Aceptar). El instalador de componentes se instala los componentes solicitados. Seleccione **Finish** (Finalizar) una vez instalados los componentes.
4. Seleccione **OK** (Aceptar) para cerrar el cuadro de diálogo **Settings for New Projects** (Configuración para nuevos proyectos).  
5. Abra el archivo build.gradle en el directorio **app** y, luego, agregue la siguiente línea debajo de `dependencies`. 

    ```gradle
    implementation 'com.google.android.gms:play-services-gcm:16.0.0'
    ```
5. Seleccione el icono **Sync Now** (Sincronizar ahora) en la barra de herramientas.

    ![Sincronización con Gradle](./media/notification-hubs-android-studio-add-google-play-services/gradle-sync.png)
1. Abra el archivo AndroidManifest.xml y, luego, agregue la siguiente etiqueta a la etiqueta *application*.

    ```xml
    <meta-data android:name="com.google.android.gms.version"
         android:value="@integer/google_play_services_version" />
    ```
