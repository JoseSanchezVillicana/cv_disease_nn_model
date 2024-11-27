# Workflow para análisis y entrenamiento de modelos de aprendizaje supervisado: Cardiovascular Disease dataset

## ***Chombo Álvarez Bernardo, Sánchez Villicaña José Antonio***

## Entendimiento del dominio

Las enfermedades cardiovasculares (CVDs) son una de las causas de muerte más frecuentes a nivel mundial. Para el 2019 la OMS estimó que 17.9 millones de personas murieron de una CVD, lo que representó un 32% de las muertes globales, de las cuales 85% fueron por causadas por un paro cardiaco. ­Los infartos o paros cardiacos suelen ser causados por el bloqueo de la circulación del corazón hacia el cerebro por depósitos de grasa en las paredes internas de los vasos sanguíneos. Factores asociados a la conducta como el mantener una dieta no saludable, la inactividad física, consumo de tabaco y de alcohol se asocian al desarrollo de CVDs. Su etiología puede manifestarse con el incremento de la presión sanguínea, incremento en los niveles de glucosa y lípidos en la sangre, así como sobrepeso y obesidad.

Para este trabajo se empleó un dataset de información estándar del estado de salud y de la presencia o ausencia de una enfermedad cardiovascular en 70,000 pacientes. Este dataset es de código abierto y puede encontrarse en la siguiente liga en la base de datos Kaggle: [https://www.kaggle.com/sulianova/cardiovascular-disease-dataset](https://www.kaggle.com/sulianova/cardiovascular-disease-dataset). Del total de pacientes, 34,979 presentan una enfermedad cardiovascular y 35,021 no la presentan. Además, este contiene 13 atributos sin contar el index (*Age\_days, Age\_year, Height, Weight, Gender, Systolic blood pressure, Diastolic blood pressure, Cholesterol, Glucose, Smoking, Alcohol intake, Physical activity, Cardiovascular Disease Status*), de las cuales algunos son numéricos, otros categóricos y otros binarios.

## Exploración de datos

Tras realizar la exploración del dataset, se encontró que 29,330 pacientes que no presentan una CVD y 23,055 pacientes que sí presentan una CVD tienen el colesterol en niveles normales; 3,799 pacientes que no presentan una CVD y 5,750 que sí la presentan tienen niveles de colesterol elevados; 1,892 pacientes que presentan una CVD y 6,174 que no la presentan padecen hipertensión. Pacientes mujeres y hombres tienen distribuciones diferentes en sus alturas, siendo los hombres los que cuentan con datos de altura más altos. Al momento de comparar las presiones sistólicas y diastólicas se encontró una aparente relación entre ambas a niveles altos con presencia de algunos valores que no hacían sentido como valores negativos, extremadamente altos o casos donde la presión sistólica fuese menor a la diastólica. Para las mediciones de la altura y el peso, también se encontraron valores muy bajos (55 cm de altura; 11 kg de peso) y valores muy altos (250 cm de altura).

## Preparación de datos

Se emplearon diversos criterios para el pre procesamiento y el filtrado del dataset. De primera instancia, se crearon datos categóricos para los grupos de edad en donde se obtuvieron 5 grupos de edad (20-30, 30-40, 40-50, 50-60, 60-70). Posteriormente, se filtraron las presiones sistólica (sis) y diastólica (días) con los siguientes criterios: 70 \<= sis \<= 300, 40 \<= días \<= 200; para después eliminar a los pacientes que tuvieran presiones diastólicas mayores que las presiones sistólicas puesto que esto es imposible (este filtrado ya elimina valores negativos e iguales a 0), y se crearon valores categóricos para la presión sistólica (normal: 70-120, elevado: 120-130, hipertensión: 130-300). Después se le prestó atención a otra medida que es el pulso arterial. Este se define como la diferencia entre la presión sistólica y la presión diastólica. Con este nuevo valor, se procedió a eliminar valores menores a 30 por no ser considerados normales y estar fuera del espectro de una enfermedad cardiovascular, y se generaron valores categóricos (normal: 30-60, elevado: 60-80, patológico: 80-150). Esto asegura que todos los datos con los que se esté trabajando en función de los diferentes valores de presión sean reales.

Otro filtrado que se implementó fue el remover los *outliers* de los atributos de peso y de altura, puesto que se encontraban valores imposiblemente altos y bajos. Para hacer esto, se calcularon los diferentes cuartiles y con ellos el rango intercuartílico (IQR por sus siglas en inglés). Con este valor se procedió a descartar todos los valores para cada una de estas dos columnas que fueran menores o iguales al primer cuartil (Q1) menos una vez y media el rango intercuartílico (IQR), y todos los que fueran mayores o iguales al tercer cuartil (Q1) más una vez y media el rango intercuartílico (IQR). Este es un método estandarizado que asegura un filtrado homogéneo y bajo criterios comunes no arbitrarios. Finalmente, se calculó el índice de masa corporal (BMI por sus siglas en inglés) con el peso dividido entre la altura, expresada en metros, elevada al cuadrado.

Como un primer acercamiento, mediante la función **bestNormalize::bestNormalize()** en R se evaluó si los atributos del dataset requerían una transformación/normalización. La función devolvió los diferentes métodos de normalización para los atributos, pero ninguno de ellos los transformaba adecuadamente puesto que forzaba a una distribución normal cuando desde un inicio ya se acercaban a este tipo de distribución. 

## Creación de modelo: Regresión Logística Múltiple

Se probó un modelo de predicción estadística como primer acercamiento. Debido a la naturaleza del problema de clasificación, la regresión apropiada es la logística. Para hacerlo comparable con otros modelos que toman en cuenta múltiples atributos, decidimos hacer la regresión también múltiple. Se utilizaron los mismos datos normalizados que se mencionan en la sección anterior.

Obtuvimos una R2=0.2364 después de hacer el cálculo de la X2. También graficamos los residuales, los cuales por inspección visual no parecen tener un sesgo. Pero al momento de hacer el histograma de los mismos, no siguen una distribución normal, confirmado por una prueba de Kolmogorov-Smirnov con un p=2.2e-16. Encontramos que todos los atributos, con excepción de algunas (pulso, índice de masa corporal, género, altura), parecen ser significativos para el modelo. A partir de esto realizamos la gráfica de dispersión de la presencia de enfermedad cardiovascular según la presión sistólica y niveles de colesterol. 

## Creación de modelo: Red Neuronal Profunda

El primer paso fue decidir el modelo de aprendizaje, las Redes Neuronales Profundas (*Deep Neural Networks*, DNNs). Modelos computacionales inspirados en la estructura y funcionamiento del cerebro humano. Estas redes consisten en capas de nodos (*neuronas artificiales*) interconectadas, que procesan información en pasos sucesivos para resolver problemas complejos. Son una parte fundamental del aprendizaje profundo (*Deep Learning*), un subcampo del aprendizaje automático (*Machine Learning*). En este caso, lo que buscamos es un modelo predictor que, en base a los atributos incluidos en el dataset (más los que inferimos en este trabajo), sea capaz de clasificar a cada paciente como “con enfermedad cardiovascular” o “sano”.

Lo primero es una vez que nos quedamos con el dataset procesado, como se discute en la sección anterior. Debido a que este procesamiento fue de limpieza (consideramos que las transformaciones no daban buenos resultados), pasamos a transformar los datos para aumentar su utilidad en la DNN. El procedimiento fue estandarizar los datos con la función **sklearn.preprocessing.StandardScaler()** la cual resta la media a los datos y divide entre la desviación estándar. De esta forma centramos los datos en el espacio, de manera estándar. En este paso se procedió a también escalar los atributos categóricos debido a que necesitaban ser más comparables entre sí al estar estandarizados.

El siguiente paso fue diseñar la arquitectura de la red. Existen ciertas limitaciones, como que la entrada de la capa inicial debe ser de la misma dimensión que la cantidad de atributos predictores. Otra regla es que la entrada de cada capa intermedia debe ser de la misma dimensión que la salida de la capa anterior. Por último, la salida de la última capa debe coincidir con la cantidad de clases a predecir. En este caso estamos ante un problema de clasificación binaria. Es importante tener esta consideración presente y no tratar el problema como un problema de clasificación multiclase erróneamente. Fuera de estas limitaciones mencionadas, la arquitectura puede ser tan flexible como uno quiera realmente. Nosotros nos decantamos por una expansión y siguiente compresión de las capas, de manera simétrica. Llegando a la capa más intermedia con el tamaño máximo de 512 nodos. De ahí se vuelven a colapsar las capas, de manera gradual, hasta tener la capa de salida con 1 sólo nodo. Por último, al tratarse de un problema de clasificación binaria y para tener flexibilidad en pasos más adelante, se aplica la función de activación sigmoidea para tener finalmente una probabilidad en lugar de un logit.

Para el entrenamiento del modelo, se dividió el dataset inicial estandarizado en proporciones 80:20 para la fase de entrenamiento y de test respectivamente. Se hizo la división del dataset con el parámetro *random\_state= 42* por efectos de reproducibilidad. Posteriormente se crearon batches de tamaño 128, con el parámetro *shuffle=True* para aregurarnos de evitar posibles sesgos ocasionados por el orden intrínseco de los datos. Por limitaciones computacionales, se limitó el entrenamiento a 10 épocas como máximo, aunque es relevante mencionar que se vió aumento de la precisión entre cada época por lo que es razonable suponer que se podría obtener una mejor precisión con mayores recursos. Se utilizó la función de pérdida **torch.nn.BCELoss()** para el manejo de probabilidades, debido a la naturaleza del problema de clasificación. También se utilizó la función de optimización **torch.optim.AdamW() con** una tasa de pérdida de *lr=1e-5* .

Gracias a que la DNN nos está regresando una probabilidad de clasificar a cada paciente como enfermo o sano, podemos hacer distintos barridos de esta probabilidad para quedarnos con aquella con la mejor precisión. Después de una época de entrenamiento, la clasificación como “enfermo” siempre que el modelo predice una probabilidad \>= 50%, era bastante mejor que probabilidades más exigentes. Pero después de 10 épocas, además de que el modelo mejoraba en general, también las precisiones del mismo con probabilidades más exigentes del 60% y 70%. Esto nos habla de que con más épocas de entrenamiento, mejora el poder predictivo así como la confianza con la que el modelo lo hace. Podemos suponer que con suficientes épocas de entrenamiento, y posiblemente mayores recursos computacionales, podríamos tener un modelo tanto con mayor precisión como con mayor robustez de sus predicciones.

Al final contamos con una DNN entrenada durante 10 épocas, con batches de tamaño de 128 elementos, con una precisión final de 73.7%. Mayores pruebas, a distintas arquitecturas, así como distintas tasas de optimizaciones por época, serían necesarias para determinar el mejor posible modelo para estos datos.

## Análisis de resultados

En este caso observamos que la DNN superó a la regresión logística múltiple, lo cual puede deberse a varios factores. Las DNN superan a la regresión logística en problemas predictivos complejos debido a su capacidad para modelar relaciones no lineales y capturar patrones intrincados entre las variables. Mientras la regresión logística asume una relación lineal entre las características y el logit de la probabilidad, las redes neuronales utilizan múltiples capas y funciones de activación no lineales para aprender representaciones jerárquicas de los datos, lo que las hace más flexibles y poderosas. Además, pueden manejar variables de distinta naturaleza, identificar interacciones implícitas y minimizar el impacto del ruido en los datos. Aunque esto implica mayor capacidad de ajuste, también las hace más propensas al sobreajuste y menos interpretables en comparación con la simplicidad y transparencia de la regresión logística.

Este punto es crucial, ya que el proceso de entrenamiento de una red neuronal profunda, basado en el ajuste iterativo de parámetros, genera un efecto de “caja negra”. Aunque en teoría es posible rastrear e identificar estos ajustes, la naturaleza misma de las múltiples capas de la red dificulta esta tarea. Por ello, la interpretabilidad y transparencia de métodos como la regresión logística siguen siendo valiosas. Estos modelos estadísticos permiten identificar patrones en los datos de manera más directa, lo que puede ser útil para optimizar los conjuntos de datos antes de utilizarlos en procesos de aprendizaje más complejos, como las redes neuronales, que tienden a oscurecer las relaciones subyacentes entre las variables.

Como punto final, es importante destacar que otros modelos de aprendizaje, con paradigmas distintos a las DNN o modelos estadísticos como las regresiones, también pueden ser interesantes y útiles. El flujo de trabajo presentado aquí tiene la ventaja de ser adaptable a una variedad de enfoques, independientemente de sus implementaciones específicas. Esto abre la posibilidad de expandir y mejorar el trabajo a futuro mediante entrenamientos más extensos y refinados, destacando así el potencial de la ciencia de datos en la actualidad.


Se puede encontrar la parte del procesamiento y la creación de la red neuronal
en las siguientes ligas:

- [Procesamiento](https://github.com/JoseSanchezVillicana/cv_disease_nn_model/tree/master/docs/Processing.html)
- [DNN](https://github.com/JoseSanchezVillicana/cv_disease_nn_model/blob/master/bin/classification.ipynb)