
# Schwäbische Nuclei U-Net

## Definition of our training data
- [`stage1_train_fixed.zip `](https://drive.google.com/open?id=1tVUQjHYZqyAZ7QeIAWQ_FlP1xReSZ0Kv) with `md5sum 9a3e938a312baa30fcea84c476a278cb` 
We merged the original `stage1_train.zip` with [this fixed stage1 train data](https://github.com/lopuhin/kaggle-dsbowl-2018-dataset-fixes/tree/master/stage1_train),
generated the combined masks and contours under `prep_masks` under each imageID directory, and programmatically removed the following imageIDs from the training set on the fly.
* ###### 7b38c9173ebe69b4c6ba7e703c0c27f39305d9b2910f46405993d2ea7a963b80
* ###### adc315bd40d699fd4e4effbcce81cd7162851007f485d754ad3b0472f73a86df
* ###### 12aeefb1b522b283819b12e4cfaf6b13c1264c0aadac3412b4edd2ace304cb40
- For stage 2, if allowed, we'll add [`stage1_test.zip`](https://www.kaggle.com/c/data-science-bowl-2018/data) with the released masks to our training data

## How to reproduce our submissions


### Train our colour models

- Train 3 models with full size of colour train images with data augmentation and then window on all augmented data and rotate/flip on all windowing data for 10 epochs. With a limitation of total training image size of roughly 300k on AWS with one GPU V100


- model 1c mosaic with Gaussian noise and speckle noise data augmentation, `sigma = 0.1`

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --noise 0.1 --rotate --epoch 10 --valsplit 0 --colouronly`

- model 2c mosaic with perspective transform data augmentation, `sigma = 0.175`

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --transform 0.175 --rotate --epoch 10 --valsplit 0 --colouronly`

- model 3c mosaic only 

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --rotate --epoch 10 --valsplit 0 --colouronly`



### Train our grey models and retrain our colour models

- Train 3 models with full size of grey/colour train images with data augmentation and then window on all augmented data and rotate/flip on all windowing data for 10 epochs. With a limitation of total training image size of roughly 300k on AWS with one GPU V100

- model 1g mosaic with Gaussian noise and speckle noise data augmentation, `sigma = 0.1`

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --noise 0.1 --rotate --epoch 10 --valsplit 0 --loadmodel output_from_model_1c/`


- model 2g mosaic with perspective transform data augmentation, `sigma = 0.175`

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --transform 0.175 --rotate --epoch 10 --valsplit 0 --loadmodel output_from_model_2c/`


- model 3g mosaic only 

`python schwaebische_nuclei.py train --maxtrainsize 300000 --mosaic --rotate --epoch 10 --valsplit 0 --loadmodel output_from_model_3c/`


### Model predictions blending 

#### Predict on grey images using each grey models: `model_1g, model_2g, model_3g`

- Window and rotate/flip test images and predict on those grey test images using each model

- Average weighted predictions of `model_1g`, `model_2g` and `model_3g` with weight distribution `[4, 2, 1]`:

`predictions = ( 4 * model_1g + 2 * model_2g + 1 * model_1g) / 7`

- Predictions on contours and masks will be generated automatically and saved to the disk

#### Predict on colour images using each colour models: `model_1c, model_2c, model_3c`

- Window and rotate/flip test images and predict on those colour test images using each model

- Average weighted predictions of `model_1c`, `model_2c` and `model_3c` with weight distribution `[4, 2, 1]`:

`predictions = ( 4 * model_1c + 2 * model_2c + 1 * model_3c ) / 7` by issuing the following command:


```
python schwaebische_nuclei.py predict --loadmodel output_from_all_models/ --weights 4 2 1
```

### Generate submissions

- After weighted predictions get averaged, CSV will be generated by the above command, we use this for our submissions
- Predictions on contours and masks will be generated automatically and saved to the disk
 



