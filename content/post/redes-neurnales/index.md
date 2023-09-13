---
title: Como crear un clasificador de imagenes con redes neuronales
tags: 
- IA 
- Python 
- Tensorflow 
- Keras
categories:
- Tutoriales
date: 2023-8-7 00:00:00+0000
description: En este post se muestra como crear un clasificador de imágenes con redes neuronales.
---

En la actualidad la inteligencia artificial esta en todas partes, desde los asistentes de voz de los smartphones hasta los sistemas de reconocimiento facial de las redes sociales. En este post vamos a ver como crear un clasificador de imágenes con redes neuronales.

Como lenguaje de programación vamos a usar Python ya que es uno de los lenguajes más populares para la inteligencia artificial. Para crear el clasificador de imágenes vamos a usar la librería Tensorflow-Keras, que es una librería de alto nivel que nos permite crear redes neuronales de forma sencilla. En especifico vamos a crear una CNN (Convolutional Neural Network).

## ¿Que es una CNN?
Una CNN es una red neuronal que se utiliza principalmente para clasificar imágenes. Una CNN es una red neuronal que tiene capas de convolución y capas de pooling. Las capas de convolución son capas que extraen características de las imágenes, mientras que las capas de pooling reducen la dimensionalidad de las imágenes.

## ¿Que necesitamos para crear el clasificador de imágenes?
Como todas las redes neuronales para entrenar se necesita un dataset, en este caso de imágenes, para entrenar vamos a utilizar una dataset real no una de ejemplo, ya que las de ejemplo se importan directamente al código y no muestra como se hace el preprocesamiento del dataset, que es uno de los pasos mas importantes para la creación de nuestro modelo, nuestro modelo es tan bueno como el dataset que le demos.

## Preprocesamiento del dataset
Vamos a descargar el [dataset de clasificación de fabricación de kaggle](https://www.kaggle.com/datasets/ravirajsinh45/real-life-industrial-dataset-of-casting-product?resource=download) que contiene ejemplos de piezas fabricadas en aluminio, donde se clasifican en 2 categorías diferentes, defectuosas y no defectuosas. El dataset contiene 3000 imágenes de 2 categorías diferentes, 1500 imágenes de piezas defectuosas y 1500 imágenes de piezas no defectuosas. El dataset esta dividido en 2 carpetas, una para las imágenes de entrenamiento y otra para las imágenes de prueba y dentro de cada carpeta hay 2 carpetas, una para las imágenes defectuosas y otra para las imágenes no defectuosas. De manera que el dataset tiene la siguiente estructura:

    dataset
    ├───train
    │   ├───def_front
    │   └───ok_front
    └───validation
        ├───def_front
        └───ok_front


Con esto ya tenemos nuestro dataset, ahora vamos a crear nuestro modelo.

## Creación del modelo
Como la creación de los modelos de redes neuronales necesitan ejecutar bloques de código de manera repetitiva para encontrar el mejor modelo, vamos a usar google colab, que es un servicio de google que nos permite ejecutar código de manera remota en la nube, de manera que no necesitamos tener un ordenador con una GPU para entrenar nuestros modelos, ya que google colab nos permite usar una GPU de manera gratuita.

Como mencione anteriormente vamos a usar Tensorflow-Keras, por lo cual vamos a importar las librerías necesarias para crear nuestro modelo.
```python
import tensorflow as tf
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow.keras.optimizers import RMSprop
from tensorflow.keras.preprocessing import image
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import os
import random
import numpy as np
```
Importamos la librería de Tensorflow ademas de otras que nos serán útiles para el preprocesamiento del dataset y la visualización de las imágenes como el caso de matplotlib.

Como mencione vamos a usar colab, por lo cual necesitamos subir nuestro dataset a colab, para reducir el tiempo de subida vamos a convertir nuestra carpeta de dataset en un zip y subirlo a Google Drive, al tenerlo ya en nuestro Drive vamos a descomprimirlo con la librería zipfile.
```python
import zipfile
with zipfile.ZipFile("/content/drive/MyDrive/IA/T.zip","r") as zip_ref:
zip_ref.extractall("/content/drive/MyDrive/IA/")
```
Con esto ya tenemos nuestra carpeta de dataset descomprimida en nuestro Drive.

Ahora podemos proseguir con la creación de las clases del dataset, y con esto vemos la importancia de la estructura de nuestra carpeta del dataset, ya que keras nos permite crear las clases de manera automática, solo necesitamos especificar la ruta de la carpeta de entrenamiento y la carpeta de prueba. Iniciando con la carpeta de entrenamiento.
```python
train_ds = tf.keras.utils.image_dataset_from_directory(
    '/content/drive/MyDrive/IA/casting_data/train',
    validation_split=0.2,
    subset="training",
    seed=123,
    image_size=(300,300))
```
Esta función tiene varios parámetros, el primero es la ruta de la carpeta de entrenamiento, el segundo es el porcentaje de imágenes que se van a usar para la validación, el tercero es el tipo de dataset, en este caso es de entrenamiento, el cuarto es la semilla para la generación de números aleatorios, y el último es el tamaño de las imágenes que vamos a usar, en este caso 300x300.

Ahora vamos con la dataset de validación.
```python
val_ds = tf.keras.utils.image_dataset_from_directory(
  '/content/drive/MyDrive/IA/T/train',
  validation_split=0.2,
  subset="validation",
  seed=200,
  image_size=(300,300))
```
Esta función tiene los mismos parámetros que la anterior, solo cambia el tipo de dataset, en este caso es de validación, y la semilla para la generación de números aleatorios.

Si imprimimos las clases creadas podemos ver el nombre de las clases y vemos que keras las ha creado de manera automática con el nombre de las carpetas.
```python
class_names = train_ds.class_names
print(class_names)
```
```bash
['def_front', 'ok_front']
```
Ahora vamos a visualizar algunas imágenes de nuestro dataset, para esto vamos a crear una función que nos permita visualizar las imágenes.
```python
import matplotlib.pyplot as plt # Importar la librería de matplolib para gráficas

plt.figure(figsize=(10, 10))
for images, labels in train_ds.take(1):
  for i in range(9):
    ax = plt.subplot(3, 3, i + 1)
    plt.imshow(images[i].numpy().astype("uint8"))
    plt.title(class_names[labels[i]])
    plt.axis("off")
```
<p align="center">
<img src="https://i.imgur.com/qlN7lmK.png">
</p>



Ahora vamos a crear nuestro modelo.
```python
num_classes = len(class_names)

model = Sequential([
  layers.Rescaling(1./255, input_shape=(300, 300, 3)),
  layers.Conv2D(16, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(32, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(64, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Flatten(),
  layers.Dense(128, activation='relu'),
  layers.Dense(num_classes)
])
```
Este modelo tiene 10 capas, la primera capa es de normalización, la segunda es una capa de convolución con 16 filtros, la tercera es una capa de max pooling, la cuarta es una capa de convolución con 32 filtros, la quinta es una capa de max pooling, la sexta es una capa de convolución con 64 filtros, la séptima es una capa de max pooling, la octava es una capa de aplanamiento, la novena es una capa densa con 128 neuronas y la última es una capa densa con 2 neuronas, ya que tenemos 2 clases.

Esta es la estructura básica de cualquier red CNN, la cual se compone de capas de convolución, capas de max pooling y finalmente capas densas.

Ahora compilamos el modelo.
```python
model.compile(optimizer='adam',
              loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
              metrics=['accuracy'])
```
Para asegurarnos que todo salio bien vamos a imprimir el resumen del modelo.
```python
model.summary()
```
```bash
Model: "sequential_3"
_________________________________________________________________
 Layer (type)                Output Shape              Param #
=================================================================
 rescaling_5 (Rescaling)     (None, 300, 300, 3)       0

 conv2d_15 (Conv2D)          (None, 300, 300, 16)      448

 max_pooling2d_15 (MaxPoolin  (None, 150, 150, 16)     0
 g2D)

 conv2d_16 (Conv2D)          (None, 150, 150, 32)      4640

 max_pooling2d_16 (MaxPoolin  (None, 75, 75, 32)       0
 g2D)

 conv2d_17 (Conv2D)          (None, 75, 75, 64)        18496

 max_pooling2d_17 (MaxPoolin  (None, 37, 37, 64)       0
 g2D)

 flatten_5 (Flatten)         (None, 87616)             0

 dense_8 (Dense)             (None, 128)               11214976

 dense_9 (Dense)             (None, 2)                 258

=================================================================
Total params: 11,238,818
Trainable params: 11,238,818
Non-trainable params: 0
_________________________________________________________________
```
Ahora vamos a entrenar el modelo.
```python
epochs=4
history = model.fit(
  train_ds,
  validation_data=val_ds,
  epochs=epochs
)
```
Vamos a iniciar el entrenamiento con 4 épocas, pero si queremos podemos aumentar el número de épocas para mejorar el modelo.

## Evaluación del modelo
Para evaluar un modelo tenemos dos métricas que nos pueden ayudar a saber si el modelo esta bien entrenado, estas son la precisión y la pérdida. Para entenderlas las graficaremos.

```python
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']

loss = history.history['loss']
val_loss = history.history['val_loss']

epochs_range = range(epochs)

plt.figure(figsize=(8, 8))
plt.subplot(1, 2, 1)
plt.plot(epochs_range, acc, label='Training Accuracy')
plt.plot(epochs_range, val_acc, label='Validation Accuracy')
plt.legend(loc='lower right')
plt.title('Training and Validation Accuracy')

plt.subplot(1, 2, 2)
plt.plot(epochs_range, loss, label='Training Loss')
plt.plot(epochs_range, val_loss, label='Validation Loss')
plt.legend(loc='upper right')
plt.title('Training and Validation Loss')
plt.show()
```
En la gráfica de la izquierda podemos ver que la precisión de entrenamiento y en la de la derecha la pérdida de entrenamiento.


<p align="center">
<img src="https://i.imgur.com/IckUEug.png">
</p>

Para interpretar estas gráficas podemos ver que la precisión de entrenamiento aumenta con cada época, pero la precisión de validación se estanca en 0.8, lo cual nos dice que el modelo esta sobreajustado, ya que no esta generalizando bien.

De la misma manera podemos ver que la pérdida de entrenamiento disminuye con cada época, pero la pérdida de validación se estanca en 0.4, lo cual nos dice que el modelo esta sobreajustado, ya que no esta generalizando bien.

En resumen entre mas cerca estén las lineas azules y naranjas, mejor es el modelo. En este caso hay un pequeño sobreajuste, pero no es tan grave.

Con esto podemos finalizar el modelo, ahora vamos a evaluarlo con datos que no ha visto.

```python
test_ds = tf.keras.utils.image_dataset_from_directory(
  '/content/drive/MyDrive/IA/casting_data/test',
  seed=123,
  image_size=(300,300))
```
creamos un dataset de validación y evaluamos el modelo.
```python
model.evaluate(test_ds,return_dict=True)
```
Esto nos devuelve un diccionario con las métricas que definimos al compilar el modelo, en este caso la precisión y la pérdida.

```bash
23/23 [==============================] - 66s 2s/step - loss: 0.0397 - accuracy: 0.9902
{'loss': 0.039654314517974854, 'accuracy': 0.9902098178863525}
```
Vemos entonces que la precisión es de 0.99 y la pérdida es de 0.04, lo cual es muy bueno.

## Predicción
Ahora vamos a probar el modelo con una imagen que no ha visto y que seria una entrada del modelo en la vida real.
```python
image_path = '/content/drive/MyDrive/IA/casting_data/test/def_front/cast_def_0_1189.jpeg'
image = tf.keras.preprocessing.image.load_img(image_path)
input_arr = tf.keras.preprocessing.image.img_to_array(image)
input_arr = np.array([input_arr])  # Convert single image to a batch.
predictions = model.predict(input_arr)
```
Todo esto es para convertir la imagen en un array de numpy y poder predecir con el modelo ya que en realidad los modelos no se entrenan con imagenes, sino con arrays que componen las imagenes. Ahora vemos la predicción.

```python
score = tf.nn.softmax(predictions[0])
print(
    "Esta imagen parece ser {} con un {:.2f} % de exactitud."
    .format(class_names[np.argmax(score)], 100 * np.max(score))
)
```
```bash
Esta imagen parece ser def_front con un 100.00 % de exactitud.
```
Lo cual es correcto, ya que la imagen es de un defecto frontal. Con esto finalmente podemos decir que el modelo esta listo para ser usado en la vida real. Ahora lo guardamos para poder usarlo en el futuro.

```python
model.save('/content/drive/MyDrive/IA/casting_data/model')
```
Para cargar el modelo en el futuro solo tenemos que hacer lo siguiente.
```python
model = tf.keras.models.load_model('/content/drive/MyDrive/IA/casting_data/model')
```

## Extra
Como mencione este modelo tiene un sobreajuste, pero no es tan grave, para mejorar el modelo podemos hacer lo siguiente:
- Aumentar el número de épocas.
- Agregar más datos de entrenamiento.
- Agregar dropout.

Una solución para el sobreajuste es el dropout, que consiste en eliminar aleatoriamente neuronas de la red, para que no se sobreajusten. Para agregarlo solo tenemos que agregar una capa de dropout después de cada capa de flatten y antes de las capas densas, dropout de 0.2, lo cual significa que el 20% de las neuronas serán eliminadas aleatoriamente.

```python
model = Sequential([
  layers.Rescaling(1./255, input_shape=(300, 300, 3)),
  layers.Conv2D(16, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(32, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(64, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Flatten(),
  layers.Dropout(0.2),
  layers.Dense(128, activation='relu'),
  layers.Dense(num_classes)
])
```
Hacemos los mismo pasos para compilar y entrenar el modelo, esta vez nos da las siguientes métricas.

<p align="center">
<img src="https://i.imgur.com/gwG8vbT.png">
</p>


Donde se ve que al final las dos lineas son casi iguales, lo cual significa que no hay sobreajuste. Ahora evaluamos el modelo con los datos de validación.
```bash
final.evaluate(test_ds,return_dict=True)
23/23 [==============================] - 19s 815ms/step - loss: 0.0224 - accuracy: 0.9958
{'loss': 0.02243557944893837, 'accuracy': 0.9958041906356812}
```
Con esto terminamos el tutorial, espero que les haya gustado y que les sirva para sus proyectos.