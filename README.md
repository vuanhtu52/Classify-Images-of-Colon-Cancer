# Classify Images of Colon Cancer

Assume you are a team of machine learning engineers working for a biomedical startup company. Your task is to develop a machine learning system that can classify histopathology images of colon cells. A basic description of histopathology images can be found here.

You will be using a modified version of the "CRCHistoPhenotypes" dataset for this task. The data set for you to use in this assignment has been specifically prepared for you, and is provided on Canvas. The data set consists of 27x27 RGB images of colon cells from 99 different patients and you are expected to use the data set to perform two tasks:

Classify images according to whether given cell image represents a cancerous cells or not (isCancerous).

Classify images according to cell-type, such as: fibroblast, inflammatory, epithelial or others.

The correct classification of the images is given by the "data_labels_mainData.csv" and "data_labels_extraData.csv". For the first 60 patients, the medical experts have provided labels isCancerous and cell-type. However for the remaining 39 patients, the medical experts have only provided labels for isCancerous.

The X_main_data.npy and X_extra_data.npy are images from the main and extra datasets which have been converted to bumpy arrays.