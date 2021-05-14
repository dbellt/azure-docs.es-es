---
title: 'Inicio rápido: Adición de la posibilidad de incorporar una reunión de Teams a una aplicación de Android mediante Azure Communication Services'
description: En este inicio rápido, aprenderá a usar la biblioteca de integración de Teams de Azure Communication Services para Android.
author: palatter
ms.author: palatter
ms.date: 01/25/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: cbf56f6567c3c7857afcd71ed9f6dc67be48f9d1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108792383"
---
En este inicio rápido, aprenderá cómo unirse a una reunión de Microsoft Teams mediante la biblioteca insertada de Teams de Azure Communication Services para Android.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio), para crear la aplicación de Android.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.

## <a name="setting-up"></a>Instalación

### <a name="create-an-android-app-with-an-empty-activity"></a>Creación de una aplicación de Android con una actividad vacía

En Android Studio, seleccione Start a new Android Studio project (Iniciar un nuevo proyecto de Android Studio).

:::image type="content" source="../media/android/studio-new-project.png" alt-text="Captura de pantalla que muestra el botón &quot;Start a new Android Studio Project&quot; (&quot;Iniciar un nuevo proyecto de Android Studio&quot;) seleccionado en Android Studio.":::

Seleccione la plantilla de proyecto "Actividad vacía" en "Teléfono y tableta".

:::image type="content" source="../media/android/studio-blank-activity.png" alt-text="Captura de pantalla que muestra la opción &quot;Actividad vacía&quot; seleccionada en la pantalla de la plantilla de proyecto.":::

Asigne al proyecto el nombre `TeamsEmbedAndroidGettingStarted`, establezca que el lenguaje es Java y seleccione un SDK mínimo de "API 21: Android 5.0 (Lollipop)", o superior.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Captura de pantalla que muestra la opción &quot;Actividad vacía&quot; seleccionada en la pantalla 2 de la plantilla de proyecto.":::


### <a name="install-the-azure-package"></a>Instalación del paquete de Azure

En el archivo `build.gradle` de nivel de aplicación (**carpeta de la aplicación**), agregue las siguientes líneas a las secciones de dependencias y de Android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
}
```

```groovy
dependencies {
    ...
    implementation 'com.azure.android:azure-communication-common:1.0.0-beta.8'
    ...
}
```

Actualice los valores del archivo `build.gradle`.

```groovy
 buildTypes {
        release {
        ...
            minifyEnabled true
            shrinkResources true
        ...
    }
}
```


### <a name="install-the-teams-embed-package"></a>Instalación del paquete de inserción de Teams

Descargue el paquete [`MicrosoftTeamsSDK`.](https://github.com/Azure/communication)

Luego, descomprima la carpeta `MicrosoftTeamsSDK` en la carpeta de la aplicación del proyecto. Por ejemplo, `TeamsEmbedAndroidGettingStarted/app/MicrosoftTeamsSDK`.

### <a name="add-teams-embed-package-to-your-buildgradle"></a>Incorporación del paquete de inserción de Teams a build.gradle

En el archivo `build.gradle` de nivel de aplicación, agregue la siguiente línea al final del archivo:

```groovy
apply from: 'MicrosoftTeamsSDK/MicrosoftTeamsSDK.gradle'
```

Sincronice el proyecto con archivos de Gradle.

### <a name="create-application-class"></a>Creación de una clase de aplicación

Cree un archivo de clase Java denominado `TeamsEmbedAndroidGettingStarted`, Esta clase será la clase de aplicación, que debe extender `TeamsSDKApplication`. [Documentación de Android](https://developer.android.com/reference/android/app/Application)

:::image type="content" source="../media/android/application-class-location.png" alt-text="Captura de pantalla que muestra dónde se crea una clase de aplicación en Android Studio":::

```java
package com.microsoft.teamsembedandroidgettingstarted;

import com.microsoft.teamssdk.app.TeamsSDKApplication;

public class TeamsEmbedAndroidGettingStarted extends TeamsSDKApplication {
}
```

### <a name="update-themes"></a>Actualización de temas

Establezca el nombre de estilo en `AppTheme` en los archivos `theme.xml` y `theme.xml (night)`.

:::image type="content" source="../media/android/theme-settings.png" alt-text="Captura de pantalla que muestra los archivos theme.xml en Android Studio":::

```xml
<style name="AppTheme" parent="Theme.AppCompat.DayNight.DarkActionBar">
```

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
```

### <a name="add-permissions-to-application-manifest"></a>Adición de permisos al manifiesto de aplicación

Agregue el permiso `RECORD_AUDIO` al manifiesto de aplicación (`app/src/main/AndroidManifest.xml`):  


```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.yourcompany.TeamsEmbedAndroidGettingStarted">
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <application
```

### <a name="add-app-name-and-theme-to-application-manifest"></a>Incorporación del nombre de la aplicación y el tema al manifiesto de aplicación

Agregue "xmlns: Tools ="http://schemas.android.com/tools" al manifiesto.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.yourcompany.TeamsEmbedAndroidGettingStarted">
```

Agregue `.TeamsEmbedAndroidGettingStarted` a `android:name`, `android:name` a `tools:replace` y cambie `android:theme` a `@style/AppTheme`

```xml
<application
    android:name=".TeamsEmbedAndroidGettingStarted"
    tools:replace="android:name"
    android:theme="@style/AppTheme"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true">
```

### <a name="set-up-the-layout-for-the-app"></a>Configuración del diseño de la aplicación

Cree un botón con el identificador `join_meeting`. Vaya al archivo de diseño (`app/src/main/res/layout/activity_main.xml`) y reemplace el contenido del archivo por lo siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/join_meeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Join Meeting"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Creación de los enlaces y scaffolding de actividades principales

Con el diseño creado, se puede el scaffolding básico de la actividad junto con los enlaces necesarios. La actividad controlará la solicitud de los permisos en tiempo de ejecución, la creación del cliente de la reunión y la incorporación a una reunión cuando se presiona el botón. Cada proceso se tratará en su propia sección. 

El método `onCreate` se reemplazará para invocar a `getAllPermissions` y `createAgent`, y para agregar los enlaces para el botón `Join Meeting`. Solo se producirá una vez cuando se cree la actividad. Para obtener más información sobre `onCreate`, consulte la guía [Descripción del ciclo de vida de la actividad](https://developer.android.com/guide/components/activities/activity-lifecycle).

Vaya a **MainActivity.java** y reemplace el contenido por el siguiente código:

```java
package com.yourcompany.teamsembedandroidgettingstarted;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;

import com.azure.android.communication.common.CommunicationTokenCredential;
import com.azure.android.communication.common.CommunicationTokenRefreshOptions;
import com.azure.android.communication.ui.meetings.MeetingUIClientJoinOptions;
import com.azure.android.communication.ui.meetings.MeetingUIClient;
import com.azure.android.communication.ui.meetings.MeetingUIClientTeamsMeetingLinkLocator;

import java.util.ArrayList;
import java.util.concurrent.Callable;

public class MainActivity extends AppCompatActivity {

    private final String displayName = "John Smith";

    private MeetingUIClient meetingUIClient;
    private MeetingUIClientJoinOptions meetingJoinOptions;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        meetingJoinOptions = new MeetingUIClientJoinOptions(displayName, false);
        
        getAllPermissions();
        createMeetingClient();

        Button joinMeeting = findViewById(R.id.join_meeting);
        joinMeeting.setOnClickListener(l -> joinMeeting());
    }

    private void createMeetingClient() {
        // See section on creating meeting client
    }

    private void joinMeeting() {
        // See section on joining a meeting
    }

    private void getAllPermissions() {
        // See section on getting permissions
    }
}
```

### <a name="request-permissions-at-runtime"></a>Solicitud de permisos en tiempo de ejecución

Para Android 6.0 y las versiones posteriores (nivel de API 23) y `targetSdkVersion` 23 o las versiones posteriores, los permisos se conceden en tiempo de ejecución en lugar de cuando se instala la aplicación. Para admitir esto, se puede implementar `getAllPermissions` para llamar a `ActivityCompat.checkSelfPermission` y `ActivityCompat.requestPermissions` para cada permiso necesario.

```java
/**
 * Request each required permission if the app doesn't already have it.
 */
private void getAllPermissions() {
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 202);
    }
}
```

> [!NOTE]
> Al diseñar la aplicación, tenga en cuenta cuándo deben solicitarse estos permisos. Se deben solicitar a medida que sean necesarios, pero no con anterioridad. Para más información, consulte [Guía de permisos de Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca de integración de Teams de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| MeetingUIClient| MeetingUIClient es el punto de entrada principal a la biblioteca de integración de Teams. |
| MeetingUIClientJoinOptions | MeetingUIClientJoinOptions se usa para las opciones configurables, como el nombre para mostrar. |
| MeetingUIClientTeamsMeetingLinkLocator | MeetingUIClientTeamsMeetingLinkLocator se usa para establecer la dirección URL de la reunión para unirse a una reunión. |
| MeetingUIClientGroupCallLocator | MeetingUIClientGroupCallLocator se usa para establecer el identificador de grupo al que se va a unir. |
| MeetingUIClientCallState | MeetingUIClientCallState se utiliza para notificar los cambios en el estado de la llamada. Estas son las opciones: `connecting`, `waitingInLobby`, `connected` y `ended`. |
| MeetingUIClientEventListener | MeetingUIClientEventListener se usa para recibir eventos, como cambios en el estado de la llamada. |
| MeetingUIClientIdentityProvider | MeetingUIClientIdentityProvider se usa para asignar los detalles del usuario a los usuarios de una reunión. |
| MeetingUIClientUserEventListener | MeetingUIClientUserEventListener se usa para proporcionar información sobre las acciones del usuario en la interfaz de usuario. |

## <a name="create-a-meetingclient-from-the-user-access-token"></a>Creación de una instancia de MeetingClient a partir del token de acceso de usuario

Se puede crear una instancia de un cliente de reunión autenticado con un token de acceso de usuario. Normalmente, este token lo genera un servicio con autenticación específica de la aplicación. Para más información sobre los tokens de acceso de usuario, consulte la guía [Tokens de acceso de usuario](../../access-tokens.md). En el inicio rápido, reemplace `<USER_ACCESS_TOKEN>` por un token de acceso de usuario generado para el recurso de Azure Communication Service.

```java
private void createMeetingClient() { 
    try {
        CommunicationTokenRefreshOptions refreshOptions = new CommunicationTokenRefreshOptions(tokenRefresher, true, "<USER_ACCESS_TOKEN>");
        CommunicationTokenCredential credential = new CommunicationTokenCredential(refreshOptions);
        meetingUIClient = new MeetingUIClient(credential);
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to create meeting client: " + ex.getMessage(), Toast.LENGTH_SHORT).show();
    }
}
```

## <a name="setup-token-refreshing"></a>Actualización del token de instalación

Cree un método Callable `tokenRefresher`. Luego, cree un método `fetchToken` para obtener el token del usuario. [Aquí puede encontrar las instrucciones necesarias para hacerlo](../../access-tokens.md?pivots=programming-language-java)

```java
Callable<String> tokenRefresher = () -> {
    return fetchToken();
};

public String fetchToken() {
    // Get token
    return USER_ACCESS_TOKEN;
}
```

## <a name="get-the-teams-meeting-link"></a>Obtención del vínculo de la reunión de Teams

El vínculo de la reunión de Teams se puede recuperar mediante las Graph API. Esto se detalla en la [documentación de Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
El SDK de llamada de Communication Services acepta un vínculo a toda la reunión de Teams. Este vínculo se devuelve como parte del recurso `onlineMeeting`, al que se accede bajo la propiedad [`joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). También puede obtener la información necesaria de la reunión de la dirección URL **Unirse a la reunión** de la propia invitación a la reunión de Teams.

## <a name="start-a-meeting-using-the-meeting-client"></a>Inicio de una reunión con el cliente de reunión

La incorporación a una reunión se puede realizar a través de `MeetingUIClient` y solo requiere `MeetingUIClientTeamsMeetingLinkLocator` y `MeetingUIClientJoinOptions`. Reemplace `<MEETING_URL>` por la dirección URL de la reunión de Teams.

```java
/**
 * Join a meeting with a meetingURL.
 */
private void joinMeeting() {
    try {
        MeetingUIClientTeamsMeetingLinkLocator meetingUIClientTeamsMeetingLinkLocator = new MeetingUIClientTeamsMeetingLinkLocator(<MEETING_URL>);
        meetingUIClient.join(meetingUIClientTeamsMeetingLinkLocator, meetingJoinOptions);
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to join meeting: " + ex.getMessage(), Toast.LENGTH_SHORT).show();
    }
}
```

## <a name="launch-the-app-and-join-a-meeting"></a>Inicio de la aplicación e incorporación a una reunión

Ahora se puede iniciar la aplicación con el botón "Ejecutar aplicación" de la barra de herramientas (Mayús + F10). 

:::image type="content" source="../media/android/quickstart-android-join-meeting.png" alt-text="Captura de pantalla que muestra la aplicación completada.":::

:::image type="content" source="../media/android/quickstart-android-joined-meeting.png" alt-text="Captura de pantalla que muestra la aplicación completa después de incorporarse a la reunión.":::

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/teams-embed-android-getting-started).
