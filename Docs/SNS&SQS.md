# Configurar SNS y SQS en AWS

Esta guía cubre la configuración de un tema SNS, la creación de una cola SQS, la suscripción de la cola SQS al tema SNS, y las políticas necesarias para que todo funcione correctamente.

## Pasos para la configuracion

**1. Crear una Cola SQS**
**2. Crear un Tema SNS**
**3. Suscribir la Cola SQS al Tema SNS**
**4. Configurar Política de Acceso para SQS**
**5. Publicar un Mensaje de Prueba en SNS y verificar que llega a SQS**
**6. Conceder a los usuarios permisos**


## Paso 1: Crear una Cola SQS
_Una cola SQS es donde se almacenan los mensajes que SNS enviará._

1. Ve a la sección de **SQS** en la consola de AWS.
2. Haz clic en **Create queue**.
3. Selecciona el tipo de cola **Standard**.
4. Asigna un nombre único a la cola, por ejemplo, `MyQueue`.
5. Deja la configuración predeterminada o ajusta según tus necesidades.
6. Haz clic en **Create Queue**.

## Paso 2: Crear un Tema SNS
_Un tema SNS es un recurso que permite enviar mensajes a múltiples suscriptores (en este caso, a una cola SQS)._

1. Ve a la sección de **SNS** en la consola de AWS.
2. En el panel izquierdo, selecciona **Topics**.
3. Haz clic en **Create topic**.
4. Selecciona el tipo de tema como **Standard**.
5. Asigna un nombre único al tema, por ejemplo, `MyTopic`.
6. Deja la configuración predeterminada o ajusta según tus necesidades.
7. Haz clic en **Create topic**.

## Paso 3: Suscribir la Cola SQS al Tema SNS
_Ahora, necesitas suscribir la cola SQS al tema SNS para que los mensajes publicados en SNS sean entregados a SQS._

 1. Haz clic en **Create subscription**.
 2. En **Protocol**, selecciona **Amazon SQS**.
 3. En **Endpoint**, selecciona el _ARN_ de la cola _SQS_ que desea suscribir al tema _SNS_
 4. Haz clic en **Create subscription**.

> **Nota:** Al ser el protocolo por SQS este suscripcion se confirma automaticamente.

## Paso 4: Configurar Política de Acceso para SQS
_Para que SNS pueda enviar mensajes a tu SQS, necesitas configurar una política de acceso en la cola SQS. Esta política permite que SNS interactúe con la cola._

1. Ir a la consola de SQS.
2. Selecciona la cola que creaste (`MyQueue`).
3. En la sección **Access Policy**, haz clic en **Edit**.
4. Añade la siguiente política JSON (asegúrate de sustituir los ARN por los correspondientes de tu SNS y SQS):
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSNSToSendMessages",
      "Effect": "Allow",
      "Principal": {
        "Service": "sns.amazonaws.com"
      },
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:us-east-1:123456789012:MyQueue",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:sns:us-east-1:123456789012:MyTopic"
        }
      }
    }
  ]
}
```
* **Version:** Esta es una versión más antigua de la política de AWS. Si no tienes una razón específica para usar esta versión, te recomendaría cambiar a la versión más reciente.
* **Sid:** Esto es un identificador opcional para la declaración, lo cual puede ser útil para referenciar la política.
* **Principal:** El principal debe ser `sns.amazonaws.com`, que es el servicio SNS.
* **Action:** Se otorga permiso para la acción `SQS:SendMessage`, que es la acción necesaria para que SNS pueda enviar mensajes a SQS.
* **Resource:** Este campo debe contener el ARN de tu cola SQS.
* **Condition**: La condición `aws:SourceArn` asegura que solo los mensajes provenientes del tema SNS específico puedan ser enviados a la cola SQS.

5.Haz clic en **Save Changes**.

## Paso 5: Publicar un Mensaje en SNS y verificar que llega a SQS
_Ahora que todo está configurado, puedes probar la integración publicando un mensaje en el tema SNS y verificando que la cola SQS lo reciba._

1. **Publicar un Mensaje**
    * Ir a la consola de **SNS**.
    * Selecciona el tema, para este ejemplo `MyTopic`.
    * Haz clic en **Publish message**.
    * Escribe un mensaje de prueba, por ejemplo, `Mensaje de prueba para SQS.`
    * Haz clic en `Publish message`.
2. **Verificar que el Mensaje Llega**
    * Ir a la consola de **SQS**
    * Selecciona la cola para este ejemplo `MyQueue`.
    * Haz clic en **Queue Actions** y selecciona **View/Delete Messages**.
    * Si todo está configurado correctamente, deberías ver el mensaje de prueba en la cola.

## Paso 6: Conceder a los usuarios permiso

#### Paso 6.1: Crear Usuario
1. En el panel izquierdo de IAM, haz clic en **Users**.
2. En el campo **User name**, escribe el nombre del nuevo usuario (por ejemplo, `MyUser`).
3. Activa la opcion **Provide user access to the AWS Management Console**
4. Seleciona **I want to create an IAM user**
5. Configurar las opciones de contraseña.
    * Si habilitas el acceso a la consola de AWS, puedes elegir entre las siguientes opciones para la contraseña:
        * **Autogenerated password:** AWS genera una contraseña aleatoria para el usuario.
        * **Custom password:** Puedes crear tu propia contraseña para el usuario.
        * **Users must create a new password at next sign-in - Recommended:** Marca esta opción si deseas que el usuario cambie su contraseña la primera vez que inicie sesión.
6. Luego, haz click en **Next**.
7. No agregaremos aun los permisos asi que hacemos click en **Next**.
8. Haz click en **Create user**
9. Guardar las credenciales.

#### Paso 6.2: Crear permiso para el SNS

1. Selecionamos el usuario creado anteriormente.
2. Hacemos click en **Add permissions**
3. Seleciona la opcion **Create inline policy**.
4. En el apartdado de **Service** busca SNS.
5. Filtra el Actions **Publish** y selecionalo.
6. En el apartado de ***Resources*** que se abrio haz clien en **Add ARNs**
7. En el campo **Resource ARN** pequa el ARN que Tema de SNS, para este ejemplo `arn:aws:sns:us-east-1:123456789012:MyTopic`
8. Haz click en **Add ARNs**
9. Haz click en **Next**
10. Agrega un nombre para el permiso, por ejemplo, `MyPolicyPublishSNSMyTopic`.
11. Haz click en **Create policy**

#### Paso 6.3: Crear permiso para el SQS

1. Selecionamos el usuario creado anteriormente.
2. Hacemos click en **Add permissions**
3. Seleciona la opcion **Create inline policy**.
4. En el apartdado de **Service** busca SQS.
5. Filtra el Actions **ReceiveMessage** y **DeleteMessage** selecionarlos.
6. En el apartado de ***Resources*** que se abrio haz clien en **Add ARNs**
7. En el campo **Resource ARN** pequa el ARN que Tema de SNS, para este ejemplo `arn:aws:sqs:us-east-1:123456789012:MyQueue`
8. Haz click en **Add ARNs**
9. Haz click en **Next**
10. Agrega un nombre para el permiso, por ejemplo, `MyPolicyReceiveAndDeleteMessageSQSMyQueue`.
11. Haz click en **Create policy**
