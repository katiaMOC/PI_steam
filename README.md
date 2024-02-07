## Proyecto MLOps: Sistema de recomendacion de Videojuegos par aUsuarios Steam
![Imagen de un gato](https://canalusb.cubadebate.cu/wp-content/uploads/2023/12/Steam.jpg)


# Prueba de concepto para proyecto de Steam

## Introducción

Este proyecto simula el rol de un MLOps Engineer, es decir, la combinación de un Data Engineer y Data Scientist, para la plataforma multinacional de videojuegos Steam. Para su desarrollo, se entregan unos datos y se solicita un Producto Mínimo Viable que muestre una API deployada en un servicio en la nube y la aplicación de dos modelos de Machine Learning, por una lado, un análisis de sentimientos sobre los comentarios de los usuarios de los juegos y, por otro lado, la recomendación de juegos a partir de dar el nombre de un juego y/o a partir de los gustos de un usuario en particular.

## Contexto

Steam es una plataforma de distribución digital de videojuegos desarrollada por Valve Corporation. Fue lanzada en septiembre de 2003 como una forma para Valve de proveer actualizaciones automáticas a sus juegos, pero finalmente se amplió para incluir juegos de terceros. Cuenta con más de 325 millones de usuarios y más de 25.000 juegos en su catálogo. Es importante tener en cuenta que las cifras publicadas por SteamSpy son hasta el año 2017, porque a principios de 2018 Steam limitó la forma de obtener estadísticas, por eso no hay datos tan precisos.

## Datos

Para este proyecto se proporcionaron tres archivos JSON:

* **australian_user_reviews.json** es un dataset que contiene los comentarios que los usuarios realizaron sobre los juegos que consumen, además de datos adicionales como si recomiendan o no ese juego, emoticones de gracioso y estadísticas de si el comentario fue útil o no para otros usuarios. También presenta el id del usuario que comenta con su url del perfil y el id del juego que comenta.

* **australian_users_items.json** es un dataset que contiene información sobre los juegos que juegan todos los usuarios, así como el tiempo acumulado que cada usuario jugó a un determinado juego.

* **output_steam_games.json** es un dataset que contiene datos relacionados con los juegos en sí, como los título, el desarrollador, los precios, características técnicas, etiquetas, entre otros datos.

En el archivo `Diccionario_de_datos-md` se encuetran los detalles de cada una de las variables de los conjuntos de datos.

## Tareas desarrolladas

### Transformaciones

Se realizó la extracción, transformación y carga (ETL) de los tres conjuntos de datos entregados. Dos de los conjuntos de datos se encontraban anidados, es decir había columnas con diccionarios o listas de diccionarios, por lo que aplicaron distintas estrategias para transformar las claves de esos diccionarios en columnas. Luego se rellenaron algunos nulos de variables necesarias para el proyecto, se borraron columnas con muchos nulos o que no aportaban al proyecto, para optimizar el rendimiento de la API y teneniendo en cuenta las limitaciones de almacenamiento del deploy. Para las transformaciones se utilizó la librería Pandas.

Los detalles del ETL se puede ver en el archivo `ETL_steam_games.ipynb`, `ETL_user_items.ipynb` y `ETL_user_reviews.ipynb` 

### Feature engineering

Uno de los pedidos para este proyecto fue aplicar un análisis de sentimiento a los reviews de los usuarios. Para ello se creó una nueva columna llamada 'sentiment_analysis' que reemplaza a la columna que contiene los reviews donde clasifica los sentimientos de los comentarios con la siguiente escala:

* 0 si es malo,
* 1 si es neutral o esta sin review
* 2 si es positivo.

Dado que el objetivo de este proyecto es realizar una prueba de concepto, se realiza un análisis de sentimiento básico utilizando TextBlob que es una biblioteca de procesamiento de lenguaje natural (NLP) en Python. El objetivo de esta metodología es asignar un valor numérico a un texto, en este caso a los comentarios que los usuarios dejaron para un juego determinado, para representar si el sentimiento expresado en el texto es negativo, neutral o positivo. 

Esta metodología toma una revisión de texto como entrada, utiliza TextBlob para calcular la polaridad de sentimiento y luego clasifica la revisión como negativa, neutral o positiva en función de la polaridad calculada. En este caso, se consideraron las polaridades por defecto del modelo, el cuál utiliza umbrales -0.2 y 0.2, siendo polaridades negativas por debajo de -0.2, positivas por encima de 0.2 y neutrales entre medio de ambos.

Por otra parte, y bajo el mismo criterio de optimizar los tiempos de respuesta de las consultas en la API y teniendo en cuenta las limitaciones de almacenamiento en el servicio de nube para deployar la API, se realizaron dataframes auxiliares para cada una de las funciones solicitadas. En el mismo sentido, se guardaron estos dataframes en formato *parquet* que permite una compresión y codificación eficiente de los datos.

Todos los detalles del desarrollo se pueden ver en la Jupyter Notebook `Feature_eng.ipynb`

### Análisis exploratorio de los datos

Se realizó el EDA a los tres conjuntos de datos sometidos a ETL con el objetivo de identificar las variables que se pueden utilizar en la creación del modelo de recmendación. Para ello se utilizó la librería Pandas para la manipulación de los datos y las librerías Matplotlib y Seaborn para la visualización.

En particular para el modelo de recomendación, se terminó eligiendo construir un dataframe específico con el id del usuario que realizaron reviews, los nombres de los juegos a los cuales se le realizaron comentarios y una columna de rating que se construyó a partir de la combinación del análisis de sentimiento y la recomendación a los juegos.

El desarrollo de este análisis se encuentra en la Jupyter Notebook `EDA.ipynb`

### Modelo de aprendizaje automático

Se crearon dos modelos de recomendación, que generan cada uno, una lista de 5 juegos ya sea ingresando el nombre de un juego o el id de un usuario.

En el primer caso, el modelo tiene una relación ítem-ítem, esto es, se toma un juego y en base a que tan similar es ese juego con el resto de los juegos se recomiendan similares. En el segundo caso, el modelo aplicar un filtro usuario-juego, es decir, toma un usuario, encuentra usuarios similares y se recomiendan ítems que a esos usuarios similares les gustaron.

Para generar estos modelos se adoptaron algoritmos basados en la memoria, los que abordan el problema del **filtrado colaborativo** utilizando toda la base de datos, tratando de encontrar usuarios similares al usuario activo (es decir, los usuarios para los que se les quiere recomendar) y utilizando sus preferencias para predecir las valoraciones del usuario activo.

Para medir la similitud entre los juegos (item_similarity) y entre los usuarios (user_similarity) se utilizó la **similitud del coseno** que es una medida comúnmente utilizada para evaluar la similitud entre dos vectores en un espacio multidimensional. En el contexto de sistemas de recomendación y análisis de datos, la similitud del coseno se utiliza para determinar cuán similares son dos conjuntos de datos o elementos, y se calcula utilizando el coseno del ángulo entre los vectores que representan esos datos o elementos.

El desarrollo para la creación de los dos modelos se presenta en la Jupyter Notebook `Modelo_recomendacion.ipynb`

### Desarrollo de API

Para el desarrolo de la API se decidió utilizar el framework FastAPI, creando las siguientes funciones:

* **developer**: Esta función recibe como parámetro 'developer', que es la empresa desarrolladora del juego, y devuelve la cantidad de items que desarrolla dicha empresa y el porcentaje de contenido Free por año por sobre el total que desarrolla.

* **userdata**: Esta función tiene por parámentro 'user_id' y devulve la cantidad de dinero gastado por el usuario, el porcentaje de recomendaciones que realizó sobre la cantidad de reviews que se analizan y la cantidad de items que consume el mismo.

* **userforgenre**: Esta función recibe como parámetro el género de un videojuego y devuelve el top 5 de los usuarios con más horas de juego en el género ingresado, indicando el id del usuario y el url de su perfil.

* **best_developer_year**: Esta función recibe como parámetro el año de lanzamiento del juego y devuelve el top 3  de desarrolladores con juegos MÁS recomendados por usuarios para el año dado.

* **sentiment_analysis**: Esta función recibe como parámetro la empresa desarrolldora y según a ello devuelve una lista con la cantidad de registros de reseñas de usuarios que se encuentren categorizados con un análisis de sentimiento, como Negativo, Neutral y Positivo.

* **recomendacion_juego**: Esta función recibe como parámetro el nombre de un juego y devuelve una lista con 5 juegos recomendados similares al ingresado.

* **recomendacion_usuario**: Esta función recibe como parámetro el id de un usuario y devuelve una lista con 5 juegos recomendados para dicho usuario teniendo en cuenta las similitudes entre los usuarios.

> *NOTA: Aún no esta deployado en render, debido a que sobrepasamos la capacidad gratuita de render, sin embargo de mostrara de manera local en un video.*

El desarrollo de las funciones de consultas generales se puede ver en la Jupyter Notebook `funcionesAPI.ipynb`
El desarrollo del código para las funciones del modelo de recomendación se puede ver en la Jupyter Notebook `Modelo_recomendacion.ipynb`

El código para generar la API se encuentra en el archivo `main.py` y las funciones para su funcionamiento se encuentran en `api_functions.py`. En caso de querer ejecutar la API desde localHost se deben seguir los siguientes pasos:

- Clonar el proyecto haciendo `git clone https://github.com/katiaMOC/PI_steam.git`.
- Preparación del entorno de trabajo en Visual Studio Code:
    * Crear entorno `Python -m venv env`
    * Ingresar al entorno haciendo `venv\Scripts\activate`
    * Instalar dependencias con `pip install -r requirements.txt`
- Ejecutar el archivo main.py desde consola activando uvicorn. Para ello, hacer `uvicorn main:app --reload`
- Hacer Ctrl + clic sobre la dirección `http://XXX.X.X.X:XXXX` (se muestra en la consola).
- Una vez en el navegador, agregar `/docs` para acceder a ReDoc.
- En cada una de las funciones hacer clic en *Try it out* y luego introducir el dato que requiera o utilizar los ejemplos por defecto. Finalmente Ejecutar y observar la respuesta.

### Deployment



### Video



## Oportunidades de mejoras

Dado que el objetivo de este proyecto fue presentar un Producto Mínimo Viable, se realizaron algunos análisis básicos que se podrían mejorar en próximas etapas, con la idea de lograr un producto completo. Por ejemplo:

* **Análsis de sentimiento**: se puede hacer la limpieza de los comentarios, dado que los mismos se encuentran en distintos idiomas, con emoticones y signos de puntuación. Por otra parte, se puede evaluar el rendimiento del modelo probando con distintos umbrales de clasificación.

* **Mejorar EDA**: se puede hacer un análisis exploratorio de datos mas exhaustivo, buscando mas relaciones entre los juego y usarios que permitan crear un puntaje mas representativo para hacer las recomendaciones.

* **Mejorar ETL**: se pueden haces más transformaciones en algunas variables usadas en la API, como por ejemplo precios, donde muchos campos tenían palabras y solo se cambió por precio cero, porque muchos textos se referian a juegos gratuitos, pero no se observó en detalle. También había datos faltantes que se completaron con 0, pero no se investigó si eran juegos gratuitos. Esto puede afectar a los resultados de la API donde pregunta por porcentaje de juegos gratuitos.

* **Otras opciones de servicio en la nube**: se pueden investigar otras formas de deployar la API de modo de no tener las limitaciones de capacidad de almacenamiento y poder utilizar la última función del modelo de recomendación o buscar alternativas para almacenar los datos por fuera de Render y conectar con esa fuente para las consultas.#   P I _ s t e a m  
 