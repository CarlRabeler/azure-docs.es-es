## <a name="add-the-applications-registration-information-to-your-app"></a>Adición de información de registro de la aplicación a su aplicación

En este paso, necesita configurar la dirección URL de redireccionamiento de la información de registro de su aplicación y luego agregar el identificador de la aplicación a la aplicación JavaScript SPA.

### <a name="configure-redirect-url"></a>Configuración de la URL de redireccionamiento

Configure el campo `Redirect URL` con la dirección URL de la página index.html basada en el servidor web y luego haga clic en *Actualizar*.


> #### <a name="visual-studio-instructions-for-obtaining-redirect-url"></a>Instrucciones en Visual Studio para obtener la dirección URL de redireccionamiento
> Para obtener la dirección URL de redireccionamiento:
> 1.    En el *Explorador de soluciones*, seleccione el proyecto y fíjese en la ventana `Properties` (si no ve una ventana de propiedades, presione `F4`)
> 2.    Copie el valor de `URL` en el Portapapeles:<br/> ![Propiedades del proyecto](media/active-directory-develop-guidedsetup-javascriptspa-configure/vs-project-properties-screenshot.png)<br />
> 3.    Pegue el valor como `Redirect URL` en la parte superior de esta página y, luego, haga clic en `Update`

<p/>

> #### <a name="setting-redirect-url-for-python"></a>Configuración de la dirección URL de redireccionamiento para Python
> Para Python, puede establecer el puerto de servidor web a través de una línea de comandos. Esta configuración paso a paso usa el puerto 8080 como referencia, pero puede usar cualquier otro puerto que esté disponible. En cualquier caso, use las instrucciones siguientes para configurar una dirección URL de redireccionamiento en la información de registro de aplicación:<br/>
> Establezca `http://localhost:8080/` como `Redirect URL` en la parte superior de esta página, o bien use `http://localhost:[port]/` si usa un puerto TCP personalizado (donde *[puerto]* es el número de puerto TCP personalizado) y, luego, haga clic en "Update" (Actualizar)

### <a name="configure-your-javascript-spa-application"></a>Configuración de la aplicación JavaScript SPA

1.  Cree un archivo denominado `msalconfig.js` que contenga la información de registro de la aplicación. Si se usa Visual Studio, seleccione el proyecto (carpeta raíz del proyecto), haga clic con el botón derecho y seleccione: `Add` > `New Item` > `JavaScript File`. Asígnele el nombre `msalconfig.js`.
2.  Agregue el siguiente código al archivo `msalconfig.js`:

```javascript
var msalconfig = {
    clientID: "[Enter the application Id here]",
    redirectUri: location.origin
};
``` 
