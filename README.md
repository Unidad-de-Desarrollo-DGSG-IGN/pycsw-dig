# PyCSW para DIG
[pycsw](https://pycsw.org/) es un servidor que implementa el estándar OGC CSW escrito en Python.

Su documentación se encuentra en https://pycsw.org/docs/

## Despliegue

La aplicación se encuentra preparada para manejarse dentro de un contenedor Docker.
Los pasos para su despliegue son:

1. Descargar los archivos desde el repositorio:

```bash
git clone https://github.com/Unidad-de-Desarrollo-DGSG-IGN/pycsw-dig
```
2.  Ejecutar el contenedor Docker parado en el mismo directorio donde se clonó el repositorio (por ejemplo, `cd pycsw-dig`):
```bash
# En entornos Windows
docker run --rm --name pycsw-test --volume .\config\default.cfg:/home/pycsw/pycsw/default.cfg --volume .\db:/var/lib/pycsw --volume .\gisdata-master:/gisdata-master --detach --publish 8000:8000 geopython/pycsw
#En entornos Linux
docker run --rm --name pycsw-test --volume ./config/default.cfg:/home/pycsw/pycsw/default.cfg --volume ./db:/var/lib/pycsw --volume ./gisdata-master:/gisdata-master --detach --publish 8000:8000 geopython/pycsw
```
3. Para saber si el servidor se ha desplegado correctamente acceder a http://localhost:8000/

## Administración
Por defecto, pycsw funciona con una base de datos SQLite, la misma se encuentra en el archivo `./db/data.db`
La información que allí contiene se generó a partir de la ejecución del siguiente comando:

```bash
#Inicializa una nueva base de datos
docker exec -ti pycsw-test pycsw-admin.py setup_db -c pycsw/default.cfg
#Recorre el directorio indicando (de forma recursiva) y carga en la base de datos todo XML de metadatos que encuentre
docker exec -ti pycsw-test pycsw-admin.py load_records -c pycsw/default.cfg -p /gisdata-master/gisdata/metadata/good -r
```
Como se puede apreciar, toda la administración de pycsw se realiza a través de la ejecución del programa por línea de comando `pycsw-admin.py`. A través del comando:
```bash
docker exec -ti pycsw-test pycsw-admin.py --help
```
se listan las opciones que ofrece dicho programa.
No existe un administrador gráfico para gestionar metadatos ni para visualizarlos.

Para revisar los metadatos que se han registrado en el sistema se puede recorrer el directorio `/gisdata-master/gisdata/metadata/good`
Como se mencionó anteriormente, no existe una forma visual de ver los metadatos, la aplicación solo ofrece una API para su consulta. Esta API tampoco ofrece la posibilidad de gestionar los metadatos, solo se puede hacer esta tarea a través de `pycsw-admin.py`.

Todos los atributos de los metadatos son registrados en la base de datos en una única tabla:

```sql
CREATE TABLE records (
	identifier TEXT NOT NULL, 
	typename TEXT NOT NULL, 
	schema TEXT NOT NULL, 
	mdsource TEXT NOT NULL, 
	insert_date TEXT NOT NULL, 
	xml VARCHAR NOT NULL, 
	anytext TEXT NOT NULL, 
	metadata VARCHAR, 
	metadata_type TEXT NOT NULL, 
	language TEXT, 
	type TEXT, 
	title TEXT, 
	title_alternate TEXT, 
	abstract TEXT, 
	edition TEXT, 
	keywords TEXT, 
	keywordstype TEXT, 
	themes TEXT, 
	parentidentifier TEXT, 
	relation TEXT, 
	time_begin TEXT, 
	time_end TEXT, 
	topicategory TEXT, 
	resourcelanguage TEXT, 
	creator TEXT, 
	publisher TEXT, 
	contributor TEXT, 
	organization TEXT, 
	securityconstraints TEXT, 
	accessconstraints TEXT, 
	otherconstraints TEXT, 
	date TEXT, 
	date_revision TEXT, 
	date_creation TEXT, 
	date_publication TEXT, 
	date_modified TEXT, 
	format TEXT, 
	source TEXT, 
	crs TEXT, 
	geodescode TEXT, 
	denominator TEXT, 
	distancevalue TEXT, 
	distanceuom TEXT, 
	wkt_geometry TEXT, 
	servicetype TEXT, 
	servicetypeversion TEXT, 
	operation TEXT, 
	couplingtype TEXT, 
	operateson TEXT, 
	operatesonidentifier TEXT, 
	operatesoname TEXT, 
	degree TEXT, 
	classification TEXT, 
	conditionapplyingtoaccessanduse TEXT, 
	lineage TEXT, 
	responsiblepartyrole TEXT, 
	specificationtitle TEXT, 
	specificationdate TEXT, 
	specificationdatetype TEXT, 
	platform TEXT, 
	instrument TEXT, 
	sensortype TEXT, 
	cloudcover TEXT, 
	bands TEXT, 
	links TEXT, 
	contacts TEXT, 
	PRIMARY KEY (identifier)
);
```

Una vez cargada la información en la base de datos de pycsw, la misma puede ser consultada mediante las solicitudes HTTP estándares de CSW. Por ejemplo, http://localhost:8000/csw?service=CSW&version=2.0.2&request=GetCapabilities
