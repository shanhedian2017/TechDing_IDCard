### CCF大数据与计算智能大赛之《基于OCR的身份证要素提取》复赛A榜12名，B榜14名：TechDing，方案介绍：

大赛整体介绍请看官方链接：https://www.datafountain.cn/competitions/346

### 我的解决步骤：

1.get_IDImg获取的imgs_folder中所有原始图像的正面反面图像，并使用分类模型来预测出图像是正面正立，倒立，反面正立，倒立哪一种，并将所有倒立图片旋转为正立。

2.从所有正立图像中获取出文本所在区域ROI，有两种方法：前期我用目标检测算法YOLO3来获取出各个重要文本所在的区域，但发现这种方法有些漏检，且得到的结果也不太理想。
    后期我直接用固定像素范围来做roi提取，比如姓名区域最长为3个汉字，所以取3个汉字区域，如果是2个汉字的后面一部分就是空白，比如对地址，选择三行的区域。
    这种方法对于地址和反面的签发单位需要判断一下，可以训练一个二分类模型查看每一行第一个汉字是否为空白，如果不是空白，表示这一行有汉字，还需要截取这一行的区域来预测。

3. 使用各种不同的文本识别模型来识别roi中的文本，模型为CRNN，使用CTC作为loss function，故而可以得到多个不同CRNN模型。
    对于民族，一共只有26个类别，故而此处可以用分类模型来对民族ROI进行分类，准确率可能会更高一些，同样，对于性别，只有两个类别，故而用分类模型结果会更准确。

4. 文本修正：为了提高准确率，识别出来的文本要进行一些修正，修正方法多种多样：比如对于生日和身份证号中间的几位数字是一一对应的，故而两者相互印证可以修改错误日期。
    还比如：身份证号的最前面几个数字是出生地的省份城市，可以将其和地址进行相互联动来修正。（这种方法可以提高竞赛的得分，但实际生活中这种trick不可取，
    因为身份证号是出生时的地址，但正面的地址是迁移户口后再办理身份证时所在的地址，所以这种方法在实际情况下慎用。），还比如：地址和签发单位是一一对应的，可以联动修正。

### 我的一些tricks：

1. 由于所提供的图片不太可能覆盖所有的地址和汉字，故而我用程序来生成一些带有地址的图片，比如，收集全国所有地名组成一个txt，然后将身份证正面的地址抹去组成多个模板，
    再用程序将多个地址写入到该图片对应位置处，可以产生无限多个图片样本。对于其他信息，比如年龄，身份证号等，也可以用同样的方法生成无限多个图像样本，再在随机位置打水印即可。
    实际上，这个身份证图像数据集应该也是用程序来生成的，再打上水印，而不是收集的真实身份证图片。

2. 对于CRNN要识别的图像，考虑长宽比，如果太长了，可以修改CRNN的kenel size，使其更加不均衡。

3. 准备训练集时，不仅仅用官方提供的训练集，还可以用程序生成自己想要的其他各种图片，然后对这些图片做增强处理，提高模型的泛化能力。

4. 在训练样本中，特意加入了没有任何文本的背景图片，并用特殊字符作为label，防止出现一些特殊的没有任何文本的图片。

#### 数据集请去官方网站上下载。我训练的一些模型的下载地址：百度网盘：链接：链接：https://pan.baidu.com/s/1TvxBoZUuC4AJeYEr95Mucg 
提取码：cpl1
