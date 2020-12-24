Model Zoo 调研
---

主要功能
---
- 用户能够上传模型套件（提供工具）
- 用户能够下载模型套件（提供工具）
- 上传/下载的模型套件一般包含
    - 模型文件（不同框架，不同版本等）
    - 脚本
        - train脚本，inference脚本，validate脚本，其他脚本等
    - README
        - 任务场景，模型背景介绍等
        - 模型详细信息
            - framework, version, size, metrics, performance等
        - 模型输入输出
            - input格式, output格式, preprocessing, postprocessing等
        - 模型使用方法
            - train, inference，validation的使用方法
        - Reference, contributor, license等
    - Jupyter Notebook
    - 数据集
        - 训练集，太大的训练集可以只提供url
        - 测试集，大小合适的测试集可以和模型放在一起，方便验证模型效果
- 分好类的模型索引
    
不同平台的Model Zoo的特点（例子的详细信息可以从下文《实例》一节找到）
---
- 训练框架的Model Zoo
    - 一般只提供本训练框架模型格式的模型
    - 一般对训练的支持比较好
    - 也能比较方便的推理，因为主流训练框架的推理方案比较多
    - 例子
        - 3. PaddlePaddle
        - 5. PyTorchHub（Beta）
- 推理框架的Model Zoo
    - 一般只提供本推理框架模型格式的模型
    - 一般对推理的支持比较好
    - 一般对训练的支持不太好
    - 例子
        - 6. TorchServe Model Zoo（Beta）
        - 7. OpenVino Model Zoo
- 深度学习平台的Model Zoo
    - 一般对本平台的支持好，如果决定要使用该平台的话，是个不错的选择
    - 似乎对继续训练的支持程度差一些
    - 例子
        - 4. Ali PAI ModelHub公共模型仓库
        - 3. PaddlePaddle
- 深度学习工具包的Model Zoo
    - 一体化程度好，模型的下载，训练，推理基本都可以在工具包内完成
    - 通用性差一些
    - 例子
        - 2. GluonCV
- 开源的模型索引页
    - 就是简单的模型链接索引
    - 例子
        - 8. Model Zoo

实例
---
1. ONNX Model Zoo
    - 类型: 标准化的神经网络表达格式
    - 链接: https://github.com/onnx/models
    - 特点: 
        - ONNX是一个标准化的神经网络表示格式，支持它的框架比较多, 拿来直接推理很方便
        - 继续训练的话，一种是可以用模型贡献者提供的脚本，没有的话就只能自己写，似乎有支持训练onnx模型的框架
    - 模型文件夹构成(例子见https://github.com/onnx/models/tree/master/vision/classification/mobilenet):
        - 模型文件
        - Jupyter Notebook
        - Readme文件
    - 运行支持
        - 训练: 由模型的Contributor自己提供，一般是在Jupyter Notebook里
        - 推理: 首页上提供了通用的推理脚本模板(用ONNX推理其实很方便，因为主流框架大都支持)
    - 模型下载方式
        - github UI上手动下载
        - git LFS(Large File Storage)的命令行工具下载
    - 模型数量: 30多个
    - 模型种类: 
        - Vision
            - Image Classification
            - Object Detection & Image Segmentation
            - Body, Face & Gesture Analysis
            - Image Manipulation
        - Language
            - Machine Comprehension
            - Machine Translation
            - Language Modelling
        - Other
            - Visual Question Answering & Dialog
            - Speech & Audio Processing
            - Other interesting models
        
2. GluonCV
    - 类型: 计算机视觉工具库（python）
    - 链接: https://cv.gluon.ai/model_zoo/index.html
    - 特点: 
        - GluonCV是一个计算机视觉的工具包，支持大量MXNet模型和少量Pytorch模型，使用方式是调用python库函数
        - 下载模型，训练，推理，数据处理都调用python库的API直接完成
    - 模型文件夹构成(例子见): 没有模型文件夹，直接调用python gluon库的API即可
    - 运行支持
        - 训练: 官网有非常详细的tutorial，可以直接用MXNet框架
        - 推理: 官网有非常详细的tutorial，可以直接用MXNet框架
    - 模型下载方式
        - 调用python gluon库APIs
    - 模型数量: 30多个
    - 模型种类: 都是CV的
        - Classification
        - Detection
        - Segmentation
        - Pose Estimation
        - Action Recognition
        - Depth Prediction
    
3. PaddlePaddle
    - 类型: 深度学习框架
    - 链接: 下面两个模块都有模型
        - Paddle基础模型库: https://www.paddlepaddle.org.cn/modelbase
        - PaddleHub: https://www.paddlepaddle.org.cn/hub
    - 特点: 
        - 只支持PaddlePaddle框架
        - 数量超多的模型，大多是百度自己做的
        - 对中文的场景支持的好
        - 模型介绍，使用文档，脚本非常全，也很细致，可以直接在Paddle框架里开发
    - 模型文件夹构成(例子见): 没有模型文件夹，用PaddleHub命令行工具下载
    - 运行支持
        - 训练: 官网上有非常详细的tutorial，有脚本，可以直接在Paddle框架里开发
        - 推理: 官网上有非常详细的tutorial，有脚本，可以直接在Paddle框架里开发
    - 模型下载方式
        - 用PaddleHub命令行工具
        - 网页上直接下载
    - 模型数量: 特别多，数不清
    - 模型种类: 细分种类多，省略了
        - 文本
        - 图像
        - 视频
        - 语音
        
4. Ali PAI ModelHub公共模型仓库
    - 类型: 公有机器学习平台
    - 链接: https://help.aliyun.com/document_detail/183592.html?spm=a2c4g.11186623.3.2.986a3e187rSCXT
    - 特点: 
        - 可以在PAI上一键部署，好像也只能在PAI上部署，其他开源支持做的不太好
    - 模型文件夹构成(例子见): 
    - 运行支持
        - 训练: 似乎不支持再训练
        - 推理: 在PAI上一键部署
    - 模型下载方式: 无法下载
    - 模型数量: 10个左右
    - 模型种类: 
        - 图像智能处理类模型
        - 语音智能识别（ASR）类模型
        - 自然语言处理（NLP）类模型
 
5. PyTorchHub（Beta）
    - 类型: 训练框架生态
    - 链接: https://pytorch.org/
    - 特点: 
        - 主要是研究人员贡献的基于torch开发的模型，每个模型对应一个github仓库
        - 模型直接从torch库里加载，直接使用torch做后续开发
        - 为上传的模型还定制了一个简洁的tutorial页
    - 模型文件夹构成(例子见): 
        - 每个模型有一个对应的github库，实际上模型就是研究人员贡献的github库
    - 运行支持
        - 训练: 直接用torch，查看模型的原github库可以找到更多信息
        - 推理: 直接用torch，查看模型的原github库可以找到更多信息
    - 模型下载方式
        - 可以直接用python的torch.hub库load模型
    - 模型数量: 34个
    - 模型种类: 没有分类
    
6. TorchServe Model Zoo（Beta）
    - 类型: 推理框架
    - 链接: 
        - TorchServe: https://github.com/pytorch/serve
        - TorchServe Model Zoo: https://pytorch.org/serve/model_zoo.html
    - 特点: 
        - 只能配套TorchServe推理框架使用
        - 由于是为推理平台做的模型市场，所以主要面向推理场景，没找到继续训练的方法
        - 模型市场的首页很简洁，就是一个表格，列了模型和其他信息
    - 模型文件夹构成: 
        - 1个.mar文件，可以直接feed到TorchServe里推理
    - 运行支持
        - 训练: 模型格式应该只有TorchServe支持，应该不能训练
        - 推理: 需要用TorchServe推理
    - 模型下载方式
        - 直接下载（UI上点或者wget）
    - 模型数量: 20个
    - 模型种类: 没有分类，多数是CV的
        
7. OpenVino Model Zoo
    - 类型: 推理框架
    - 链接: https://github.com/openvinotoolkit/open_model_zoo
    - 特点: 
        - 主要目标是提供在OpenVino上可推理的模型
        - 下载工具可以下载很多从开源社区收集来的模型，但是没有太多说明，好像也没啥用，用户完全可以自己找
    - 模型文件夹构成: 分两类
        - intel: 这类下载下来是OpenVino的模型格式
        - public: 没有固定模式，从开源社区收集来的
    - 运行支持
        - 训练: public的模型可以原框架上训练，没有任何说明
        - 推理: intel的模型可以直接推理，有统一的tutorial，public的模型可以用OpenVino的转换器转换后进行推理
    - 模型下载方式
        - 提供一个下载脚本
    - 模型数量: 挺多，没有细数
    - 模型种类: 
        - Object Detection Models
        - Object Recognition Models
        - Reidentification Models
        - Semantic Segmentation Models
        - Instance Segmentation Models
        - Human Pose Estimation Models
        - Image Processing
        - Text Detection
        - Text Recognition
        - Action Recognition Models
        - Compressed Models
    
8. Model Zoo
    - 类型: 一个开源的github开源模型集合页
    - 链接: https://modelzoo.co/
    - 特点:
        - 一个开源的的github开源模型集合页，只提供链接到模型的github仓库
        - 模型数量非常大
    （其他略，需要自己跳转到github仓库查看具体信息）