# Resumen de la sección 9: Escenarios Avanzados
___

1. __Funciones Escalares.__
2. __Funciones con Valores de Tabla.__
3. __Columnas Calculadas.__
4. __Secuencias.__
5. __Conflicto de Concurrencia por Campo.__
6. __Conflictos de Concurrencia por Fila.__
7. __Manejo de Conflictos de Concurrencia.__
8. __Conflictos de Concurrencia con el Modelo Desconectado.__
9. __Tablas Temporales.__
10. __CUD en la Tabla Temporal.__
11. __Consultando la Tabla Temporal e Histórica.__
12. __TemporalAsOf.__
13. __TemporalFromTo.__
14. __TemporalContainedIn.__
15. __TemporalBetween.__
16. __Restuarando un Registro Borrado.__
17. __Personalizando la Tabla Temporal e Histórica.__

#### Funciones Escalares

Una función definida por el usuario es una función en nuestra Base de Datos el cual podemos usar para encapsular funcionalidad, pero solo para consultas no se puede modificar la Base de Datos. 

El resultante de esta función puede ser un escalar o un conjunto de resultado. 

Para este ejemplo, creamos dos funciones escalares, la primera para calcular la suma de los costos de las ordenes de una factura y la segunda el promedio de dichos costos. 

Primero creamos una data en nuestro `SeedingFacturas.cs`. 

![seedingFacturas](/PeliculasWebAPI/images/seedingFacturas.png)

Creamos la migración para que empuje los cambios hacia la Base de Datos, de la misma forma, creamos una segunda migración limpia el cual vamos a definir Funciones SQL. 

![migracionFuncion](/PeliculasWebAPI/images/migracionFuncionesDefinidas.png)

La primera forma de definir una función por usuario, es a través de nuestro DbContext, en este caso en `ApplicationDbContext.cs`.

![primerFormaDbContext](/PeliculasWebAPI/images/facturaSumaDbContext.png)

La segunda forma es a través de una clase que va ser definida de igual forma en el DbContext con la línea: 

    /* Registro de Clase Auxiliar de Funciones Definidas por el Usuario */
    Escalares.RegistrarFunciones(modelBuilder);

Nuestra entidad `Escalares.cs`.

![escalares](/PeliculasWebAPI/images/escalares.png)

Finalmente vamos a crear un `endpoint` de tipo `GET` en nuestro `FacturaController.cs` el cual implementa las funciones que hicimos. 

![facturaGet](/PeliculasWebAPI/images/FacturaController%20FuncionesDefinidas.png)

Probamos nuestro `endpoint` y tenemos como resultado un status `200`.

![result](/PeliculasWebAPI/images/funcionesDefinidas%20Result.PNG)

Desde nuestra terminal vemos el query SQL que se ejecutó.

![resultCmd](/PeliculasWebAPI/images/funcionesDefinidas%20Result%20cmd.PNG)

#### Funciones con Valores de Tabla

Son definidas por el usuario que envés de devolver un escalar retorna un conjunto de resultados, es decir, filas y columnas

Vamos a definir una vista pero como una función definia por el usuario, creamos una migración. 

![funcionVTmigration](/PeliculasWebAPI/images/funcionesValoresTabla%20migration.png)

En nuestro `ApplicationDBContext.cs` definimos las siguiente código para usar la función que creamos. 

        /* Para usar la función definida por el usuario */
        modelBuilder.Entity<PeliculaConteos>()
                    .HasNoKey()
                    .ToTable(name: null);

        modelBuilder.HasDbFunction(() => PeliculaConteo(0));

Ademá de definir una función que devolverá lo que queremos. 

![funcionDbContext](/PeliculasWebAPI/images/funcionesValoresTabla%20dbContext.png)

En nuestro `PeliculasController.cs` definimos un nuevo `endpoint` de tipo `GET` que nos devolverá la función que definimos. 

![peliculaControllFVT](/PeliculasWebAPI/images/peliculasConteo%20FuncionVT.png)

Al probar el `endpoint` obtenemos un status `200`.

![resultFVT](/PeliculasWebAPI/images/funcionesValoresTabla%20Result.PNG)

#### Columnas Calculadas

Nos permiten automatizar el llenado de las columnas con el resultado de alguna operación, existen dos tipos de columnas calculadas: 

- Las que guardan el valor final en la columna. 
- Las que no la hacen, calcula su valor en base a su consulta, sin embargo esta operación puede consumir tiempo de ejecución. 

Por ejemplo, si queremos agregar una columna de Total y Cantidad a nuestra entidad `FacturaDetalle.cs`. 

![facturaDetalle](/PeliculasWebAPI/images/FacturaDetalle.png)

Al hacer su configuración, usamos el método `HasComputedColumnSql()` donde guardaremos el valor en la columna calculada. 

![FacturaDetalleConfig](/PeliculasWebAPI/images/FacturaDetalleConfig.png)

Hacemos la migración. 

![migracionFacturaDetalle](/PeliculasWebAPI/images/TotalCalculadoMigracion.png)

Creamos un nuevo `endpoint` en nuestro `FacturaController.cs` de tipo `GET` el cual nos traerá la información de un registro con la columna que agregamos. 

![facturaController](/PeliculasWebAPI/images/FacturaDetalleController.png)

Al probar, obtenemos un status `200` con el resultado esperado. 

![columnCalcResult](/PeliculasWebAPI/images/Columnas%20Calculadas%20Result.PNG)

#### Secuencias

Si queremos agregar una columna de sencuencia que permita tener valores de tipo _id_ usamos la forma secuencia. 

Supongamos que queremos tener una secuencia de factura, modificamos nuestra `Factura.cs` donde agremos una nueva columna que tendrá la secuencia.

![facturaSencuencia](/PeliculasWebAPI/images/facturaSecuencia.png)

En nuestro `ApplicationDBContext.cs` agregamos la siguiente línea el cual va mapear la secuencia hacia el esquema. 

    /* Creamos la secuencia numero de factura */
    modelBuilder.HasSequence<int>("NumFactura", "factura");

Modificamos nuestro `FacturaConfig.cs` para agregar la línea:

    builder.Property(f => f.NumFactura)
            .HasDefaultValueSql("NEXT VALUE FOR factura.NumFactura");

El cual va asignar el siguiente valor para la columna _NumFactura_ de tipo secuencia. 

Hacemos nuestra migración y empujamos los cambios hacia la Base de Datos. 

![migracionSecuencia](/PeliculasWebAPI/images/migracionSecuencia.png)

Al actualizar nuestra Base de Datos, vemos reflejado los cambios. 

![facturaBD](/PeliculasWebAPI/images/NumFactura%20Result.PNG)

#### Conflicto de Concurrencia por Campo

Ocurren cuandos dos personas intentan realizar un cambio sobre un registro y por error el cambio de la segunda persona sobreescribe el cambio realizado por la primera persona. 

Esto se puede solucionar por campo o por fila, en el caso del campo, el manejo se hace a nivel de un único campo y no de todos. 

Por ejemplo, en nuestro `GenerosController.cs` creamos un `endpoint` que simulará un registro de concurrencia. 

![generosControllerConc](/PeliculasWebAPI/images/ConcurrenciaGenerosController.png)

Por lo que al principio, nuestra tabla _Generos_ de nuestra Base de Datos, se encuentra: 

![GeneroDB1](/PeliculasWebAPI/images/GeneroDB1.PNG)

Al probar los resultados, obtenemos un status `200`. 

![ConcurrenciaResu](/PeliculasWebAPI/images/Concurrencia%20Result.PNG)

Al verificar nuestra tabla de _Generos_ vemos que se realizó el cambió pero se sobreescribió el registro. 

![GenerosDB2](/PeliculasWebAPI/images/GeneroDB2.PNG)

Para evitar este error, si queremos hacer una verificación de concurrencia para el campo _Nombre_ de la entidad `Genero.cs` con la propiedad `[ConcurrencyCheck]`, el cual verificar que la data no haya sido actualiza por alguien más al mismo tiempo.

![GeneroConcurrencia](/PeliculasWebAPI/images/Genero%20Concurrencia.png)

Otra forma de validar que un campo no se actualice por otra persona al mismo tiempo, lo podemos configurar en el `API Fluent` de la entidad, con la propiedad `IsConcurrencyToken()`.

_Opcional en el API Fluent_

    /* Damos propiedades al campo Nombre de la clase Genero */
    builder.Property(prop => prop.Nombre)
           .HasMaxLength(150)
           .IsRequired()
           .IsConcurrencyToken();

Al probar de nuevo, recibimos un status `500`, porque ya hay un conflicto de que un registro se está intentando actualizar al mismo tiempo, solo ejecutará la actualiación de la primera persona que hizo dicha actualización. 

![ConcucrenciResult2](/PeliculasWebAPI/images/Concurrencia%20Result%202.PNG)

En nuestra tabla de Generos de nuestra Base de Datos, ya aparece el registro correcto. 

![generoDB3](/PeliculasWebAPI/images/GeneroDB3.PNG)

#### Conflictos de Concurrencia por Fila

En el anterior punto vimos el conflicto de concurrencia por un campo, aquí se hará si algunos de los campos de la fila están siendo actualizados al mismo tiempo.

Por ejemplo, en nuestra entidad `Factura.cs` no queremos que dos personas actualicen una misma factura al mismo tiempo usando la propiedad `[Timestamp]`. 

![facturaConcu](/PeliculasWebAPI/images/FacturaConcurrencia.png)

Aunque podemos hacer la misma configuración desde el `API Fluent` con el método `IsRowVersion()`.

_Opcional en el API Fluent_

    builder.Property(f => f.Version).IsRowVersion();

Aplicamos una migración para que los cambios se ejecuten en la Base de Datos. 

![facturaConcurrenciaMigracion](/PeliculasWebAPI/images/FacturaConcurrencia%20Migracion.png)

Creamos un `endpoint` que simulará la doble actualización de una factura al mismo en nuestro `FacturasController.cs`.

![facturasControllerConcu](/PeliculasWebAPI/images/FacturaController%20Concurrencia.png)

Verificamos de manera inicial nuestra tabla de Facturas en nuestra Base de Datos. 

![facturasDB1](/PeliculasWebAPI/images/FacturaDB1.PNG)

Al ejecutar, recibimos un status `500` el cual nos indica que dos personas quieren actualizar el mismo registro al mismo tiempo. 

![concurrenciaResult](/PeliculasWebAPI/images/Facturas%20Concurrencia%20Fila%20Result.PNG)

Al verificar de nuevo nuestra tabla de Facturas en nuestra Base de Datos, notamos que solo se actualizó un registro, pero no ambos. 

![facturasDB2](/PeliculasWebAPI/images/FacturaDB2.PNG)

#### Manejo de Conflictos de Concurrencia

Al momento solo hemos visto los conflictos al usar concurrencia, por lo que cual veremos la forma de manejar los conflictos, se manejan dos exceopciones, el primero es el valor que se quiso ejecutar y el segundo el valor anterior antes de actualizarse. 

En nuestro `FacturasController.cs` creamos un `endpoint` que simulará el manejo de los conflictos. 

![facturaControllerManConcu](/PeliculasWebAPI/images/FacturaController%20Manejo%20Concurrencia.png)

Si probamos, nos devuelve un status `400` y es porque ya un usuario antes actualizó el registro, por el cual devuelve un error. 

![manejoConcurrencia1](/PeliculasWebAPI/images/ManejoConcurrencia%20Result.PNG)

Visto desde nuestra consola: 

![manejoConcurrencia2](/PeliculasWebAPI/images/ManejoConcurrencia%20Result%20Cmd.PNG)

#### Conflictos de Concurrencia con el Modelo Desconectado

Por el momento solo hemos visto los conflictos de concurrencia para el modelo conectado, en este punto lo veremos desde el modelo desconectado, como en ambientes Web. 

Creamos dos nuevos `endpoint`, uno para obtener una factura y otra para actualizar la factura. 

Para obtener una factura.

![obtFactura](/PeliculasWebAPI/images/FacturaController%20ObtFact.png)

Para actualizar una factura. 

![actFactura](/PeliculasWebAPI/images/FacturaController%20ActFact.png)

Al probar, vemos que en ambos nos devuelve un status `200`. 

Para obtener una factura.

![resultObtFac](/PeliculasWebAPI/images/ObtenerFactura%20Result.PNG)

Para actualizar una factura. 

![resultActFac](/PeliculasWebAPI/images/ActualizarFactura%20Result.PNG)

El problema es cuando intentamos actualizar de nuevo el registro, nos devuelve un status `500`.

![errorActFac](/PeliculasWebAPI/images/ActualizarFactura%20Result%202.PNG)

Este mismo caso pasa en `GenerosController.cs` por el cual haremos cambios, primero creamos un DTO el cual va permitir actualizar el registro. 

![generoDTO](/PeliculasWebAPI/images/GeneroDTO.png)

En nuestro `AutoMapperProfile.cs` mapeamos nuestro DTO con nuestra entidad de `Genero.cs`.

    CreateMap<GeneroActualizacionDTO, Genero>();

En nuestro `GenerosController.cs` modificamos nuestro `endpoint` de tipo `PUT` para que pueda solucionarse el conflicto de concurrencia. 

![generosControllePut](/PeliculasWebAPI/images/GenerosControllerPUT.png)

Vemos que nuestra tabla Generos en nuestra Base de Datos, tenemos los registros. 

![generosDB4](/PeliculasWebAPI/images/GeneroDB4.PNG)

Al hacer la actualización, recibimos un status `200`. 

![conflictosResult](/PeliculasWebAPI/images/ConflictosConcurrencia%20Result.PNG)

Al volver a consultar nuestra tabla Generos de nuestra Base de Datos, ya tenemos el registro actualizado. 

![generosDB5](/PeliculasWebAPI/images/GeneroDB5.PNG)

#### Tablas Temporales

En una tabla temporal cuando se crea, crea una tabla de tipo historia, donde guarda los históricos de los cambios cuando se hacen un actualizado o eliminado de un registro. 

Para configurarlo, podemos hacerlo desde el _API Fluent_ de `GeneroConfig.cs`.

![generoConfig](/PeliculasWebAPI/images/GeneroTablaTemporal.png)

Posterior hacemos la migración y empujamos los cambios hacia nuestra Base de Datos. 

![generoMigracion](/PeliculasWebAPI/images/GeneroTablaTemporal%20Migracion.png)

Por lo cual se generan dos tablas o versiones, la primera es donde tienen los datos como se conocen, solo que se agrega dos campos de `PeriodStart` y `PeriodEnd`.

![generoTablaTemporalN](/PeliculasWebAPI/images/GeneroTablaTemporal%20Normal.PNG)

En la segunda tabla muestra el historico y se irá actualizando a medida de que se hagan actualización o eliminación de registros. 

![generoTTHistorico](/PeliculasWebAPI/images/GeneroTablaTemporal%20Historico.PNG)

#### CUD en la Tabla Temporal

Creamos un nuevo genero y se da el registro en nuestra tabla temporal de Generos. 

![generoDB6](/PeliculasWebAPI/images/GeneroDB6.PNG)

Al actualizar el registo desde nuestro `endpoint` obtenemos un status `200`.

![actuGeneroTempo](/PeliculasWebAPI/images/PutGeneros.PNG)

Al visualizar las tabla Generos Temporal e Historico, esto obtenemos, en la tabla Genero Temporal tenemos el nuevo PeriodStart con la nueva actualización mientras que el PeriodEnd fue lo que tuvo el último registro: 

![generoHistorico](/PeliculasWebAPI/images/GeneroHistorico.PNG)

![generoDB7](/PeliculasWebAPI/images/GeneroDB7.PNG)

Si volvemos a actualizar el mismo registro, en nuestra Tabla Generos Historicos, vemos los cambios. 

![generoHistorico2](/PeliculasWebAPI/images/GeneroHistorico2.PNG)

Si borramos el genero de forma definitiva, a través desde nuestro `endpoint`, obtenemos un status `200`.

![generoBorrado](/PeliculasWebAPI/images/GeneroBorrado.PNG)

Si consultamos la tabla Generos de nuestra Base de Datos, notamos que el registro se eliminó: 

![generoDB8](/PeliculasWebAPI/images/GeneroDB8.PNG)

Vemos el historico desde nuestra tabla Generos Historico. 

![generoHistorico3](/PeliculasWebAPI/images/GeneroHistorico3.PNG)

#### Consultando la Tabla Temporal e Histórica

Creamos un `endpoint` que va actualizar repetidas ocasiones un genero, esto desde `GenerosController.cs`.

![mofiicarVariaVeces](/PeliculasWebAPI/images/GenerosControllerModificar.png)

Al probar el `endpoint` obtenemos un status `200`.

![ModifcarVariasVecesResult](/PeliculasWebAPI/images/ModificarVariasVecesGenero.PNG)

Si consultamos nuestra tabla, tenemos lo siguiente. 

![GeneroDB9](/PeliculasWebAPI/images/GeneroDB9.PNG)

Nuestra tabla Generos Historico, registra la serie de cambios. 

![GenerosHistorico4](/PeliculasWebAPI/images/GeneroHistorico4.PNG)

Si queremos consultar nuestra tabla Generos Temporal, modificamos nuestro `endpoint` de `GenerosController.cs`.

![GenerosControllerid](/PeliculasWebAPI/images/GenerosControllerConsultaTemporal.png)

Al probar nuestro `endpoint` obtenemos un status `200` con los datos que nos interesan. 

![GenerosConsultaResult](/PeliculasWebAPI/images/GeneroConsulta.PNG)

Si queremos ver el historico de las versiones, creamos un `endpoint` en `GenerosController.cs`. 

Con el método `TemporalAll()` traemos todos los registros de nuestra tabla temporal como del histórico. 

![GenerosControllerVeriones](/PeliculasWebAPI/images/GenerosControllerConsultaVersiones.png)

Al probar nuestro `endpoint` nos devuelve un status `200` el cual nos trae nuestra data de tabla del histórico. 

![GenerosConsultaVersiones](/PeliculasWebAPI/images/GeneroConsultaVersiones.PNG)

#### TemporalAsOf

Podemos usar esta función para saber los datos de un registro en cierta fecha indicada. 

Para ello creamos un nuevo `endpoint` en `GenerosController.cs`.

![TemporalAsOfGC](/PeliculasWebAPI/images/GeneroControllerTemporalAsOf.png)

Al probar nuestro `endpoint` tenemos un status `200` con la versión del registro en la fecha determinada. 

![TemporalAsOfResult](/PeliculasWebAPI/images/TemporalAsOf%20Result.PNG)


#### TemporalFromTo

También podemos filtrar todo las versiones de un registro por un rango de tiempo, por el cual, usamos el método `TemporalFromTo()`. 

Creamos en un nuevo `endpoint` en nuestro `GenerosController.cs`.

![TemporalFromToGenCon](/PeliculasWebAPI/images/GeneroControllerTemporalFromTo.png)

Al dar el rango de fechas del id, obtenemos el status `200` con la data. 

_Rango de Fecha_

![TemporalFromToResult1](/PeliculasWebAPI/images/TemporalFromTo%20Result%201.PNG)

_Data Resultante_

![TemporalFromToResult2](/PeliculasWebAPI/images/TemporalFromTo%20Result%202.PNG)

#### TemporalContainedIn

Sirve para ver que versiones estuvieron activas o terminaron de estar activas en un cierto tiempo de fecha, es decir, totalmente contenidos. 

Creamos el `endpoint` en nuestro `GenerosController.cs`, el cual va contener el método mencionado. 

![TemporalContainedInGenContr](/PeliculasWebAPI/images/GeneroControllerTemporalContainedIn.png)

Al dar el rango de fecha indicado, solo mostrará los géneros que estén totalmente contenidos en ese rango de fechas. 

![TemporalContainedIn1](/PeliculasWebAPI/images/TemporalContainedIn%20Result%201.PNG)

El resultado nos devuelve un status `200` con dos géneros.

![TemporalContainedIn2](/PeliculasWebAPI/images/TemporalContainedIn%20Result%202.PNG)

#### TemporalBetween

Con este método podemos realizar una consulta en los registros activos en un rango de fecha con la particularidad de que si la fecha final coincide con la fecha de inicio de un registro, este es incluido en el resultado. 

Creamos el `endpoint` en nuestro `GenerosController.cs`, el cual va contener el método mencionado. 

![TemporalBetweenGenController](/PeliculasWebAPI/images/GeneroControllerTemporalBetween.png)

Al dar el rango de fecha indicado, muestra los generos que están en ese rango de fecha más el registro que está próximo a la derecha. 

![TemporalBetween1](/PeliculasWebAPI/images/TemporalBetween%20Result1.PNG)

Es parecido al método `TemporalFromTo()` solo que `TemporalBetween()` tiene la particularidad de incluir el registro del extremo derecho que coincide con la misma fecha, el cual devuelve un status `200` con los registros mencionados. 

![TemporalBetween2](/PeliculasWebAPI/images/TemporalBetween%20Result2.PNG)

#### Restuarando un Registro Borrado

Si tenemos registros eliminados de la tabla temporal, pero que aún se tiene registro en el histórico, podemos restaurarlo, inclusivo, podemos restaurar la versión del registro que deseamos. 

Creamos el `endpoint` de tipo `POST` en nuestro `GenerosController.cs`, el cual va contener el método mencionado, con un query arbitrario.

![RestauraBorradoGenCont](/PeliculasWebAPI/images/GeneroControllerRestauraBorrado.png)

Notamos que en nuestra Tabla Generos Temporal no tenemos el registro de la versión 3 con _id: 27_.

![GeneroDB10](/PeliculasWebAPI/images/GeneroDB10.PNG)

Al probar dando el id y la fecha de la versión del registro. 

![RestaurarBorrado1](/PeliculasWebAPI/images/RestaurarBorrado1.PNG)

Obtenemos un status `200` de que el registro se pudo restaurar e insertar en la Tabla Generos Temporal. 

![ResturarBorrado2](/PeliculasWebAPI/images/RestaurarBorrado2.PNG)

Al ver en nuestra tabla Generos Temporal en nuestra Base de Datos, ya se encuentra el registro resturado. 

![GeneroDB11](/PeliculasWebAPI/images/GeneroDB11.PNG)

#### Personalizando la Tabla Temporal e Histórica

Al configurar una tabla como temporal, se agregan los campos `PeriodStart` y `PeriodEnd` además de que se crea una tabla con el nombre `History`, se puede personalizar estas configuraciones .

Vamos a configurar la tabla `Factura` como temporal, para ello lo hacemos desde su `API Fluent` de `FacturaConfig.cs`.

![facturaConfig](/PeliculasWebAPI/images/FacturaConfig.png)

Posteriormente hacemos la migración y empujamos los cambios hacia la Base de Datos. 

![FacturaTemporalMigracion](/PeliculasWebAPI/images/FacturasTemporalMigracion.png)

Al comprobar los cambios en la Base de Datos, notamos que la Tabla Factura Temporal ya tiene los cambios personalizados. 

![Facturatemporal](/PeliculasWebAPI/images/FacturasDB1.PNG)

En la tabla Factura Historico ya viene de igual forma personalizada. 

![FacturaHistorico](/PeliculasWebAPI/images/FacturasHistorico1.PNG)

