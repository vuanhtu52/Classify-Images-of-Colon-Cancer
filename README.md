# Classify Images of Colon Cancer
## Project Description
This project aims to  develop a machine learning system that can classify histopathology images of colon cells. The system uses a modified version of the "CRCHistoPhenotypes" dataset for this task. The dataset consists of 27x27 RGB images of colon cells from 99 different patients and is used to perform two tasks:
- Classify images according to whether a given cell image represents a cancerous cell or not (isCancerous).
- Classify images according to cell type, such as: fibroblast, inflammatory, epithelial or others.

For the first 60 patients, the medical experts have provided labels for isCancerous and cell-type. However, for the remaining 39 patients, the medical experts have only provided labels for isCancerous.

![(a) Example histopathelogy image of the colon with individual cells marked with "blue" rectangles of size 27x27. (b) Example of different cell types present in histopathelogy images of the colon.](Images/cells.png)

## Experiment Design
| | Total instances | Fibroblast | Inflammatory | Epithelial | Miscellaneous | Normal | Cancerous |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| _**Main dataset**_ |
| Train | 6332 | 1208 | 1627 | 2610 | 887 | 3722 | 2610 |
| Validation | 1584 | 302 | 407 | 653 | 222 | 931 | 653 |
| Test | 1980 | 378 | 509 | 816 | 277 | 1164 | 816 |
| _**Extra dataset**_ |
| Train | 8307 | N/A | N/A | N/A | N/A | 5915 | 2392 |
|Validation | 2077 | N/A | N/A | N/A | N/A| 1479 | 598 |

- For the first 60 patients (Main dataset), we split the data into train, validation and test sets with the 60/20/20 ratio. The models are trained and fine-tuned on the train and validation sets. The test set is reserved to evaluate how well the models perform on unseen data. 
- For the remaining 39 patients (Extra dataset), we split them into train and validation sets with the 80/20 ratio. This dataset is later used to improve the performance on the cancer detection task by utilizing the extra isCancerous labels.

## Model Evaluation
### 1. Baselines 
| | Training time (s) | Accuracy | Precision | Recall | F1 |
| ------ | ------ | ------ | ------ | ------ | ------ | 
| _**Cancer detection**_ |
| VGG | 70 | 0.8955 | 0.8955 | 0.8955 | 0.8951 |
| Customized model | 38 | 0.8914 | 0.8978 | 0.8914 | 0.8926 |
| SC-CNN | 49 | 0.8924 | 0.8971 | 0.8924 | 0.8934 | 
| RCCNet | 258 | **0.9126** | **0.9137** | **0.9126** | **0.9129** |
| _**Cell type classification**_ |
| VGG | 60 | 0.7288 | 0.8030 | 0.7288 | 0.7502 |
| Customized model | 24 | 0.7455 | 0.7464 | 0.7455 | 0.7429 |
|Softmax CNN | 64 | 0.7237 | **0.8189** | 0.7237 | 0.7621 |
| RCCNet | 159 | **0.7687** | 0.7635 | **0.7687** | **0.7626** |   

For the baselines, we have researched 3 different architectures and propose a customized model of our own based on the VGG architecture. All models are trained in 50 epochs. If there is no improvment within 10 epochs, the training will stop to avoid overfitting.

The models use Adam optimizer with intial learning rate of 6 x 10<sup>-5</sup> iteratively decreased by a factor of \sqrt{0.1} if there is no improvement of validation loss during the training. The images  go through an augmentation process where they are distorted, zoomed and flipped randomly. 

### 2. Model improvement 
| Model | Training time (s) | Accuracy | Precision | Recall | F1 |
| ------ | ------ | ------ | ------ | ------ | ------ |
| _**Cancer detection**_ | 
| RCCNet (main) | 258 | **0.9126** | **0.9137** | **0.9126** | **0.9219** |
| RCCNet (main + extra) | 521 | 0.9040 | 0.9111 | 9.9040 | 0.9052 |
| _**Cell type classification**_ |
| RCCNet | 159 | 0.7687 | 0.7635 | 0.7687 | 0.7626 |
| RCCNet + pseudo labelling | 399 | **0.8040** | **0.8033** | **0.8040** | **0.8032** | 

We attempt to improve the performance on the cancer detection task by combining the training and validation sets from the main and extra datasets to retrain our model. The result on the test set is worse, which suggests there might be lots of noise on the extra dataset.

For the cell type classification task, we apply a technique called pseudo labelling with the following steps:
- Train a classification model on the main dataset (called the teacher model).
- Use the teacher model to predict cell types for the extra dataset.
- Combine the main dataset and the predicted labels on the extra dataset to train a new classification model (called the student model).

The result shows significant improvement on the test set, which indicates our method is effective.

### 3. Parameter tuning
| Run | Batch size | Optimizer | Training time (s) | Accuracy | Precision | Recall | F1 |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| _**Cancer detection**_ |
| 1 | 64 | Adam | 258 | 0.9126 | 0.9137 | 0.9126 | **0.9219** |
| 2 | 64 | SGD | 171 | 0.8909 | 0.8963 | 0.8909 | 0.8919 |
| 3 | 64 | Nadam | 181 | 0.9121 | 0.9130 | 0.9121 | 0.9124 |
| 4 | 32 | Adam | 169 | **0.9152** | **0.9162** | **0.9152** | 0.9154 |
| 5 | 32 | SGD | 152 | 0.8995 | 0.9015 | 0.8995 | 0.9000 |
| 6 | 32 | Nadam | 114 | 0.9116 | 0.9124 | 0.9116 | 0.9118 |
| _**Cell type classification**_ |
| 1 | 64 | Adam | 399 | 0.8040 | 0.8033 | 0.8040 | 0.8032 |
| 2 | 64 | SGD | 331 | 0.7646 | 0.7854 | 0.7646 | 0.7727 | 
| 3 | 64 | Nadam | 358 | 0.7944 | 0.7987 | 0.7944 | 0.7956 |
| 4 | 32 | Adam | 372 | 0.7960 | 0.7970 | 0.7960 | 0.7954 |
| 5 | 32 | SGD | 385 | 0.7798 | 0.7831 | 0.7798 | 0.7799 |
| 6 | 32 | Nadam | 420 | **0.8081** | **0.8136** | **0.8081** | **0.8099** | 

At final stage, we try to tweak the batch size and optimizer to improve our models. The results show a slight improvement on both tasks. 

## Conclusion
At the end of the project, we delivered the two best models for the two tasks mentioned above after thorough research and analysis. 
- Cancer detection: The best model is the RCCNet trained on the main dataset, using the Adam optimizer with batch size 32. 
- Cell type classification: The best model is the RCCNet using the pseudo labelling technique. The model uses the Nadam optimizer with batch size 32.

For future work, we could perform a more in-depth process for parameter tuning by tweaking more parameters inside each layer such as padding, stride, number of filters, etc. to further improve the models' performances.

## Acknowledgement
Thank you Jin for helping me deliver this project. You have been a great teammate!








 

