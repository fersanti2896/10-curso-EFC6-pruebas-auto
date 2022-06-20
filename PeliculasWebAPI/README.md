# Resumen de la sección 10: Pruebas Automáticas
___

1. __Pruebas Unitarias.__
2. __Introducción a los Mocks.__
3. __Configurando el Proveedor en Memoria.__

#### Pruebas Unitarias

Una prueba unitaria se encarga de probar una unidad de trabajo, tienen la particularidad de ser muy rápidas. 

En la misma solución, se crea un nuevo proyecto que será de pruebas y dentro del mismo se agrega la referencia al proyecto al cual se hará las pruebas. 

![ReferenciaPruebas](/PeliculasWebAPI/images/ReferenciaProyectoPruebas.PNG)

Se crea un archivo de pruebas `ServicioUsuarioPruebas.cs`, el cual tendrá los tres pasos para realizar la prueba hacia `UsuarioService.cs` del método `ObtenerUsuarioId()`.

![ServicioUsuarioPruebas](/PeliculasWebAPI/images/ServicioUsuarioPruebas.png)

Al ejecutar las pruebas, estas son éxitosas. 

![PruebasExitosasServicioUsario](/PeliculasWebAPI/images/PruebasServicioUsuarioExitosa.PNG)

#### Introducción a los Mocks

Es un objeto el cual intenta suplantar una dependencia de una clase, es útil si se quiere usar dependencia como _Web Services_. 

Nos permite personalizar el comportamiento de dependencias de las clases que queremos ejecutar. 

Desde la prueba unitarias podemos probar una clase ignorando sus dependencias, aunque no es obligatoria usar mocks. 

Al crear una prueba para `ActualizadorObservableCollectionService.cs` se crea dos _Mocks_, uno que hará la función del mapeador y otra del id. 

Se crea el archivo de prueba con nombre `ActualizadorObservableCollectionPruebas.cs` con tres pruebas a evaluar. 

![ActualizadorObservableCollectionPruebas](/PeliculasWebAPI/images/ActualizadorObservableCollectionPruebas.png)

Al ejecutar las pruebas, todas son éxitosas. 

![ResultadoActualizadoObservable](/PeliculasWebAPI/images/ActualizadorObservableCollectionPruebas%20Resultado.PNG)


#### Configurando el Proveedor en Memoria