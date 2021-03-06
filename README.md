# Importing Images with Keras Image Generator

Image data is a somewhat new concept, and is  certainly formatted differently than most data we have worked with so far. 

Luckily keras provides us with some useful tools for easily importing image datasets.


```python
from keras.preprocessing.image import ImageDataGenerator, img_to_array, load_img

# Turn of TensorFlow deprecation warnings
import warnings
warnings.filterwarnings('ignore', category=DeprecationWarning)
warnings.filterwarnings('ignore', category=FutureWarning)

# Modeling
from keras.models import Sequential 
from keras.layers import Dense 
```

In this repo, our image data is stored like so:

```
data/training_set/cat
                    |__cat.1.jpg
                    |__...
                  dog
                    |__dog.1.jpg
                    |__...
data/test_set/cat
                |__cat.4001.jpg
                |__...
              dog
                |__dog.4001.jpg
                |__...
```
The organization is such that our training and test set folders both contain two folders. 1 for each class.  
We can easily import these images, by using a Keras ImageDataGenerator. 

ImageDataGenerator objects have several parameters that allow us to rotate, flip, or shift images which can improve the likelihood of our model generalizing to new data. We will not these features this morning, but we  will use the rescale parameter to normalize the image's pixel values.


```python
datagen = ImageDataGenerator(rescale=1.0/256.0)
```

Now that we've instantiated an `ImageDataGenerator` object, we can create generators for the training and testing data.


```python
# prepare an iterators for each dataset 
train_generator = data_generator.flow_from_directory('data/training_set/', class_mode='binary', 
                                       target_size=(256,256), batch_size=64) 
test_generator = data_generator.flow_from_directory('data/test_set/', class_mode='binary',
                                     target_size=(256,256), batch_size=64)
```

    Found 8000 images belonging to 2 classes.
    Found 2000 images belonging to 2 classes.


We then compile a basic CNN model.


```python
from keras.models import Sequential
from keras.layers import Conv2D, MaxPooling2D
from keras.layers import Activation, Dropout, Flatten, Dense

model = Sequential() 
model.add(Conv2D(32, (3, 3), activation='relu', padding='same', input_shape=(256, 256, 3))) 
model.add(MaxPooling2D((2, 2))) 
model.add(Flatten()) 
model.add(Dense(128, activation='relu')) 
model.add(Dense(1, activation='sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
```

And then instead of using `model.fit`, we use `model.fit_generator` and instead of directly feeding the model the data, we use the generator in place of the training data.

>This single epoch took about 15 minutes to run.


```python
model.fit_generator(
        train_generator,
        epochs=1)
```

    Epoch 1/1
    125/125 [==============================] - 884s 7s/step - loss: 2.5482 - accuracy: 0.5825





    <keras.callbacks.callbacks.History at 0x63f9563d0>




```python
acc = model.evaluate_generator(test_generator, steps=len(test_generator), verbose=1)
list(zip(model.metrics_names,acc))
```

    32/32 [==============================] - 9s 293ms/step





    [('loss', 0.5680567026138306), ('accuracy', 0.6754999756813049)]


