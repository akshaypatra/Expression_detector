##  imports

import os
import numpy as np
Imports:

-> os is used for interacting with the file system.
-> numpy is used for loading .npy files and working with arrays.


##  Initializes variables

X = None
y = None
is_init = False
label = []
dictionary = {}
c = 0 


-> X, y: Will hold the combined data and labels.
-> is_init: Flag to check if the first file has been processed.
-> label: List to store class names (from filenames).
-> dictionary: Maps each class name to a unique integer label.
-> c: Counter for assigning unique label values.



<!-- Line 12 -->

for i in os.listdir():

-> Loops over all files in the current directory.


    if i.endswith(".npy") and not i.startswith("labels"):

-> Processes only .npy files except labels.npy.


        class_name = i.split('.')[0]

-> Extracts the class name (e.g., from cat.npy, gets "cat").


        data = np.load(i)

-> Loads the NumPy array from the file into memory.


        size = data.shape[0]

-> Gets how many samples are in the file (assuming data is 2D or more).



        if not is_init:

-> For the first file only:


            X = data
            y = np.array([class_name] * size).reshape(-1, 1)
            is_init = True

-> Initializes X with the data.
-> Creates the y array with repeated class_name for each row.
-> Sets is_init to True so future files are appended.


        else:
            X = np.concatenate((X, data))
            y = np.concatenate((y, np.array([class_name] * size).reshape(-1, 1)))

-> For all subsequent files:
-> Appends data to X.
-> Appends corresponding labels to y.


        label.append(class_name)

-> Adds the class name to the list of labels.


        dictionary[class_name] = c
        c += 1

-> Assigns a unique number (c) to the class name and stores it in dictionary.
-> Increments the counter for the next class.



<!-- Line 29 : -->

for i in range(y.shape[0]):
    y[i, 0] = dictionary[y[i, 0]]

-> For each row in y, you replace the string class label with its corresponding integer index from dictionary



y = np.array(y, dtype="int32")

-> Ensures that the label array is of type int32 (needed for categorical operations).

y = utils.to_categorical(y)

-> Uses Keras utility to one-hot encode the numeric labels.
E.g., if labels were [0, 1], this becomes [[1, 0], [0, 1]].

<!-- Line 42 : -->

X_new = X.copy()
y_new = y.copy()
counter = 0

-> Creates new copies of the data and label arrays for shuffling.


cnt = np.arange(X.shape[0])
np.random.shuffle(cnt)

-> Generates an array of indices from 0 to X.shape[0] - 1, then shuffles them.

<!-- Line 49 : -->

for i in cnt: 
    X_new[counter] = X[i]
    y_new[counter] = y[i]
    counter = counter + 1

-> Uses the shuffled indices to reorder X and y into X_new and y_new.
-> This effectively shuffles the dataset.


ip = layers.Input(shape=(X.shape[1],))

-> Creates the input layer for a neural network.
-> Input shape is the number of features per sample (i.e., X.shape[1]).

<!-- Line 56 : -->

m = layers.Dense(512, activation="relu")(ip)
m = layers.Dense(256, activation="relu")(m)

-> Adds two hidden layers:
    -- First with 512 neurons and ReLU activation. (activation functions in neural networks.)
    -- Second with 256 units and ReLU.



op = layers.Dense(y.shape[1], activation="softmax")(m) 


-> Output layer:
    --  Number of units = number of classes (y.shape[1] after one-hot encoding).
    --  Uses softmax to produce class probabilities.



model = models.Model(inputs=ip, outputs=op)

-> Defines the final model by connecting the input and output layers.

<!-- Line 63 -->

model.compile(optimizer='rmsprop', loss="categorical_crossentropy", metrics=['acc'])

-> Compiles the model:
    -- Optimizer: RMSprop (good for small datasets).
    -- Loss: Categorical crossentropy (since it's multi-class classification).
    -- Metric: Accuracy.



model.fit(X, y, epochs=50)

-> Trains the model on your original dataset for 50 epochs.(An epoch refers to one complete pass through the entire training dataset)

model.save("model.h5")

-> Saves the trained model to a file model.h5 (standard Keras model format).


np.save("labels.npy", np.array(label))

-> Saves the list of class names (label) to a .npy file.
-> This is helpful later to decode predictions (i.e., map class index back to class name).