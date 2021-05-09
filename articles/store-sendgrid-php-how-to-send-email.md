---
title: Uso del servicio de correo electrónico SendGrid (PHP) | Microsoft Docs
description: Obtenga información acerca de cómo enviar correo electrónico con el servicio de correo electrónico SendGrid en Azure. Ejemplos de código escritos en PHP.
documentationcenter: php
services: ''
manager: sendgrid
editor: mollybos
author: thinkingserious
ms.assetid: 51a9928a-4c9e-4b0a-aca8-9a408aeb6f47
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 10/30/2014
ms.author: erikre
ms.reviewer: elmer.thomas@sendgrid.com; erika.berkland@sendgrid.com; vibhork; matt.bernier@sendgrid.com
ms.openlocfilehash: ed13dd922595a051db77426281f062efd5176f57
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166735"
---
# <a name="how-to-use-the-sendgrid-email-service-from-php"></a>Uso del servicio de correo electrónico SendGrid desde PHP

Esta guía describe cómo realizar tareas comunes de programación con el servicio de correo electrónico SendGrid en Azure. Los ejemplos están escritos en PHP.
Entre los escenarios descritos se incluyen **creación de correo electrónico**, **envío de correo electrónico** e **incorporación de datos adjuntos**. Para obtener más información sobre SendGrid y el envío de correo electrónico, consulte la sección [Pasos siguientes](#next-steps) .

## <a name="what-is-the-sendgrid-email-service"></a>¿Qué es el servicio de correo electrónico SendGrid?

SendGrid es un [servicio de correo electrónico basado en la nube] que ofrece un sistema confiable de [entrega de correo electrónico transaccional], escalabilidad y análisis en tiempo real junto, con API flexibles que facilitan la integración personalizada. Entre los escenarios de uso de SendGrid comunes se incluyen:

* Envío automático de recibos a los clientes
* Administración de listas de distribución para enviar a los clientes prospectos electrónicos y ofertas especiales cada mes
* Recopilación de diversas métricas en tiempo real, como las direcciones de correo electrónico bloqueadas y la capacidad de respuesta del cliente.
* Generación de informes para ayudar a identificar tendencias
* Reenvío de consultas de los clientes
* Envío de notificaciones de correo electrónico desde su aplicación

Para más información, consulte [https://sendgrid.com][https://sendgrid.com].

## <a name="create-a-sendgrid-account"></a>Creación de una cuenta de SendGrid

[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="using-sendgrid-from-your-php-application"></a>Uso de SendGrid desde la aplicación PHP

El uso de SendGrid en una aplicación PHP de Azure no requiere una configuración ni una codificación especiales. Puesto que SendGrid es un servicio, se accede a él exactamente igual desde una aplicación en la nube que desde una aplicación local.

## <a name="how-to-send-an-email"></a>Envío de un correo electrónico

Puede enviar correo electrónico usando SMTP o la API web que proporciona SendGrid.

### <a name="smtp-api"></a>API SMTP

Para enviar correo electrónico mediante la API SMTP de SendGrid, use *Swift Mailer*, una biblioteca basada en componentes para el envío de correo electrónico desde aplicaciones PHP. Puede descargar la biblioteca de [Swift Mailer](https://swiftmailer.symfony.com/) v5.3.0 (use [Composer] para instalar Swift Mailer). El envío de correo electrónico con la biblioteca implica crear instancias de las clases `Swift\_SmtpTransport`, `Swift\_Mailer` y `Swift\_Message`, establecer las propiedades correspondientes y llamar al método `Swift\_Mailer::send`.

```php
<?php
 include_once "vendor/autoload.php";
 /*
  * Create the body of the message (a plain-text and an HTML version).
  * $text is your plain-text email
  * $html is your html version of the email
  * If the receiver is able to view html emails then only the html
  * email will be displayed
  */
 $text = "Hi!\nHow are you?\n";
 $html = "<html>
       <head></head>
       <body>
           <p>Hi!<br>
               How are you?<br>
           </p>
       </body>
       </html>";
 // This is your From email address
 $from = array('someone@example.com' => 'Name To Appear');
 // Email recipients
 $to = array(
       'john@contoso.com'=>'Destination 1 Name',
       'anna@contoso.com'=>'Destination 2 Name'
 );
 // Email subject
 $subject = 'Example PHP Email';

 // Login credentials
 $username = 'yoursendgridusername';
 $password = 'yourpassword';

 // Setup Swift mailer parameters
 $transport = Swift_SmtpTransport::newInstance('smtp.sendgrid.net', 587);
 $transport->setUsername($username);
 $transport->setPassword($password);
 $swift = Swift_Mailer::newInstance($transport);

 // Create a message (subject)
 $message = new Swift_Message($subject);

 // attach the body of the email
 $message->setFrom($from);
 $message->setBody($html, 'text/html');
 $message->setTo($to);
 $message->addPart($text, 'text/plain');

 // send message
 if ($recipients = $swift->send($message, $failures))
 {
     // This will let us know how many users received this message
     echo 'Message sent out to '.$recipients.' users';
 }
 // something went wrong =(
 else
 {
     echo "Something went wrong - ";
     print_r($failures);
 }
```

### <a name="web-api"></a>API Web

Use la [función curl][curl function] de PHP para enviar correo electrónico usando la API web de SendGrid.

```php
<?php

 $url = 'https://api.sendgrid.com/';
 $user = 'USERNAME';
 $pass = 'PASSWORD';

 $params = array(
      'api_user' => $user,
      'api_key' => $pass,
      'to' => 'john@contoso.com',
      'subject' => 'testing from curl',
      'html' => 'testing body',
      'text' => 'testing body',
      'from' => 'anna@contoso.com',
   );

 $request = $url.'api/mail.send.json';

 // Generate curl request
 $session = curl_init($request);

 // Tell curl to use HTTP POST
 curl_setopt ($session, CURLOPT_POST, true);

 // Tell curl that this is the body of the POST
 curl_setopt ($session, CURLOPT_POSTFIELDS, $params);

 // Tell curl not to return headers, but do return the response
 curl_setopt($session, CURLOPT_HEADER, false);
 curl_setopt($session, CURLOPT_RETURNTRANSFER, true);

 // obtain response
 $response = curl_exec($session);
 curl_close($session);

 // print everything out
 print_r($response);
```

La API web de SendGrid es muy similar a la API de REST, aunque no es realmente una API de RESTful porque, en la mayoría de las llamadas, se pueden usar los verbos GET y POST indistintamente.

## <a name="how-to-add-an-attachment"></a>Adición de un correo electrónico

### <a name="smtp-api"></a>API SMTP

El envío de datos adjuntos con la API SMTP implica una línea adicional de código al script de ejemplo para enviar un mensaje de correo electrónico con Swift Mailer.

```php
<?php
 include_once "vendor/autoload.php";
 /*
  * Create the body of the message (a plain-text and an HTML version).
  * $text is your plain-text email
  * $html is your html version of the email
  * If the receiver is able to view html emails then only the html
  * email will be displayed
  */
 $text = "Hi!\nHow are you?\n";
  $html = "<html>
      <head></head>
      <body>
         <p>Hi!<br>
            How are you?<br>
         </p>
      </body>
      </html>";

 // This is your From email address
 $from = array('someone@example.com' => 'Name To Appear');

 // Email recipients
 $to = array(
      'john@contoso.com'=>'Destination 1 Name',
      'anna@contoso.com'=>'Destination 2 Name'
 );
 // Email subject
 $subject = 'Example PHP Email';

 // Login credentials
 $username = 'yoursendgridusername';
 $password = 'yourpassword';

 // Setup Swift mailer parameters
 $transport = Swift_SmtpTransport::newInstance('smtp.sendgrid.net', 587);
 $transport->setUsername($username);
 $transport->setPassword($password);
 $swift = Swift_Mailer::newInstance($transport);

 // Create a message (subject)
 $message = new Swift_Message($subject);

 // attach the body of the email
 $message->setFrom($from);
 $message->setBody($html, 'text/html');
 $message->setTo($to);
 $message->addPart($text, 'text/plain');
 $message->attach(Swift_Attachment::fromPath("path\to\file")->setFileName("file_name"));

 // send message
 if ($recipients = $swift->send($message, $failures))
 {
      // This will let us know how many users received this message
      echo 'Message sent out to '.$recipients.' users';
 }
 // something went wrong =(
 else
 {
      echo "Something went wrong - ";
      print_r($failures);
 }
```

La línea adicional de código es la siguiente:

```php
 $message->attach(Swift_Attachment::fromPath("path\to\file")->setFileName('file_name'));
```

Esta línea de código llama al método attach en el objeto `Swift\_Message` y usa el método estático `fromPath` en la clase `Swift\_Attachment` para obtener y adjuntar un archivo a un mensaje.

### <a name="web-api"></a>API Web

El envío de datos adjuntos mediante la API web es muy similar al envío de un mensaje de correo electrónico con la API web. No obstante, tenga en cuenta que, en el ejemplo siguiente, el parámetro array debe contener este elemento:

```php
    'files['.$fileName.']' => '@'.$filePath.'/'.$fileName
```

#### <a name="example"></a>Ejemplo

```php
<?php

 $url = 'https://api.sendgrid.com/';
 $user = 'USERNAME';
 $pass = 'PASSWORD';

 $fileName = 'myfile';
 $filePath = dirname(__FILE__);

 $params = array(
     'api_user' => $user,
     'api_key' => $pass,
     'to' =>'john@contoso.com',
     'subject' => 'test of file sends',
     'html' => '<p> the HTML </p>',
     'text' => 'the plain text',
     'from' => 'anna@contoso.com',
     'files['.$fileName.']' => '@'.$filePath.'/'.$fileName
 );

 print_r($params);

 $request = $url.'api/mail.send.json';

 // Generate curl request
 $session = curl_init($request);

 // Tell curl to use HTTP POST
 curl_setopt ($session, CURLOPT_POST, true);

 // Tell curl that this is the body of the POST
 curl_setopt ($session, CURLOPT_POSTFIELDS, $params);

 // Tell curl not to return headers, but do return the response
 curl_setopt($session, CURLOPT_HEADER, false);
 curl_setopt($session, CURLOPT_RETURNTRANSFER, true);

 // obtain response
 $response = curl_exec($session);
 curl_close($session);

 // print everything out
 print_r($response);
```

## <a name="how-to-use-filters-to-enable-footers-tracking-and-analytics"></a>Uso de filtros para habilitar pies de página, seguimiento y análisis

SendGrid proporciona funcionalidad de correo electrónico adicional mediante el uso de *filtros*. Estas configuraciones se pueden agregar a un mensaje de correo electrónico para permitir una funcionalidad específica, como habilitar el seguimiento de clics, el análisis de Google, el seguimiento de las suscripciones, etc.

Los filtros se pueden aplicar a un mensaje usando la propiedad filters. Cada filtro se especifica mediante un hash que contiene la configuración específica del filtro. En el siguiente ejemplo se habilita el filtro de pie de página y se especifica un mensaje de texto que se anexará al final del mensaje de correo electrónico. En este ejemplo usaremos biblioteca [sendgrid-php].

Use [Compositor] para instalar la biblioteca:

```bash
php composer.phar require sendgrid/sendgrid 2.1.1
```

### <a name="example"></a>Ejemplo

```php
<?php
 /*
  * This example is used for sendgrid-php V2.1.1 (https://github.com/sendgrid/sendgrid-php/tree/v2.1.1)
  */
 include "vendor/autoload.php";

 $email = new SendGrid\Email();
 // The list of addresses this message will be sent to
 // [This list is used for sending multiple emails using just ONE request to SendGrid]
 $toList = array('john@contoso.com', 'anna@contoso.com');

 // Specify the names of the recipients
 $nameList = array('Name 1', 'Name 2');

 // Used as an example of variable substitution
 $timeList = array('4 PM', '5 PM');

 // Set all of the above variables
 $email->setTos($toList);
 $email->addSubstitution('-name-', $nameList);
 $email->addSubstitution('-time-', $timeList);

 // Specify that this is an initial contact message
 $email->addCategory("initial");

 // You can optionally setup individual filters here, in this example, we have
 // enabled the footer filter
 $email->addFilter('footer', 'enable', 1);
 $email->addFilter('footer', "text/plain", "Thank you for your business");
 $email->addFilter('footer', "text/html", "Thank you for your business");

 // The subject of your email
 $subject = 'Example SendGrid Email';

 // Where is this message coming from. For example, this message can be from
 // support@yourcompany.com, info@yourcompany.com
 $from = 'someone@example.com';

 // If you do not specify a sender list above, you can specify the user here. If
 // a sender list IS specified above, this email address becomes irrelevant.
 $to = 'john@contoso.com';

 # Create the body of the message (a plain-text and an HTML version).
 # text is your plain-text email
 # html is your html version of the email
 # if the receiver is able to view html emails then only the html
 # email will be displayed

 /*
  * Note the variable substitution here =)
  */
 $text = "
 Hello -name-,
 Thank you for your interest in our products. We have set up an appointment to call you at -time- EST to discuss your needs in more detail.
 Regards,
 Fred";

 $html = "
 <html>
 <head></head>
 <body>
 <p>Hello -name-,<br>
 Thank you for your interest in our products. We have set up an appointment
 to call you at -time- EST to discuss your needs in more detail.

 Regards,

 Fred<br>
 </p>
 </body>
 </html>";

 // set subject
 $email->setSubject($subject);

 // attach the body of the email
 $email->setFrom($from);
 $email->setHtml($html);
 $email->addTo($to);
 $email->setText($text);

 // Your SendGrid account credentials
 $username = 'sendgridusername@yourdomain.com';
 $password = 'example';

 // Create SendGrid object
 $sendgrid = new SendGrid($username, $password);

 // send message
 $response = $sendgrid->send($email);

 print_r($response);
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que conoce los fundamentos del servicio de correo electrónico SendGrid, siga estos vínculos para obtener más información:

* Documentación de SendGrid: <https://sendgrid.com/docs>
* Biblioteca PHP de SendGrid: <https://github.com/sendgrid/sendgrid-php>
* Oferta especial de SendGrid para clientes de Azure: <https://sendgrid.com/windowsazure.html>

Para obtener más información, consulte también el [Centro para desarrolladores de PHP](https://azure.microsoft.com/develop/php/).

[https://sendgrid.com]: https://sendgrid.com
[https://sendgrid.com/transactional-email/pricing]: https://sendgrid.com/transactional-email/pricing
[special offer]: https://www.sendgrid.com/windowsazure.html
[Packaging and Deploying PHP Applications for Azure]: https://msdn.microsoft.com/library/windowsazure/hh674499(v=VS.103).aspx
[http://swiftmailer.org/download]: http://swiftmailer.org/download
[curl function]: https://php.net/curl
[servicio de correo electrónico basado en la nube]: https://sendgrid.com/email-solutions
[entrega de correo electrónico transaccional]: https://sendgrid.com/transactional-email
[sendgrid-php]: https://github.com/sendgrid/sendgrid-php/tree/v2.1.1
[Compositor]: https://getcomposer.org/download/
