Script
=====================

Presentacion
---------------

¿Quién soy, de dónde vengo?

- Me llamo Jorge Sanz...
- Trabajo en Prodevelop
- Trabajamos para puertos de todo tipo

El mundo real es duro
------------------------

- En los puertos hay delineantes

  - Conocimientos de dibujo técnico, topografía
  - Pocos o ningún conocimiento de bases de datos o  GIS
  - Solo usan AutoCAD
  - Dibujos

    - http://www.flickr.com/photos/kingcounty/8139974275/
    - http://www.flickr.com/photos/kozmicdogz/5618334387/
    - http://www.flickr.com/photos/the-lobster/4014103606/

- Los puertos quieren usar GIS

  - Gestión de los espacios (€€€)
  - Gestión de emergencias
  - Conocimiento del estado actual del puerto
  - ...

- Por lo tanto hay que alimentar un GIS con datos del puerto

  - Toca cargar datos CAD en bases de datos geoespaciales
  - Nuestra pelea diaria desde hace 20 años

Caso de uso real
-----------------------

- Estamos con una aplicación de gestion de espacios del puerto
- Cartografía en DWG
- Hay que llevar esa cartografía a una base de datos Oracle
- Lo tiene que poder hacer el delineante conforme realice
  cambios en su cartografía

Requerimientos
-----------------------

- Ejecución a demanda
- Desde AutoCAD (los delineantes VIVEN en AutoCAD)
- Debe mostrar información del resultado del proceso
- Debe ser tolerante a todos los fallos/guarradas que
  el delineante pueda meter en la cartografía

Solución
----------------------

- Geokettle como herramienta ETL de migración de datos
- Creación de un botón en AutoCAD que:

  - guarda una copia del dibujo actual como DXF
  - lanza el proceso de Geokettle: de DXF a Oracle
  - muestra los resultados

Primera parte: en AutoCAD
----------------------------

No tiene mucho interés, un simple botón programado en
Visual Basic que lanza un archivo de lotes de DOS
que parametriza todo el proceso y acaba lanzando una llamada
a la herramienta de consola de GeoKettel (``kitchen.bat``)

::

  CALL Kitchen.bat /level %LOG% /file "sincronizar-dxf-oracle.kjb" "/param:GPC_logfile=%LOGFILE%" "/param:GPC_logfile_detail=%LOGFILEDETAIL%" "/param:GPC_dxf_file=%DXF%" "/param:GPC_dxf_capa=%CAPA%" "/param:GPC_db_host=%DB_HOST%" "/param:GPC_db_port=%DB_PORT%" "/param:GPC_db_name=%DB_NAME%" "/param:GPC_db_user=%DB_USER%" "/param:GPC_db_pass=%DB_PASS%" "/param:GPC_tabla_tmp=%BP_TABLA%" "/param:GPC_tabla_tmp_geom=%BP_TABLA_GEOM%" "/param:GPC_tabla_tmp_idx=%BP_TABLA_INDEX%"

Segunda parte:
------------------------------

Qué es geokettle
+++++++++++++++++++++++++++++

- Kettle (ahora Pentaho Data Integration) es una herramienta ETL

  - Extracción
  - Transformación
  - Carga

- Dispone de tres interfaces de usuario

  - Spoon: Interfaz Gráfica para diseñar
  - Kitchen: CLI
  - Carte: Interfaz web

- Conceptos básicos

  - Paso: elemento mínimo de un proceso ETL

    - Leer un origen
    - Ejecutar código JavaScript para crear datos derivados
    - Cargar en un destino

  - Transformación: procedimiento ETL
  - Trabajo: coordinación de transformaciones

> The Data Integration perspective of Spoon allows you to create two basic document types: transformations and jobs. Transformations are used to describe the data flows for ETL such as reading from a source, transforming data and loading it into a target location. Jobs are used to coordinate ETL activities such as defining the flow and dependencies for what order transformations should be run, or prepare for execution by checking conditions such as, "Is my source file available?," or "Does a table exist in my database?"


- GeoKettle:

  - Fork (técnico) que añade pasos geo
  - Multiplataforma
  - LGPL
  - Se integra con OGR, GDAL, SEXTANTE
  - Usa JTS internamente
  - Dispone de pasos para trabajar con catálogos CSW, análisis de datos, transformación de coordenadas, cliente WPS,....


Nuestro trabajo:
++++++++++++++++++++++++++++++++++

Pasos:

#. Eliminar el índice de la tabla (Oracle hack, SQL)
#. Cargar el DXF con OGR

   - filtrando por capa para evitar cargar datos innecesarios

#. Filtrar geometrías que no sean de tipo linea (JavaScript)
#. Poligonizar las geometrías con JTS (JavaScript)
#. Filtrar campos
#. Cargar en tabla Oracle (borrando datos anteriores)
#. Actualizar geometrías con el CRS correcto (SQL)
#. Reparar geometrías con (SQL):

   - Sentido de giro incorrecto
   - Nodos repetidos
   - ???

#. Crear de nuevo el índice (SQL)

..note:: se genera un log sencillo del proceso

..attention:: falta log


Conclusiones
---------------------------

- Geokettle es una potente herramienta de manipulación de datos
- Es un GIS de escritorio dedicado a automatizar procesos
- Adaptado a usuarios con conocimientos variados: OGR/GDAL, SQL, JavaScript,...
