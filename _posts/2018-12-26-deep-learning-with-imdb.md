---
layout: post
title: "深度學習模型在文本的情緒預測"
date: 2018-12-26
description: "深度學習與文字探勘"
tag: [Deep Learning]
---


### 資料介紹與預測目標
資料：IMDB Movie reviews sentiment classification
標準： 88%(RNN)
目標： 90%以上

### 參數設定
#### 資料特徵選取

```
Max_feature = 20000          # 資料特徵數
Max_length  =   500          # 句子長度
```

### 深度學習模型參數設定
* 由於此次模型有包含GRU（LSTM的快速版），為避免耗費過多時間，因此 **迭代次數** 只設定3次，相對地必須提高 **批次訓練的樣本數**，在此設定為100
* 今次預測問題為文字情緒的好壞，也就是說 y_label只有0與1的值，因此 **損失函數** 設定為 **binary_crossentropy(二元分類)**，相對地輸出層的 **激活函數** 必須設定為 **sigmoid(羅吉斯回歸的一種)**

```
Epoch = 3                    # 迭代次數
batch_size = 100             # 批次訓練樣本
validation_split = 0.2       # 設定驗證集資料

loss = "binary_crossentropy" # 損失函數
optimizer = "adam"           # 優化器
metrics = "accuracy"         # 評分
```

### 多層感知器 Multiple Layer Perceptron(MLP)
#### **模型架構**
![](https://i.imgur.com/bc4dC94.png)

#### **程式碼**

```
# Initialize model
model <- keras_model_sequential()
model %>%
  # Creates dense embedding layer; outputs 3D tensor
  # with shape (batch_size, sequence_length, output_dim)

  layer_embedding(input_dim = max_features,
                  output_dim = 128,
                  input_length = maxlen) %>%
  layer_flatten() %>%
  layer_dense(units = 32, activation = "relu") %>%
  layer_dense(units = 1, activation = 'sigmoid')

# Optimizer
model %>% compile(
  loss = 'binary_crossentropy',
  optimizer = 'adam',
  metrics = c('accuracy')
)

# Train model
model %>% fit(
  x_train, y_train,
  batch_size = 100,
  epochs = 3,
  validation_split = 0.2
)

# Prediction & ConfusionMatrix
dnn_y_pred <- model %>% predict(x_test)
dnn_y_pred <- as.numeric(dnn_y_pred > 0.5)
dnn_results <- model %>% evaluate(x_test, y_test)
dnn_cMatrix <- table(dnn_y_pred,y_test)
```

#### **訓練**
![](https://i.imgur.com/BtN5V6v.jpg)

```
Train on 20000 samples, validate on 5000 samples
Epoch 1/3
20000/20000 [==============================] - 6s 280us/step - loss: 0.3990 - acc: 0.8002 - val_loss: 0.3048 - val_acc: 0.8710
Epoch 2/3
20000/20000 [==============================] - 5s 239us/step - loss: 0.0590 - acc: 0.9800 - val_loss: 0.3775 - val_acc: 0.8622
Epoch 3/3
20000/20000 [==============================] - 5s 237us/step - loss: 0.0033 - acc: 0.9998 - val_loss: 0.4416 - val_acc: 0.8704
```

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">MLP</th>
<th style="text-align:center" colspan = "3">真實值</th>


</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">11060</td>
<td style="text-align:center">1914</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1440</td>
<td style="text-align:center">10586</td>
</tr>
</tbody>
</table>

### 雙向GRU (Bidirectional GRU)
* GRU為LSTM的一種，速度比LSTM更快，且預測力差不多

#### **模型架構**
![](https://i.imgur.com/yibKunZ.png)

#### **程式碼**

```
# Initialize model
model <- keras_model_sequential()
model %>%
  # Creates dense embedding layer; outputs 3D tensor
  # with shape (batch_size, sequence_length, output_dim)

  layer_embedding(input_dim = max_features,
                  output_dim = 128,
                  input_length = maxlen) %>%
  bidirectional(layer_gru(units = 32)) %>%
  layer_global_max_pooling_1d() %>%
  layer_dense(units = 20, activation = "relu") %>%
  layer_dropout(rate = 0.05) %>%
  layer_dense(units = 1, activation = 'sigmoid')

# Optimizer
model %>% compile(
  loss = 'binary_crossentropy',
  optimizer = 'adam',
  metrics = c('accuracy')
)

# Train model
model %>% fit(
  x_train, y_train,
  batch_size = 100,
  epochs = 3,
  # validation_data = list(x_test, y_test)
  validation_split = 0.2
)

# Prediction & ConfusionMatrix
b_lstm_y_pred <- model %>% predict_proba(x_test)
b_lstm_y_pred <- as.numeric(b_lstm_y_pred > 0.5)
b_lstm_results <- model %>% evaluate(x_test, y_test)
b_lstm_cMatrix <- table(b_lstm_y_pred,y_test)
```

#### **訓練**
![](https://i.imgur.com/zY39jiF.jpg)

```
Train on 20000 samples, validate on 5000 samples
Epoch 1/3
20000/20000 [==============================] - 573s 29ms/step - loss: 0.3693 - acc: 0.8347 - val_loss: 0.2629 - val_acc: 0.8902
Epoch 2/3
20000/20000 [==============================] - 570s 29ms/step - loss: 0.1591 - acc: 0.9407 - val_loss: 0.3182 - val_acc: 0.8810
Epoch 3/3
20000/20000 [==============================] - 575s 29ms/step - loss: 0.0619 - acc: 0.9801 - val_loss: 0.3378 - val_acc: 0.8860
```

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">Bi_GRU</th>
<th style="text-align:center" colspan = "3">真實值</th>

</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">10779</td>
<td style="text-align:center">1341</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1721</td>
<td style="text-align:center">11159</td>
</tr>
</tbody>
</table>

### 卷積神經網路 Convolutional Neuron Network (CNN)+Bidirectional GRU
#### **模型架構**
![](https://i.imgur.com/vQE4kVZ.png)

#### **程式碼**

```
# Initialize model
model <- keras_model_sequential()
model %>%
  # Creates dense embedding layer; outputs 3D tensor
  # with shape (batch_size, sequence_length, output_dim)

  layer_embedding(input_dim = max_features,
                  output_dim = 256,
                  input_length = maxlen) %>%
  layer_conv_1d(filters = 128,
                padding = 'valid',
                kernel_size = 5) %>%
  layer_dropout(rate = 0.2) %>%
  bidirectional(layer_gru(units = 32, return_sequences = T)) %>%
  layer_global_max_pooling_1d() %>%
  layer_dense(units = 20, activation = "relu") %>%
  layer_dropout(rate = 0.05) %>%
  layer_dense(units = 1, activation = 'sigmoid')

# Optimizer
model %>% compile(
  loss = 'binary_crossentropy',
  optimizer = 'adam',
  metrics = c('accuracy')
)

# Train model
model %>% fit(
  x_train, y_train,
  batch_size = 100,
  epochs = 3,
  # validation_data = list(x_test, y_test)
  validation_split = 0.2
)

# Prediction & ConfusionMatrix
cnn_lstm_y_pred <- model %>% predict(x_test)
cnn_lstm_y_pred <- as.numeric(cnn_lstm_y_pred > 0.5)
cnn_lstm_results <- model %>% evaluate(x_test, y_test)
cnn_lstm_cMatrix <- table(cnn_lstm_y_pred,y_test)
```

#### **訓練**
![](https://i.imgur.com/a59AIiz.jpg)

```
Train on 20000 samples, validate on 5000 samples
Epoch 1/3
20000/20000 [==============================] - 581s 29ms/step - loss: 0.3985 - acc: 0.8095 - val_loss: 0.2526 - val_acc: 0.9016
Epoch 2/3
20000/20000 [==============================] - 580s 29ms/step - loss: 0.1638 - acc: 0.9393 - val_loss: 0.2626 - val_acc: 0.8952
Epoch 3/3
20000/20000 [==============================] - 584s 29ms/step - loss: 0.0729 - acc: 0.9766 - val_loss: 0.3134 - val_acc: 0.8956
```

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">CNN+Bi_GRU</th>
<th style="text-align:center" colspan = "3">真實值</th>

</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">11428</td>
<td style="text-align:center">1912</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1072</td>
<td style="text-align:center">10588</td>
</tr>
</tbody>
</table>

### Merge Multiple CNN model
#### **模型架構**
![](https://i.imgur.com/cbLeXZz.png)

#### **程式碼**

```
# Initialize model
input <- layer_input(shape = c(maxlen))
model <- input %>%
  # Creates dense embedding layer; outputs 3D tensor
  # with shape (batch_size, sequence_length, output_dim)

  layer_embedding(max_features, 300, input_length=maxlen, trainable = TRUE)

conv1 = model %>% layer_conv_1d(filters=128,
                                kernel_size =3,
                                padding ='valid',
                                activation='relu') %>%
                  layer_global_max_pooling_1d()


conv2 = model %>% layer_conv_1d(filters=128,
                                kernel_size =4,
                                padding ='valid',
                                activation='relu') %>%
                  layer_global_max_pooling_1d()


conv3 = model %>% layer_conv_1d(filters=128,
                                kernel_size =5,
                                padding ='valid',
                                activation='relu') %>%
                  layer_global_max_pooling_1d()


merge.layer <- layer_concatenate(c(conv1, conv2, conv3))


output <- merge.layer %>%
  layer_dense(units = 1, activation = 'sigmoid')


model <- keras_model(
  inputs = input,
  outputs = output
)



# Optimizer
model %>% compile(
  loss = 'binary_crossentropy',
  optimizer = 'adam',
  metrics = c('accuracy')
)

# Train model
model %>% fit(
  x_train, y_train,
  batch_size = 100,
  epochs = 3,
  validation_split = 0.2
)
# Prediction & ConfusionMatrix
m3_cnn_y_pred <- model %>% predict(x_test)
m3_cnn_y_pred <- as.numeric(m3_cnn_y_pred > 0.5)
m3_cnn_results <- model %>% evaluate(x_test, y_test)
m3_cnn_cMatrix <- table(m3_cnn_y_pred,y_test)
```

#### **訓練**
![](https://i.imgur.com/FuFvHjj.jpg)

```
Train on 22500 samples, validate on 2500 samples
Epoch 1/3
22500/22500 [==============================] - 16s 726us/step - loss: 0.3514 - acc: 0.8429 - val_loss: 0.2651 - val_acc: 0.8936
Epoch 2/3
22500/22500 [==============================] - 15s 685us/step - loss: 0.1357 - acc: 0.9516 - val_loss: 0.2513 - val_acc: 0.9056
Epoch 3/3
22500/22500 [==============================] - 15s 679us/step - loss: 0.0278 - acc: 0.9943 - val_loss: 0.2620 - val_acc: 0.9136

```

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">merge_CNN</th>
<th style="text-align:center" colspan = "3">真實值</th>

</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">11388</td>
<td style="text-align:center">1404</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1112</td>
<td style="text-align:center">11096</td>
</tr>
</tbody>
</table>

### Merge Multiple GRU model
#### **模型架構**
![](https://i.imgur.com/ohP9HC8.png)

#### **程式碼**

```
# Initialize model
input <- layer_input(shape = c(maxlen))
model <- input %>%
  # Creates dense embedding layer; outputs 3D tensor
  # with shape (batch_size, sequence_length, output_dim)

  layer_embedding(max_features, 300, input_length=maxlen, trainable = TRUE)

gru1 = model %>% layer_gru(units = 128, return_sequences = T)

gru2 = model %>% layer_gru(units = 64, return_sequences = T)

gru3 = model %>% layer_gru(units = 32, return_sequences = T)

merge.layer <- layer_concatenate(c(gru1, gru2, gru3))

output <- merge.layer %>%
  layer_global_average_pooling_1d() %>%
  layer_dense(units = 1, activation = 'sigmoid')


model <- keras_model(
  inputs = input,
  outputs = output
)



# Optimizers
model %>% compile(
  loss = 'binary_crossentropy',
  optimizer = 'adam',
  metrics = c('accuracy')
)

# Train model

model %>% fit(
  x_train, y_train,
  batch_size = 100,
  epochs = 3,
  validation_split = 0.2
)

# Prediction & ConfusionMatrix
m3_gru_y_pred <- model %>% predict(x_test)
m3_gru_y_pred <- as.numeric(m3_gru_y_pred > 0.5)
m3_gru_results <- model %>% evaluate(x_test, y_test)
m3_gru_cMatrix <- table(m3_gru_y_pred,y_test)
```

#### **訓練**
![](https://i.imgur.com/mz9gkPl.jpg)

```
Train on 22500 samples, validate on 2500 samples
Epoch 1/3
22500/22500 [==============================] - 896s 40ms/step - loss: 0.3408 - acc: 0.8475 - val_loss: 0.2549 - val_acc: 0.8984
Epoch 2/3
22500/22500 [==============================] - 899s 40ms/step - loss: 0.1502 - acc: 0.9449 - val_loss: 0.2571 - val_acc: 0.9060
Epoch 3/3
22500/22500 [==============================] - 898s 40ms/step - loss: 0.0587 - acc: 0.9817 - val_loss: 0.3179 - val_acc: 0.9020
```

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">merge_GRU</th>
<th style="text-align:center" colspan = "3">真實值</th>

</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">11233</td>
<td style="text-align:center">1669</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1267</td>
<td style="text-align:center">10831</td>
</tr>
</tbody>
</table>

### 集成式學習 Ensembling Learning
#### **方法 (多數決)**
![](https://i.imgur.com/NsQjT2R.png)

#### **混淆矩陣**

<table>
<thead>
<tr>
<th style="text-align:center">Ensemble</th>
<th style="text-align:center" colspan = "3">真實值</th>
</tr>
</thead>

<tbody>
<tr>
<td style="text-align:center" rowspan="3">預測值</td>
<td style="text-align:center"></td>
<td style="text-align:center">0</td>
<td style="text-align:center">1</td>
</tr>

<tr>
<td style="text-align:center">0</td>
<td style="text-align:center">11423</td>
<td style="text-align:center">1359</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1077</td>
<td style="text-align:center">11141</td>
</tr>
</tbody>
</table>

### 預測結果

模型|test_set<br>準確率
:---:|:---:
MLP|       86.584%
Bi-GRU|   87.752%
CNN+Bi-GRU|88.064%
merge-GRU|88.256%
merge-CNN| 89.936%
Ensembling model<br>(多數決:至少3個以上)|  90.256%
