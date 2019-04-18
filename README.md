# Kaggle_DogBreed

This repository is about gluon implement for [Dog Breed Identification](https://www.kaggle.com/c/dog-breed-identification) in kaggle.



# Files

- DataOverview.ipynb: Inspect Data.
- Preprocess.ipynb: Set directory structure.
- ExtractFeatures.ipynb: Extract features using pretrained network.
- Train_Test.ipynb: Build model. Train and Test.
- Stanford_Data_Pipeline.ipynb: Pipeline of Using [Stanford Dogs Dataset](http://vision.stanford.edu/aditya86/ImageNetDogs/) to train and test.

# Thanks

I learned a lot from [ypw's code](https://github.com/ypwhs/DogBreed_gluon). Thanks a lot.


总体思路进行介绍：
1.准备工作、
->观察训练集和测试集的图片数量和每个图片维度 
->并输出图像观察包含的每个种类狗的图像数量（可以发现每个种类狗的数量还是比较均匀的，分布在100数量的左右）。

2.对测试和训练的图片进行预处理。  （记住这里建立软连接的时候是一个个图片建立的，这是理解的突破点）
->这里会建立 软链接结构，包括训练集和测试机的软连接，建立训练集和测试集的软连接有所不同，其中测试集上的软连接是不知道分类的，所以会先放在文件0中保存，而训练集上数据因为类别是可知的，可以根据类别建立不同的文件，将图片进行保存。 
->建立软连接似的之后的处理更方便，，更建立抽象的，易于处理的结构，就像自己构建的训练集中将图片按种类放置的方式一样，不需要太多的权限。   分别构建软链接到train_gy和test_gy中去。

3.提取特征阶段。 
->先对图片进行规范化，包括对图片的规格进行限制，图片的偏转等，这样会增加网络结果的适应能力。
->这步会首先会加载要处理的数据，包括训练数据和测试数据，  目的是使用迁移过来的模型从这些数据中抽离出特征。
->抽离特征的过程是，按批次将训练数据和测试数据将迁移过来的网络中喂养，获取网络的输出层输出的每个样本对应的抽取特征， 将每个样本的大量图片特征进行抽取后，会将这些抽取出来的特征进行保存，用于以后的训练和调参时候使用（这也就是那种迁移之后内部的网络参数是不会变的，我们只是需要表面的预训练模型输出的每个样本对应的压缩特征集合）
->抽离特征工作之后，要保存的内容有 训练样本的抽取特征集合和对应的标签，  测试样本的抽取特征集合。  将这些所有东西保存到一个h5文件中去。

4.训练及产生结果。
->读取上步抽离特征之后保存的h5文件，从文件中分大类读取上部分保存的3大部分。
->从中获取训练数据特征集合  将这些特征对应集合进行分离，这里会分离20%数据量作为验证集合。
->接下来按批次，指定每个批次数量，把这所有喂养特征数据放入网络进行喂养，  这里构建的是一个简单的全连接层。
   在每个批次喂养中执行以下步骤：
       *将单个样本特征组合放入网络，获取网络输出对应的损失函数和准确率。
       *按照学习率使用优化器进行反向传播，进行参数的优化，并在一定批次后调整修改学习率
       *在一个批次之后后计算平均准确率和损失函数作为喂养数据的效果
       *将验证数据带入网络进行正向传播，得到准确率和损失函数值作为对网络验证效果，并将两者打印出来
->多次喂养后，就可以得到最终的无法提升的网络参数了，使用这些网络可以用来跑测试数据，得到跑出的结果按照格式保存到csv文件中来作为最后的提交结果。