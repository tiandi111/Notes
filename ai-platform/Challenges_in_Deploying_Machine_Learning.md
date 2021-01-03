论文笔记 Challenges in Deploying Machine Learning
---

##2. Machine Learning Deployment Workflow
- 数据管理：准备用于模型训练的数据
- 模型学习：model selection and training
- 模型验证：ensure that models adhere to certain functional and performance requirements
- 模型部署：integrate trained models into the software infrastructure, model maintenance, model updates

## 3. 数据的管理
### 3.1 数据收集(Data Collection)
数据收集阶段，主要的工作是发现和理解(discover and understand)哪些数据是可以获得的，以及如何去组织方便可用的存储来存这些数据。

这一阶段最主要的挑战在于寻找数据源以及理解数据的结构。在软件工程里有一条非常重要的原则，"单一职责原则"，即每个模块的功能应该做到尽可能的
内聚，单一，减少模块间耦合。遵循这个原则，生产环境经常会把一个整体服务拆分成很多单一的，细小的模块，模块之间通过网络通讯进行联系。这个实践
对机器学习数据收集带来的挑战在于数据源非常分散，格式不统一以及数据缺失。

### 3.2 数据前处理(Data Preprocessing)
作者略过了常见的数据前处理的一些方法和技术，着重讨论了"Data dispersion"即"数据分散"这个现象，即生产环境经常会有很多相关联的数据，但是
这些数据常常有不一样的scheme, convention以及独立的数据存取方法（例如有的在数据库里，有的在消息队列里，有的在日志里等）等。合并这些数据
带来了较大的挑战。

### 3.3 数据增强(Data Augmentation)
进行数据增强的原因有很多，但是最problematic的原因是缺少标注好的数据。导致标注数据稀缺的三个最主要的原因是：
- 巨量的数据<br/>
    例如network traffic analysis, 由于网络的流量非常大，给标注造成困难。
- 缺少专家<br/>
    例如医疗图像的分析。没有那么多专家天天标注数据。
- 缺少多样化(high-variance)的数据<br/>
    通常在增强学习领域里碰到这个问题，当把模型从实验室环境迁移到真实环境时，因为训练模型的数据来源比较单一（通常是实验室环境），模型可能
    在真实环境表现并不好。例如自动驾驶。
    
### 3.4 数据分析(Data Analysis)
数据需要经过分析后才能发现可能存在的偏差和unexpected distribution。通常这些分析工具需要借助高质量的分析工具。实践中人们发现的一个非常
有挑战性的领域是data profiling。这个领域亟需高质量的分析工具。

## 4. 模型学习(Model Learning)
### 4.1 模型选择(Model Selection)
实际应用中，选择模型的一个关键因素是模型的复杂性。尽管深度学习大行其道，但是现实生产环境仍然会使用一些诸如浅层神经网络或者PCA之类的基础机
器学习算法，原因在于部署复杂的深度学习模型会遇到诸多挑战和困难。

首先是部署简单的模型可以用于验证Proof of Concept的机器学习系统，而复杂模型会带来很多复杂度。来自Air B&B的研究人员介绍了一个例子，他们
在上线一个机器学习系统时，首先使用了复杂的深度学习模型，然而很快他们就发现自己没法handle复杂模型带来的各种问题，开发周期也被大量消耗，最后
他们将模型简化成一层全连接网络，这帮助他们成功地上线了整条部署模型的流水线，同时允许他们逐步的迭代系统，使用更复杂的模型。详见论文
Applying deep learning to AirBnB search。

另一个限制在于真实的部署环境中，计算资源，硬件资源是有限的。例如移动设备，航天设备等。

第三个原因在于，对于有些业务场景，模型能够被解释是非常重要的。例如一些金融业务场景。

### 4.2 训练(Training)
关于模型训练的最大担忧在于其经济成本。作者以NLP及NAS技术做了举例，两个有趣的数据，训练一个BERT模型的成本在5万~160万美金之间。一个完整的
NAS训练周期造成的二氧化碳释放量和四辆汽车全部生命周期所排放的二氧化碳量相当。

### 4.3 超参选择(Hyper-parameter selection)
机器学习模型经常会定义超参数，为了获得更好的模型性能，研究人员经常采用超参搜索技术。这带来两方面的挑战，一方面是超参搜索会造成高昂的经济
成本，另一方面是对于某些部署场景（例如移动端部署）超参搜索还需要考虑到具体的硬件环境，所以对硬件感知的超参搜索技术的需求在提升。

## 5. 模型验证(Model Verification)<br/>
模型验证的目标包括：模型应该能够对unseen data有较好的泛化能力，能够处理edge case，鲁棒性较高，并且能能够满足所有功能性需求。

### 5.1 需求编码(Requirement encoding)
为机器学习模型定义需求是非常重要的一步，因为有些场景下，常用的metrics，例如准确性等，不能很好的体现模型对实际业务场景的影响。

### 5.2 正式验证(Formal Verification)
Formal Verification指的是对模型功能性是否满足当前项目范畴内的要求的验证。作者举了一个特例是在某些场景下，模型需要满足监管要求，例如银
行的许多业务场景。

### 5.3 基于测试的验证(Test-based Verification)
基于测试的验证目标是验证模型能够较好的泛化到unseen data上。这里有两个重要的挑战。

第一个挑战是如何让模拟测试环境更接近与真实测试环境。理想情况肯定是直接在真实环境里测，但一般不这么做，因为有比较多的限制，比如自动驾驶直接
在公路上测试可能有安全隐患。生产环境直接在线上测试可能造成损失等。

第二个挑战在于对数据集本身的验证（以保证数据集里的error不会直接传播到线上去）。许多因素可能导致数据集的不一致，例如代码bug产生了不正确的
数据，数据依赖的改变等。

## 6. 模型部署(Model Deployment)
### 6.1 集成(Integration)
集成包括两部分，一部分是构建机器学习的基础架构，另一部分是模型本身的实现应该让其能够被所在的基础架构环境容易的吸收和支持。作者主要讨论了
第二部分。

第一个挑战是代码复用。作者举了一个Pinterest的例子，Pinterest内部有三个模型用了类似的embedding，导致同样的工作被重做三次，降低了效率。
（这里其实还有一个代码可复用的点就是训练推理代码复用）

第二个挑战是很多机器学习模型的代码实现是"anti-pattern"的，就是不符合通常的软件工程的原则和习惯。造成这个的原因主要是因为模型代码相比于
传统软件工程代码，增加了对外部数据的依赖。一些常见的现象包括，抽象边界的划分不清晰，"胶水代码(glue code)"，流水线杂乱无章，配置债务等。
详情可见这篇文章https://zhuanlan.zhihu.com/p/102897536或这篇论文Hidden technical debt in machine learning systems.

第三作者提出了一个建议，就是研究人员应当和工程师工作于同一份codebase之上，没有具体说明理由，详见这篇论文Software engineering for
 machine learning: A case study。

### 6.2 监控(Monitoring)
作者指出整个社区对于监控数据集和模型的关键指标的理解还处在一个早期阶段的。作者指出监控不断变化的输入数据，监控预测偏差以及模型的整体表现
是一个open problem。见论文Hidden technical debt in machine learning systems.

Monitoring and explainability of models in production这篇论文指出了用异常值监控作为模型无法用于生产环境的信号的重要性。两个原因，
一是异常值可以指出模型欠缺的泛化能力，二是因为poor calibration造成的对于out-of-distribution instance的过于自信的预测。

Deploying machine learning models for public policy: A framework这篇文章指出早期干预系统early intervention system (EIS)的重要
性。乍一看，模型监控应该是非常标准化的，例如数据的完整性检验，异常值检测和性能指标等。但实际应用中这往往包含很多的定制化的需求，这导致很多
标准化的工具在生产环境里无法使用。

### 6.3 更新(Updating)
实际用于中，模型通常需要持续迭代，保证模型吸收了新数据中的信息，保证其效果。这个过程也通常存在一些挑战。

第一个挑战是concept drift，如果无法检测到concept drift，可能对模型的表现造成较大影响。concept drift指的是随着时间的推移，模型遵循的
distribution也通常在变化。造成这个现象的原因有很多，例如公司战略策略的变化，行业规则的改变，外部重大事件的发生等。(concept drift在监控
里也算一个挑战)

第二个挑战是模型的持续部署(CD)流水线的建设。与传统软件工程的CD不一样，模型的依赖除了代码以外，还有数据集和模型。这2篇文章
https://martinfowler.com/articles/cd4ml.html 和 https://www.thoughtworks.com/insights/blog/getting-smart-applying-continuous-delivery-data-science-drive-car-sales
介绍了相关经验。