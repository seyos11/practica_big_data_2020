# BDFI 2020

A continuación se describe como realizar el despliegue de los múltiples servicios tanto en local como en contenedores Docker.


## Installation

En primer lugar hay que extrae el zip y movernos a la carpeta generada. Posteriormente llamamos al archivo download_data.sh para descargar los datos. Este paso no es realmente necesario ya que los datos vienen incorporados en el zip.

```bash
unzip practica_big_data_2020.zip
cd practica_big_data_2020
resources/download_data.sh
```
En siguiente lugar se instalan las librerías de python que son necesarias para el despliegue del proyecto que vienen en el archivo requirements.txt.

```bash
pip install -r requirements.txt
```

A continuación hay que poner en marcha los servicios de kafka y zookeeper, del cual depende kafka para un óptimo funcionamiento

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties

bin/kafka-server-start.sh config/server.properties

bin/kafka-topics.sh \
        --create \
        --zookeeper localhost:2181 \
        --replication-factor 1 \
        --partitions 1 \
        --topic flight_delay_classification_request
````

En siguiente lugar, se ha de generar la base de datos de mongo para que tenga la estructura deseada y acorde a lo que se genera en el formulario web.
```bash
./resources/import_distances.sh
````

En esta repositorio se entrega la carpeta models donde están los modelos que usa spark para realizar predicciones, consiguiendo una precisión de 0,58. No es necesario volver a entrenar.

El siguiente paso a seguir es la compilación del proyecto y de la clase de Scala MakePredicitions.scala. Para ello hay que dirigirse a la subcarpeta flight_prediciton con el terminal y con sbt compilar y generar el paquete JAR que posteriormente lanzaremos a spark.

```bash
sbt compile
sbt package
```
Si quisieramos correrlo directamente podríamos hacer un 

```bash
sbt run
```
Finalmente, se lanza en spark el archivo jar generado anteriormente.

```bash
/$SPARK_HOME/bin/spark-submit --class es.upm.dit.ging.predictor.MakePrediction --master local --packages org.mongodb.spark:mongo-spark-connector_2.11:2.3.2,org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.0 ./target/scala-2.11/flight_prediction_2.11-0.1.jar
```
Este archivo jar file esta proporcionado también con lo que no sería necesario los pasos de compilación de sbt. Simplemente valdría con ejecutar el siguiente comando para lanzarlo.

```bash
/$SPARK_HOME/bin/spark-submit --class es.upm.dit.ging.predictor.MakePrediction --master local --packages org.mongodb.spark:mongo-spark-connector_2.11:2.3.2,org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.0 ./flight_prediction_2.11-0.1.jar
```

Por último, lo que ha de hacerse es lanzar el programa web con los siguientes comandos.
```bash
export PROJECT_HOME=/home/user/Desktop/practica_big_data_2019
cd practica_big_data_2019/resources/web
python3 predict_flask.py
```
Ahora podríamos ir a la página http://localhost:5000/flights/delays/predict_kafka e interactuar con el servicio para finalmente ver cómo se devuelven las predicciones que a su vez están guardadas en mongoDB.
````bash
mongo
 > use use agile_data_science;
 >db.flight_delay_classification_response.find();
```
Para la correcta ejecución en local de todo lo explicado habría que tener descargado java,scala, spark y mongo db ya que no se ha podido proporcionar el código que automatiza eso. Sin embargo, a continuación explicamos como con Docker logramos quitarnos estos problemas de en medio.


#################################DOCKER#####################################################



##Extraemos el zip y lanzamos Docker. Posteriormente llamamos al archivo download_data.sh para descargar los datos, este paso es necesario porque se borran los datos para evitar que ocupen mucho espacio

````bash
unzip practica_big_data_2020.zip
cd practica_big_data_2020
resources/download_data.sh
docker-compose up
```

####Docker en Google cloud. IP dinámica desde la que se accede correctamente en el momento de la realización.

````bash
34.107.117.123:5000/flights/delays/predict_kafka
```

