# 🤳CLIP进行ZERO-SHOT分类的过程🤳
---

## ✔使用CLIP的Zero-Shot分类过程图

![训练](https://user-images.githubusercontent.com/128795948/227481912-f99f75b7-6a3d-4502-a8c4-8a41e0216f18.png)

## ✔CLIP模型

1.训练阶段
CLIP模型架构被分为两部分：  
（1）图像编码器  
（2）文本编码器。  

当我们对CLIP模型提供一组文本描述，这些文本描述会被编码为文本嵌入。  
然后，我们对图像重复这一输入步骤，那么图像也会被编码成图像嵌入。
  
2.测试阶段  
CLIP模型会计算图像嵌入和文本嵌入之间的成对余弦相似度。  
其中矩阵内余弦相似度具最高的文本提示会被选择为预测结果，这样就完成目标任务上的 zero-shot 分类。


同时，输入也可以不止输入一张图像，CLIP缓存了输入的文本嵌入，所以它们不必为其余的输入图像重新进行计算。  
这是CLIP端到端的工作方式。  

---

在这样的模型中，图像编码器通常用来提取图像的特征向量，而自然语言编码器则用来将自然语言序列转换为可以被计算机处理的向量。  
使用图像编码器和自然语言编码器来预测一段自然语言文本对应的图像时，我们会首先将图像输入到图像编码器中，得到一个向量 VImage。  
然后，我们将需要预测的文本输入到自然语言编码器中，得到另一个向量 VLanguage。  

其中， VLanguage是由我们输入的自然语言文本序列所产生的，这些输入可以是单词、句子、段落、文档或其他基于文本的形式，并通常由分类、情感分析、问答系统或类似的任务中的文本输入得到。  

---

深度学习模型中，自然语言编码器通常由多个循环神经网络（RNN）层构成，它们可以接受一个自然语言序列，并输出一个针对该序列的特征向量。这个特征向量通常被称为“上下文向量”，因为它包含了关于序列中每个词的上下文信息。 

因此，当我们输入一个自然语言序列到自然语言编码器中时，每个词会被依次输入到 RNN 中，并且每个 RNN 层都会输出一个中间状态，用于在后续的计算中继续传递。 

最终，自然语言编码器输出的向量 VLanguage 就是通过在这些中间状态上进行加权求和得到的，这样可以捕捉到整个文本序列的特征信息，而不仅是每个单独的词语信息。  

它包含了针对该序列的所有上下文信息，可以用于通过与图像编码器得到的向量 VImage 进行联合计算，得到我们需要的预测结果。

---

## ✔环境配置过程：

首先完成Anaconda、CUDA等的适配后，再进行环境适配。

conda添加国内源:

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/

显示添加的源：

    conda config --show channels

pip 换源：

    pip config set global.index-url https://pypi.douban.com/simple/

然后创建虚拟环境，打开Anaconda PowerShell Prompt输入：
  
      conda create -n {torch.版本} python={版本}，
  
  创建成功后，激活虚拟环境:
  
      conda activate torch1.1
  
---

## ✔测试操作过程

从github克隆下CLIP-main项目后，传输入自己的anaconda虚拟空间中，在文件夹下创建自己的py项目如：Envelope(Pred).py等诸如此类。

在pycharm里打开自己新建的文件项目，导入你的代码。

    import torch
    import clip
    from PIL import Image  

    device = "cuda" if torch.cuda.is_available() else "cpu"
    model, preprocess = clip.load("ViT-B/32", device=device)  
    
    image = preprocess(Image.open("CLIP.png")).unsqueeze(0).to(device)
    text = clip.tokenize(["a diagram", "a dog", "a cat"]).to(device)  

    with torch.no_grad():
    image_features = model.encode_image(image)
    text_features = model.encode_text(text)  
    
    logits_per_image, logits_per_text = model(image, text)
    probs = logits_per_image.softmax(dim=-1).cpu().numpy()  
    
    print("Label probs:", probs)  # prints: [[0.9927937  0.00421068 0.00299572]]  

（此代码旨在调整pytorch与torchvision与计算机上的 CUDA 版本适配，或者是在没有 GPU 的计算机上进行安装时使用）

———————————————————————————————————————————————————————————————

## ✔Zero-Shot预测红包图片以及更多拓展

下面的代码使用 CLIP 执行零镜头预测，在执行前在任意平台选择一张红包图片导入CLIP-main虚拟空间文件夹中，并再次新建一个py文件，输入代码：

    import os
    import clip
    import torch
    from PIL import Image
    
    from torchvision.datasets import CIFAR100  

    # Load the model
    device = "cuda" if torch.cuda.is_available() else "cpu"
    model, preprocess = clip.load('ViT-B/32', device)  


    # Prepare the inputs
    image = preprocess(Image.open("OP.jfif")).unsqueeze(0).to(device)
    a_list = ['car','rubbish','father','friend']
    text_inputs = torch.cat([clip.tokenize(f"a photo of a {c}") for c in a_list]).to(device)  
    
    # Calculate features
    with torch.no_grad():  
    image_features = model.encode_image(image)
    text_features = model.encode_text(text_inputs)  

    # Pick the top 5 most similar labels for the image
    image_features /= image_features.norm(dim=-1, keepdim=True)
    text_features /= text_features.norm(dim=-1, keepdim=True)
    similarity = (100.0 * image_features @ text_features.T).softmax(dim=-1)
    values，indices = similarity[0].topk(1)  

    # Print the result
    print("\nTop predictions:\n")
    for value, index in zip(values, indices):
    print(f"{a_list[index]:>16s}: {100 * value.item():.2f}%")  
    
测试完成之后，你应该能得到一个预测值--->
![1](https://user-images.githubusercontent.com/128795948/227507990-8ea63d88-5241-4b9c-ab08-c68ebeef31d7.PNG)
  
而当输入的自然语言中有red envelope这个复合词条后，CLIP在red envelope的预测值甚至直接达到99.92%  
![4](https://user-images.githubusercontent.com/128795948/227509984-3dece91e-859b-4878-9f1d-d4cd59f0ffcf.PNG)

---

## 👀拓展：现在尝试新建一个识别杯子的二分类任务

重复上述操作导入image数据，建立自己的py文件，并打开Pycharm进行操作，之前识别红包的代码基本改变classes就能识别另一个目标。  

    import os
    import clip
    import torch
    from torchvision.datasets import CIFAR100
    from PIL import Image  

    img_pah = 'CUP.jfif'
    classes = ['cup', 'not cup']  

    # Load the model
    device = "cuda" if torch.cuda.is_available() else "cpu"
    model, preprocess = clip.load('ViT-B/32', device)  


    # Prepare the inputs
    image = Image.open(img_pah)
    image_input = preprocess(image).unsqueeze(0).to(device)
    text_inputs = torch.cat([clip.tokenize(f"a photo of a {c}") for c in classes]).to(device) #生成文字描述  

    # Calculate features
    with torch.no_grad():
    image_features = model.encode_image(image_input)
    text_features = model.encode_text(text_inputs)  

    # Pick the top 5 most similar labels for the image
    image_features /= image_features.norm(dim=-1, keepdim=True)
    text_features /= text_features.norm(dim=-1, keepdim=True)
    similarity = (100.0 * image_features @ text_features.T).softmax(dim=-1) #对图像描述和图像特征
    values, indices = similarity[0].topk(1)  

    # Print the result
    print("\nTop predictions:\n")
    print('classes:{} score:{:.2f}'.format(classes[indices.item()], values.item()))  

导入图片后，编辑代码输入自然语言为cup与not_cup进行预测得到结果：

![2](https://user-images.githubusercontent.com/128795948/227509510-70836c6f-ca31-4fc2-a8ca-b00abbe99f5c.PNG)  

注意：CLIP在预测中易受到多义词的影响。有时，由于缺乏上下文，该模型无法区分一些词的含义，所以在进行预测时输入的自然语言对预测值将会有很大影响。  
同时CLIP在手写数字识别任务上很吃力，因为其训练数据集中缺乏手写数字。

---
