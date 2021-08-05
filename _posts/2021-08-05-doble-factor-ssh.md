### ¿Qué es el Doble Factor de Autenticación?
El doble factor de autenticación se refiere a una medida extra de seguridad para el inicio de sesión en distintas cuentas, frecuentemente se require de un código obtenido utilizando una aplicación o un mensaje SMS y además la contraseña preestablecida para el usuario que está inciando sesión.
La mayoría de servicios que se encuentran en Internet ya brindan la posibilidad de implementar esta medida de seguridad ante el crecimiento de secuestros de cuentas y suplantación de identidad.
Su uso no está obligado para todas nuestras cuentas, sin embargo, es altamente recomendable utilizarlo en las que consideremos más valisas, por ejemplo, correo electrónico, red social principal, servicios de almacenamiento, etc.
Como profesional de Tecnologías de la información, el uso de esta medida no está limitado a cuentas de servicios públicos, sino que es conveniente aplicarlo a servicios que nosotros administramos, por ejemplo algún servidor SSH, principalmente si se encuentra expuesto a Internet.
### ¿Cómo instalo este servicio en mi servidor?
El procedimiento que a continuación se mostrará se realizó sobre un servidor Debian 10 (Instalación mínima), sin embargo, se puede realizar sobre cualquier otra distribución GNU/Linux.
Se debe ejecutar la siguiente línea para instalar los componentes necesarios:
```
sudo apt install libpam-google-authenticator
```
![[installLGA.png]]
Una vez que se ha instalado se tiene que ejecutar **google-authenticator** para realizar las configuraciones iniciales.
Con esto se va solicitar asignar la clave secreta y preguntará si se quiere utilizar los tokens basados en tiempo, lo que se debe responder que **sí**. Al dar esta respuesta, aparecerá el código QR que se debe escanear en la aplicación del teléfono, además aparecerá la llave secreta y los códigos de emergencia.
![[qrGA.png]]
Aparecerásn las siguentes preguntas:
- ¿Quieres actualizar el archivo de configuración? **Sí**
- ¿Quieres dehabilitar el uso múltiple del mismo código? **Sí**
- Configuración para evitar problemas de sincronía: **Sí**
- ¿Limitar el límite de intentos de sesión a 3? **Sí**
![[questionsGA.png]]
Los datos de claves secretas y códigos de emergencia se deben almacenar en un lugar seguro para uso posterior.
### Configuración de SSH para usar con Google Authenticator
La siguiente configuración se debe realizar en el servidor para que funcione correctamente con las librerías implementadas.
Abrir el archivo de configuración de SSH:
```
sudo vim /etc/ssh/sshd_config
```
Se deben colocar las siguientes líneas a **yes**:
```
...
ChallengeResponseAuthentication yes
...
UsePAM yes
...

```
Guardar los cambios realizados y reiniciar el servicio SSH.
```
sudo systemctl restart sshd
```
Para hacer obligatorio el uso del código 
de la aplicación se debe configurar el archivo de las reglas PAM
```
sudo vim /etc/pam.d/sshd
```
Se debe cambiar a **yes** la siguiente línea:
```
ChallengeResponseAuthentication yes
```
También para habilitar la autenticación por medio de una única contraseña se deben agregar las siguientes dos líneas debajo de **@include common auth**:
```
#One-time password authentication via Google Authenticator
auth required pam_google_authenticator.so
```
Con estos cambios hechos, se guarda y cierra el archivo. A partir de ese momento se solicitará el código generado por la aplicación al iniciar sesión.
Ahora es momento de probar el acceso:
![[loginGA.png]]
### Desinstalación
En caso de que se requiera quitar esta configuración, ya sea porque no se cuenta con la aplicación o por algún otro motivo, basta con revertir el cambio en el archivo PAM. Esto se hace comentando la siguiente línea:
```
# auth required pam_google_authenticator.so
```
Al guardar y salir de archivo se tiene que reiniciar el servicio usando la siguiente línea:
```
sudo systemctl restart sshd
```
Una vez hechos estos cambios en el inicio de sesión sólo se solicitará la contraseña de los usuarios.
Es necesario tener en cuanta que para hacer este cambio se debe contar con un acceso físico al servidor o acceso a través de la consola virtual en el hipervisor. 
