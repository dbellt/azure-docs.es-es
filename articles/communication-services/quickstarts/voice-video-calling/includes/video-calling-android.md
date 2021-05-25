---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: bab9c177364f75461d4af70f5ac466d4e2c9ce7c
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483262"
---
Introducción al uso de Azure Communication Services con la biblioteca cliente de llamada de Communication Services para agregar llamadas de vídeo cara a cara a la aplicación Aprenderá a iniciar videollamadas y a responder a ellas con Calling SDK de Azure Communication Services para Android.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio), para crear la aplicación de Android.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.

### <a name="create-an-android-app-with-an-empty-activity"></a>Creación de una aplicación de Android con una actividad vacía

En Android Studio, seleccione Start a new Android Studio project (Iniciar un nuevo proyecto de Android Studio).

:::image type="content" source="../media/android/studio-new-project.png" alt-text="Captura de pantalla que muestra el botón &quot;Start a new Android Studio Project&quot; (&quot;Iniciar un nuevo proyecto de Android Studio&quot;) seleccionado en Android Studio.":::

Seleccione la plantilla de proyecto "Actividad vacía" en "Teléfono y tableta".

:::image type="content" source="../media/android/studio-blank-activity.png" alt-text="Captura de pantalla que muestra la opción &quot;Actividad vacía&quot; seleccionada en la pantalla de la plantilla de proyecto.":::

Seleccione el SDK mínimo de la "API 26: Android 8.0 (Oreo)" o una versión posterior.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Captura de pantalla que muestra la opción &quot;Actividad vacía&quot; seleccionada en la pantalla 2 de la plantilla de proyecto.":::

### <a name="install-the-package"></a>Instalar el paquete

Busque el nivel de proyecto `build.gradle` y asegúrese de agregar `mavenCentral()` a la lista de repositorios en `buildscript` y `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
Luego, en el nivel de módulo `build.gradle`, agregue las siguientes líneas a las dependencias y secciones de Android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0'
    ...
}
```
### <a name="add-permissions-to-application-manifest"></a>Adición de permisos al manifiesto de aplicación

Para solicitar los permisos necesarios para realizar una llamada, primero se deben declarar en el manifiesto de aplicación (`app/src/main/AndroidManifest.xml`). Reemplace el contenido del archivo por lo siguiente:

```xml
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.contoso.acsquickstart">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!--Our Calling SDK depends on the Apache HTTP SDK.
When targeting Android SDK 28+, this library needs to be explicitly referenced.
See https://developer.android.com/about/versions/pie/android-9.0-changes-28#apache-p-->
        <uses-library android:name="org.apache.http.legacy" android:required="false"/>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
    
```
### <a name="set-up-the-layout-for-the-app"></a>Configuración del diseño de la aplicación

Hace falta una entrada de texto para el identificador de destinatario, un botón para realizar la llamada y otro para colgarla. También se necesitan dos botones para activar y desactivar el vídeo local. Es necesario colocar dos contenedores para las secuencias de vídeo locales y remotas. Se pueden agregar a través del diseñador o editando el XML de diseño. Vaya a (`app/src/main/res/layout/activity_main.xml`) y reemplace el contenido del archivo por lo siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <EditText
                android:id="@+id/callee_id"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ems="10"
                android:gravity="center"
                android:hint="Callee Id"
                android:inputType="textPersonName"
                app:layout_constraintBottom_toTopOf="@+id/call_button"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintVertical_bias="0.064" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content">
                <Button
                    android:id="@+id/call_button"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="16dp"
                    android:gravity="center"
                    android:text="Call"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent" />
                <Button
                    android:id="@+id/show_preview"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="16dp"
                    android:gravity="center"
                    android:text="Show Video"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent" />
                <Button
                    android:id="@+id/hide_preview"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="16dp"
                    android:gravity="center"
                    android:text="Hide Video"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent" />
                <Button
                    android:id="@+id/hang_up"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="16dp"
                    android:gravity="center"
                    android:text="Hang Up"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent" />
            </LinearLayout>
            <FrameLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent">
                <LinearLayout
                    android:id="@+id/remotevideocontainer"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:gravity="end"
                    android:orientation="horizontal"
                    android:padding="10dp"></LinearLayout>
                <LinearLayout
                    android:id="@+id/localvideocontainer"
                    android:layout_width="180dp"
                    android:layout_height="320dp"
                    android:layout_gravity="right|bottom"
                    android:orientation="horizontal"
                    android:padding="10dp"></LinearLayout>

            </FrameLayout>
        </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Creación de los enlaces y scaffolding de actividades principales

Con el diseño creado, se pueden agregar los enlaces, así como el scaffolding básico de la actividad. La actividad administrará la solicitud de los permisos en tiempo de ejecución, la creación del agente de llamadas y la realización de llamadas cuando se presione el botón. Cada proceso se tratará en su propia sección. El método `onCreate` se reemplazará para invocar `getAllPermissions` y `createAgent`, así como para agregar los enlaces para el botón de llamada. Solo se producirá una vez cuando se cree la actividad. Para obtener más información sobre `onCreate`, consulte la guía [Descripción del ciclo de vida de la actividad](https://developer.android.com/guide/components/activities/activity-lifecycle).

Vaya a **MainActivity.java** y reemplace el contenido por el siguiente código:

```java
package com.example.acsquickstart;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import android.widget.LinearLayout;
import android.content.Context;
import com.azure.android.communication.calling.CallState;
import com.azure.android.communication.calling.CallingCommunicationException;
import com.azure.android.communication.calling.CameraFacing;
import com.azure.android.communication.calling.PropertyChangedEvent;
import com.azure.android.communication.calling.VideoDeviceInfo;
import com.azure.android.communication.common.CommunicationUserIdentifier;
import com.azure.android.communication.common.CommunicationIdentifier;
import com.azure.android.communication.common.CommunicationTokenCredential;
import com.azure.android.communication.calling.CallAgent;
import com.azure.android.communication.calling.CallClient;
import com.azure.android.communication.calling.StartCallOptions;
import com.azure.android.communication.calling.DeviceManager;
import com.azure.android.communication.calling.VideoOptions;
import com.azure.android.communication.calling.LocalVideoStream;
import com.azure.android.communication.calling.VideoStreamRenderer;
import com.azure.android.communication.calling.VideoStreamRendererView;
import com.azure.android.communication.calling.CreateViewOptions;
import com.azure.android.communication.calling.ScalingMode;
import com.azure.android.communication.calling.IncomingCall;
import com.azure.android.communication.calling.Call;
import com.azure.android.communication.calling.AcceptCallOptions;
import com.azure.android.communication.calling.ParticipantsUpdatedEvent;
import com.azure.android.communication.calling.RemoteParticipant;
import com.azure.android.communication.calling.RemoteVideoStream;
import com.azure.android.communication.calling.RemoteVideoStreamsEvent;
import com.azure.android.communication.calling.RendererListener;

import java.util.ArrayList;
import java.util.List;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;

public class MainActivity extends AppCompatActivity {

    private CallAgent callAgent;
    private LocalVideoStream currentVideoStream;
    private DeviceManager deviceManager;
    private IncomingCall incomingCall;
    private Call call;
    VideoStreamRenderer previewRenderer;
    VideoStreamRendererView preview;
    final Map<Integer, StreamData> streamData = new HashMap<>();
    private boolean renderRemoteVideo = true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getAllPermissions();
        createAgent();

        handleIncomingCall();

        Button callButton = findViewById(R.id.call_button);
        callButton.setOnClickListener(l -> startCall());
        Button hangupButton = findViewById(R.id.hang_up);
        hangupButton.setOnClickListener(l -> hangUp());
        Button startVideo = findViewById(R.id.show_preview);
        startVideo.setOnClickListener(l -> turnOnLocalVideo());
        Button stopVideo = findViewById(R.id.hide_preview);
        stopVideo.setOnClickListener(l -> turnOffLocalVideo());
    }

    /**
     * Request each required permission if the app doesn't already have it.
     */
    private void getAllPermissions() {
        // See section on requesting permissions
    }

    /**
      * Create the call agent for placing calls
      */
    private void createAgent() {
        // See section on creating the call agent
    }

    /**
     * Place a call to the callee id provided in `callee_id` text input.
     */
    private void startCall() {
        // See section on starting the call
    }

    /**
     * Handle incoming calls
     */
    private void handleIncomingCall() {
        // See section on answering incoming call
    }

    /**
     * Mid-call operations
     */
    public void turnOnLocalVideo() {
        // See setion on 
    }
    public void turnOffLocalVideo() {
        
    }

    /**
     * End calls
     */
    private void hangUp() {
        // See section on ending the call
    }    
}
```

### <a name="request-permissions-at-runtime"></a>Solicitud de permisos en tiempo de ejecución

Para Android 6.0 y las versiones posteriores (nivel de API 23) y `targetSdkVersion` 23 o las versiones posteriores, los permisos se conceden en tiempo de ejecución en lugar de cuando se instala la aplicación. A fin de admitirlo, se puede implementar `getAllPermissions` para llamar a `ActivityCompat.checkSelfPermission` y `ActivityCompat.requestPermissions` para cada permiso necesario.

```java
/**
 * Request each required permission if the app doesn't already have it.
 */
private void getAllPermissions() {
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO, Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_PHONE_STATE};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 1);
    }
}
```

> [!NOTE]
> Al diseñar la aplicación, tenga en cuenta cuándo deben solicitarse estos permisos. Se deben solicitar a medida que sean necesarios, pero no con anterioridad. Para obtener más información, consulte la [guía de permisos de Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient es el punto de entrada principal al SDK de llamadas.|
| CallAgent | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationTokenCredential | CommunicationTokenCredential se usa como la credencial de token para crear una instancia de CallAgent.|
| CommunicationIdentifier | CommunicationIdentifier se usa como un tipo de participante diferente que podría formar parte de una llamada.|

## <a name="create-an-agent-from-the-user-access-token"></a>Creación de un agente a partir del token de acceso de usuario

Con el token de usuario, se puede crear una instancia del agente de llamadas autenticado. Por lo general, este token se generará a partir de un servicio con autenticación específica de la aplicación. Para obtener más información sobre los tokens de acceso de usuario, consulte la guía [Tokens de acceso de usuario](../../access-tokens.md). En el inicio rápido, reemplace `<User_Access_Token>` por un token de acceso de usuario generado para el recurso de Azure Communication Service.

```java
/**
 * Create the call agent for placing calls
 */
private void createAgent() {
    Context context = this.getApplicationContext();
    String userToken = "<User_Access_Token>";
    try {
        CommunicationTokenCredential credential = new CommunicationTokenCredential(userToken);
        CallClient callClient = new CallClient();
        deviceManager = callClient.getDeviceManager(context).get();
        callAgent = callClient.createCallAgent(getApplicationContext(), credential).get(); 
    } catch (Exception ex) {
        Toast.makeText(context, "Failed to create call agent.", Toast.LENGTH_SHORT).show();
    }
}
```
## <a name="start-a-video-call-using-the-call-agent"></a>Inicio de una videollamada mediante el agente de llamadas

La llamada se puede realizar a través del agente de llamadas, y solo se debe proporcionar una lista de los identificadores de destinatarios y las opciones de llamada. 

Para realizar una videollamada, debe enumerar las cámaras locales mediante la API `getCameras` de `deviceManager`. Una vez que seleccione la cámara deseada, úsela para crear una instancia de `LocalVideoStream` y pásela a `videoOptions` como elemento de la matriz `localVideoStream` a un método de llamada. Cuando la llamada se conecta, se iniciará automáticamente el envío de una secuencia de vídeo de la cámara seleccionada al otro participante.

```java
private void startCall() {
    Context context = this.getApplicationContext();
    EditText calleeIdView = findViewById(R.id.callee_id);
    String calleeId = calleeIdView.getText().toString();
    ArrayList<CommunicationIdentifier> participants = new ArrayList<CommunicationIdentifier>();
    StartCallOptions options = new StartCallOptions();
    List<VideoDeviceInfo> cameras = deviceManager.getCameras();
    if(!cameras.isEmpty()) {
        VideoDeviceInfo camera = chooseCamera(cameras);
        currentVideoStream = new LocalVideoStream(camera, context);
        LocalVideoStream[] videoStreams = new LocalVideoStream[1];
        videoStreams[0] = currentVideoStream;
        VideoOptions videoOptions = new VideoOptions(videoStreams);
        options.setVideoOptions(videoOptions);
        showPreview(currentVideoStream);
    }
    participants.add(new CommunicationUserIdentifier(calleeId));

    call = callAgent.startCall(
            context,
            participants,
            options);
    
    //Subcribe to events on updates of call state and remote participants
    call.addOnRemoteParticipantsUpdatedListener(this::handleRemoteParticipantsUpdate);
    call.addOnStateChangedListener(this::handleCallOnStateChanged);
}
```

En este inicio rápido, se ha decidido usar la cámara frontal del dispositivo. La función `chooseCamera` toma la enumeración de las cámaras como entrada. Si la cámara frontal no está disponible, se usará la primera cámara disponible. Si no hay ninguna cámara disponible al pulsar `Start Call`, se iniciará una llamada de audio. Sin embargo, si el participante remoto respondió con vídeo, todavía se puede ver la secuencia de vídeo remota. 

```java
VideoDeviceInfo chooseCamera(List<VideoDeviceInfo> cameras) {
    for (VideoDeviceInfo camera : cameras) {
        if (camera.getCameraFacing() == CameraFacing.FRONT) {
            return camera;
        }
    }
    return cameras.get(0);
}
```

Una vez que se crea una instancia de `LocalVideoStream`, se puede crear un representador para mostrarla en la interfaz de usuario. 

```java
private void showPreview(LocalVideoStream stream) {
    previewRenderer = new VideoStreamRenderer(stream, this);
    LinearLayout layout = ((LinearLayout)findViewById(R.id.localvideocontainer));
    preview = previewRenderer.createView(new CreateViewOptions(ScalingMode.FIT));
    runOnUiThread(() -> {
        layout.addView(preview);
    });
}
```

## <a name="accept-an-incoming-call"></a>Aceptar una llamada entrante

La llamada entrante se puede obtener mediante la suscripción a `addOnIncomingCallListener` en `callAgent`. 

```java
private void handleIncomingCall() {
    callAgent.addOnIncomingCallListener((incomingCall) -> {
        this.incomingCall = incomingCall;
        Executors.newCachedThreadPool().submit(this::answerIncomingCall);
    });
}
```

Para aceptar una llamada con la cámara de vídeo encendida, enumere las cámaras locales mediante la API getCameras de deviceManager, elija una cámara y construya una instancia de `LocalVideoStream` y pásala a `acceptCallOptions` antes de llamar al método "accept" en un objeto `call`. 
```java
private void answerIncomingCall() {
    Context context = this.getApplicationContext();
    if (incomingCall == null){
        return;
    }
    AcceptCallOptions acceptCallOptions = new AcceptCallOptions();
    List<VideoDeviceInfo> cameras = deviceManager.getCameras();
    if(!cameras.isEmpty()) {
        VideoDeviceInfo camera = chooseCamera(cameras);
        currentVideoStream = new LocalVideoStream(camera, context);
        LocalVideoStream[] videoStreams = new LocalVideoStream[1];
        videoStreams[0] = currentVideoStream;
        VideoOptions videoOptions = new VideoOptions(videoStreams);
        acceptCallOptions.setVideoOptions(videoOptions);
        showPreview(currentVideoStream);
    }
    try {
        call = incomingCall.accept(context, acceptCallOptions).get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }

    //Subcribe to events on updates of call state and remote participants
    call.addOnRemoteParticipantsUpdatedListener(this::handleRemoteParticipantsUpdate);
    call.addOnStateChangedListener(this::handleCallOnStateChanged);
}
```

## <a name="remote-participant-and-remote-video-streams"></a>Secuencias de vídeo remotas y participantes remotos

Cuando se inicia una llamada o se responde a una llamada entrante, es necesario suscribirse al evento `addOnRemoteParticipantsUpdatedListener` para controlar a los participantes remotos. 

```java
call.addOnRemoteParticipantsUpdatedListener(this::handleRemoteParticipantsUpdate);
```

### <a name="remote-participant-and-remote-video-stream-update"></a>Actualización de secuencias de vídeo remotas y participantes remotos

En este inicio rápido solo se tratarán los participantes agregados porque se va a implementar la llamada 1:1. Cuando se quite el participante remoto, finalizará la llamada. Con los participantes agregados, nos suscribiremos a `addOnVideoStreamsUpdatedListener` para administrar las actualizaciones de las secuencias de vídeo. 

```java
public void handleRemoteParticipantsUpdate(ParticipantsUpdatedEvent args) {
    LinearLayout participantVideoContainer = findViewById(R.id.remotevideocontainer);
    handleAddedParticipants(args.getAddedParticipants(),participantVideoContainer);
}

private void handleAddedParticipants(List<RemoteParticipant> participants, LinearLayout participantVideoContainer) {
    for (RemoteParticipant remoteParticipant : participants) {
        remoteParticipant.addOnVideoStreamsUpdatedListener(videoStreamsEventArgs -> videoStreamsUpdated(videoStreamsEventArgs));
    }
}

private void videoStreamsUpdated(RemoteVideoStreamsEvent videoStreamsEventArgs) {
    for(RemoteVideoStream stream : videoStreamsEventArgs.getAddedRemoteVideoStreams()) {
        StreamData data = new StreamData(stream, null, null);
        streamData.put(stream.getId(), data);
        if (renderRemoteVideo) {
            startRenderingVideo(data);
        }
    }

    for(RemoteVideoStream stream : videoStreamsEventArgs.getRemovedRemoteVideoStreams()) {
        stopRenderingVideo(stream);
    }
}
```

### <a name="render-remote-videos"></a>Representación de vídeos remotos

Cree un representador de secuencias de vídeo remotas y asócielo a la vista para empezar a representar la vista remota. Deseche la vista para dejar de representarla. 

```java
void startRenderingVideo(StreamData data){
    if (data.renderer != null) {
        return;
    }
    LinearLayout layout = ((LinearLayout)findViewById(R.id.remotevideocontainer));
    data.renderer = new VideoStreamRenderer(data.stream, this);
    data.renderer.addRendererListener(new RendererListener() {
        @Override
        public void onFirstFrameRendered() {
            String text = data.renderer.getSize().toString();
            Log.i("MainActivity", "Video rendering at: " + text);
        }

        @Override
        public void onRendererFailedToStart() {
            String text = "Video failed to render";
            Log.i("MainActivity", text);
        }
    });
    data.rendererView = data.renderer.createView(new CreateViewOptions(ScalingMode.FIT));
    runOnUiThread(() -> {
        layout.addView(data.rendererView);
    });
}

void stopRenderingVideo(RemoteVideoStream stream) {
    StreamData data = streamData.get(stream.getId());
    if (data == null || data.renderer == null) {
        return;
    }
    runOnUiThread(() -> {
        ((LinearLayout) findViewById(R.id.remotevideocontainer)).removeAllViews();
    });
    data.rendererView = null;
    // Dispose renderer
    data.renderer.dispose();
    data.renderer = null;
}

static class StreamData {
    RemoteVideoStream stream;
    VideoStreamRenderer renderer;
    VideoStreamRendererView rendererView;
    StreamData(RemoteVideoStream stream, VideoStreamRenderer renderer, VideoStreamRendererView rendererView) {
        this.stream = stream;
        this.renderer = renderer;
        this.rendererView = rendererView;
    }
}
```

## <a name="call-state-update"></a>Actualización del estado de la llamada
En este inicio rápido, se administrarán los cambios de `CallState`. Cuando la llamada está conectada, se controla al participante remoto y, cuando está desconectada, se elimina `previewRenderer` para detener el vídeo local. 

```groovy
private void handleCallOnStateChanged(PropertyChangedEvent args) {
    if (call.getState() == CallState.CONNECTED) {
        LinearLayout participantVideoContainer = findViewById(R.id.remotevideocontainer);
        handleAddedParticipants(call.getRemoteParticipants(),participantVideoContainer);
    }
    if (call.getState() == CallState.DISCONNECTED) {
        if (previewRenderer != null) {
            previewRenderer.dispose();
        }
    }
}
```

## <a name="end-a-call"></a>Finalizar una llamada
Para finalizar la llamada, llame a la función `hangUp()` en la instancia de llamada y elimine `previewRenderer` para detener el vídeo local. 
```java
private void hangUp() {
    try {
        call.hangUp().get();
    } catch (ExecutionException | InterruptedException e) {
        e.printStackTrace();
    }
    if (previewRenderer != null) {
        previewRenderer.dispose();
    }
}
```

## <a name="run-the-code"></a>Ejecución del código

Ahora se puede iniciar la aplicación con el botón `Run 'App'` de la barra de herramientas de Android Studio. 

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-android-quickstarts/tree/main/videoCallingQuickstart).
