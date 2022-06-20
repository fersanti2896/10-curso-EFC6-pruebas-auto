# Resumen de la sección 10: Pruebas Automáticas
___

1. __Pruebas Unitarias.__
2. __Introducción a los Mocks.__
3. __Configurando el Proveedor en Memoria.__
4. __Prueba Unitaria en Entity Framework.__

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

Para hacer pruebas que incluyan al `ApplicationDbContext` de `Entity Framework` , el cual se probará los comportamientos (responsabilidades) y no sólo una función. 

Para hacerlo usamos el proveedor en memoria. Instalamos el paquete `Microsoft.EntityFrameworkCore.InMemory` y debemos instancia al menos 2 `DbContext`, uno para para cargar la data y el otro el que se va inyectar. 

Cada prueba automática debe tener su propia base de datos esto con el fin de que cada prueba no afecte a la otra. 

`NOTA: El proveedor de memoria no trabaja con tablas temporales, por el cual se deben hacer cambios en la solución del proyecto. Además de evitar usar Seeding, esto porque se registrar los datos en las tablas temporales`.

Instanciamos el `DbContext` en nuestro proyecto. 

![BasePruebas](/PeliculasWebAPI/images/BasePruebas.png)

Y en nuestro `ApplicationDBContext.cs` como tenemos data de tipo `Seeding` debemos evitar que se sobreescriban la data en las Bases de Datos Temporales.

![ApplicationDbContext](/PeliculasWebAPI/images/ApplicationDbContext.png)

#### Prueba Unitaria en Entity Framework

