使用MNIST手写数字数据集，该数据集有6万张手写数字的灰度图，每张图像的分辨率为28*28
直接上Python3代码：

```python
# loading data
from GCForest import gcForest
from sklearn.metrics import accuracy_score, recall_score, f1_score
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split


from tensorflow import keras
(X_train,Y_train),(X_test,Y_test) = keras.datasets.mnist.load_data()
print(np.shape(X_train))
# 这里784=28*28，也就是图像的高乘以宽
X_train = X_train.reshape(60000, 784)
X_test = X_test.reshape(10000, 784)
# 归一化处理
X_train = X_train/255
X_test = X_test/255
# n_jobs=-1意思是开启CPU的所有线程并行计算，提高计算速度
model = CascadeForestClassifier(random_state=1,n_jobs=-1)
model.fit(X_train, Y_train)
Y_pred = model.predict(X_test)
acc = accuracy_score(Y_test, Y_pred) * 100
rcc = recall_score(Y_test, Y_pred, average='macro')
f1 = f1_score(Y_test, Y_pred, average='macro')
print("\nTesting Accuracy: {:.3f} %".format(acc))
print("\nTesting Recall: {:.3f}".format(rcc))
print("\nTesting f1 score: {:.3f}".format(f1))
```
注意，上述代码并未运行多粒度扫描，仅仅训练了级联森林,文中直接使用南大的DF21开源Python库，该库不包含多粒度扫描实现（使用老版本代码中的多粒度扫描测试仅有2%至4%左右的准确率提升）。
效果：

```powershell
(60000, 28, 28)
[2022-08-18 01:15:15.157] Start to fit the model:
[2022-08-18 01:15:15.157] Fitting cascade layer = 0 
[2022-08-18 01:15:44.726] layer = 0  | Val Acc = 97.102 % | Elapsed = 29.569 s
[2022-08-18 01:15:45.032] Fitting cascade layer = 1 
[2022-08-18 01:16:13.678] layer = 1  | Val Acc = 97.515 % | Elapsed = 28.646 s
[2022-08-18 01:16:13.910] Fitting cascade layer = 2 
[2022-08-18 01:16:41.726] layer = 2  | Val Acc = 97.718 % | Elapsed = 27.815 s
[2022-08-18 01:16:41.949] Fitting cascade layer = 3 
[2022-08-18 01:17:09.523] layer = 3  | Val Acc = 97.793 % | Elapsed = 27.574 s
[2022-08-18 01:17:09.744] Fitting cascade layer = 4 
[2022-08-18 01:17:37.920] layer = 4  | Val Acc = 97.855 % | Elapsed = 28.176 s
[2022-08-18 01:17:38.138] Fitting cascade layer = 5 
[2022-08-18 01:18:06.130] layer = 5  | Val Acc = 97.852 % | Elapsed = 27.992 s
[2022-08-18 01:18:06.131] Early stopping counter: 1 out of 2
[2022-08-18 01:18:06.349] Fitting cascade layer = 6 
[2022-08-18 01:18:33.740] layer = 6  | Val Acc = 97.887 % | Elapsed = 27.390 s
[2022-08-18 01:18:33.975] Fitting cascade layer = 7 
[2022-08-18 01:19:01.380] layer = 7  | Val Acc = 97.882 % | Elapsed = 27.405 s
[2022-08-18 01:19:01.381] Early stopping counter: 1 out of 2
[2022-08-18 01:19:01.603] Fitting cascade layer = 8 
[2022-08-18 01:19:29.285] layer = 8  | Val Acc = 97.923 % | Elapsed = 27.682 s
[2022-08-18 01:19:29.504] Fitting cascade layer = 9 
[2022-08-18 01:19:56.809] layer = 9  | Val Acc = 97.937 % | Elapsed = 27.304 s
[2022-08-18 01:19:57.040] Fitting cascade layer = 10
[2022-08-18 01:20:25.001] layer = 10 | Val Acc = 97.913 % | Elapsed = 27.962 s
[2022-08-18 01:20:25.001] Early stopping counter: 1 out of 2
[2022-08-18 01:20:25.228] Fitting cascade layer = 11
[2022-08-18 01:20:52.843] layer = 11 | Val Acc = 97.907 % | Elapsed = 27.614 s
[2022-08-18 01:20:52.843] Early stopping counter: 2 out of 2
[2022-08-18 01:20:52.843] Handling early stopping
[2022-08-18 01:20:52.870] The optimal number of layers: 10
[2022-08-18 01:20:52.896] Start to evalute the model:
[2022-08-18 01:20:53.077] Evaluating cascade layer = 0 
[2022-08-18 01:20:53.525] Evaluating cascade layer = 1 
[2022-08-18 01:20:53.988] Evaluating cascade layer = 2 
[2022-08-18 01:20:54.450] Evaluating cascade layer = 3 
[2022-08-18 01:20:54.913] Evaluating cascade layer = 4 
[2022-08-18 01:20:55.376] Evaluating cascade layer = 5 
[2022-08-18 01:20:55.836] Evaluating cascade layer = 6 
[2022-08-18 01:20:56.299] Evaluating cascade layer = 7 
[2022-08-18 01:20:56.762] Evaluating cascade layer = 8 
[2022-08-18 01:20:57.225] Evaluating cascade layer = 9 

Testing Accuracy: 98.170 %

Testing Recall: 0.981

Testing f1 score: 0.982
```
