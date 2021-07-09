---
title: 'Inicio rápido: unión a una reunión de Teams desde una aplicación de Windows'
description: En este tutorial aprenderá a unirse a una reunión de Teams mediante Calling SDK de Azure Communication Services para Windows.
author: aurighet
ms.author: aurighet
ms.date: 05/13/2021
ms.topic: include
ms.service: azure-communication-services
ms.openlocfilehash: b26c9ff7a2a7ec2f6898ed1434184e9a016c20d3
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560631"
---
En este artículo de inicio rápido aprenderá a unirse a una reunión de Teams mediante Calling SDK de Azure Communication Services para Windows.

## <a name="prerequisites"></a>Prerrequisitos

- Una [aplicación de Windows de llamadas de Communication Services](../../getting-started-with-calling.md) operativa.
- Una [implementación de Teams](/deployoffice/teams-install).


## <a name="add-the-teams-ui-controls-and-enable-the-teams-ui-controls"></a>Incorporación y habilitación de los controles de la interfaz de usuario de Teams

Reemplace el código de MainPage.xaml por el siguiente fragmento de código. El cuadro de texto se utilizará para especificar el contexto de reunión de Teams y el botón se usará para la unión a la reunión especificada:

```xml
<Page
    x:Class="CallingQuickstart.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:CallingQuickstart"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
    <StackPanel>
        <TextBox PlaceholderText="Please enter the Teams meeting link." TextWrapping="Wrap" x:Name="TeamsLinkTextBox" Margin="10,10,10,10" />
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Center">
            <Button Content="Join Teams Meeting" Click="JoinButton_ClickAsync" x:Name="JoinButton" Margin="10,10,10,10" />
            <Button Content="Leave Meeting" Click="LeaveButton_ClickAsync" x:Name="LeaveButton" Margin="10,10,10,10" />
        </StackPanel>
        <TextBlock TextWrapping="Wrap" x:Name="CallStatusTextBlock" Margin="10,10,10,10" />
        <TextBlock TextWrapping="Wrap" x:Name="RecordingStatusTextBlock" Margin="10,10,10,10" />
    </StackPanel>
</Page>
```

## <a name="enable-the-teams-ui-controls"></a>Habilitación de los controles de la interfaz de usuario de Teams

Reemplace el contenido de `MainPage.xaml.cs` por el siguiente fragmento de contenido:

```csharp
using System;
using System.Threading.Tasks;

using Windows.UI.Core;
using Windows.UI.Popups;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

using Azure.Communication;
using Azure.Communication.Calling;

namespace CallingQuickstart
{
    public sealed partial class MainPage : Page
    {
        string user_token_ = "<User_Access_Token>";

        Call call_;

        public MainPage()
        {
            this.InitializeComponent();
        }

        private async void JoinButton_ClickAsync(object sender, RoutedEventArgs e)
        {
            if (!await ValidateInput())
            {
                return;
            }

            //
            //  Create CallClient
            //
            CallClient call_client = new CallClient();

            //
            //  Create CallAgent
            //
            CommunicationTokenCredential token_credential;
            CallAgent call_agent;

            try
            {
                token_credential = new CommunicationTokenCredential(user_token_);

                CallAgentOptions call_agent_options = new CallAgentOptions();
                call_agent = await call_client.CreateCallAgent(token_credential, call_agent_options);
            }
            catch
            {
                await new MessageDialog("It was not possible to create call agent. Please check if token is valid.").ShowAsync();
                return;
            }

            //
            //  Join a Teams meeting
            //
            try
            {
                JoinCallOptions joinCallOptions = new JoinCallOptions();
                TeamsMeetingLinkLocator teamsMeetingLinkLocator = new TeamsMeetingLinkLocator(TeamsLinkTextBox.Text);
                call_ = await call_agent.JoinAsync(teamsMeetingLinkLocator, joinCallOptions);
            }
            catch
            {
                await new MessageDialog("It was not possible to join the Teams meeting. Please check if Teams Link is valid.").ShowAsync();
                return;
            }

            //
            //  Set up call callbacks
            //
            call_.OnStateChanged += Call_OnStateChangedAsync;
            call_.OnIsRecordingActiveChanged += Call_OnIsRecordingActiveChangedAsync;
        }

        private async void Call_OnStateChangedAsync(object sender, PropertyChangedEventArgs args)
        {
            await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                CallStatusTextBlock.Text = call_.State.ToString();
            });
        }
        
        private async void Call_OnIsRecordingActiveChangedAsync(object sender, PropertyChangedEventArgs args)
        {
            await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                RecordingStatusTextBlock.Text = call_.IsRecordingActive ? "Recording is active." : "Recording is inactive.";
            });
        }
        
        private async void LeaveButton_ClickAsync(object sender, RoutedEventArgs e)
        {
            try
            {
                await call_.HangUp(new HangUpOptions());
            }
            catch
            {
                await new MessageDialog("It was not possible to leave the Teams meeting.").ShowAsync();
            }
        }
        
        private async Task<bool> ValidateInput()
        {
            if (user_token_.StartsWith("<"))
            {
                await new MessageDialog("Please enter token in source code.").ShowAsync();
                return false;
            }
        
            if (TeamsLinkTextBox.Text.Trim().Length == 0 || !TeamsLinkTextBox.Text.StartsWith("http"))
            {
                await new MessageDialog("Please enter Teams meeting link.").ShowAsync();
                return false;
            }
        
            return true;
        }
    }
}
```

## <a name="get-the-teams-meeting-link"></a>Obtención del vínculo de la reunión de Teams

El vínculo de la reunión de Teams se puede recuperar mediante las Graph API. Esto se detalla en la [documentación de Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
El SDK de llamada de Communication Services acepta un vínculo a toda la reunión de Teams. Este vínculo se devuelve como parte del recurso `onlineMeeting`, al que se puede acceder bajo la [propiedad `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). También puede obtener la información de la reunión necesaria en la dirección URL **Join Meeting** (Unirse a la reunión) de la propia invitación a la reunión en Teams.

## <a name="launch-the-app-and-join-teams-meeting"></a>Inicio de la aplicación y unión a una reunión de Teams

Para compilar y ejecutar la aplicación en Visual Studio, puede seleccionar **Depurar** > **Iniciar depuración** o con el método abreviado de teclado (F5).

Inserte el contexto de Teams en el cuadro de texto y presione *Join Teams Meeting* (Unirse a la reunión de Teams) para unirse a la reunión de Teams desde dentro de la aplicación de Communication Services.
