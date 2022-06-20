# Resumen de la sección 10: Pruebas Automáticas
___

1. __Pruebas Unitarias.__
2. __Introducción a los Mocks.__

#### Pruebas Unitarias

Una prueba unitaria se encarga de probar una unidad de trabajo, tienen la particularidad de ser muy rápidas. 

En la misma solución, se crea un nuevo proyecto que será de pruebas y dentro del mismo se agrega la referencia al proyecto al cual se hará las pruebas. 

![ReferenciaPruebas](/PeliculasWebAPI/images/ReferenciaProyectoPruebas.PNG)

Se crea un archivo de pruebas `ServicioUsuarioPruebas.cs`, el cual tendrá los tres pasos para realizar la prueba hacia `UsuarioService.cs` del método `ObtenerUsuarioId()`.

![ServicioUsuarioPruebas](/PeliculasWebAPI/images/ServicioUsuarioPruebas.png)

Al ejecutar las pruebas, estas son éxitosas. 

![PruebasExitosasServicioUsario](/PeliculasWebAPI/images/PruebasServicioUsuarioExitosa.PNG)

#### Introducción a los Mocks

