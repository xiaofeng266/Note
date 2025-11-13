# TensorRT

参考教程：

https://blog.csdn.net/qq_39523365/article/details/126124010

https://blog.csdn.net/baidu_32631625/article/details/143442736



## 1.什么是TensorRT

TensorRT是可以在NVIDIA各种GPU硬件平台下运行的一个C++**推理框架**。

我们利用Pytorch、TF或者其他框架训练好的模型，可以转化为TensorRT的格式，然后利用TensorRT推理引擎去运行我们这个模型，从而提升这个模型在英伟达GPU上运行的速度（**模型提速**）


## 2.TensorRT的加速效果

加速效果取决于模型的类型和大小，也取决于我们所使用的显卡类型。

对于GPU来说，因为底层的硬件设计，更适合并行计算也更喜欢密集型计算。TensorRT所做的优化也是基于GPU进行优化，当然也是更喜欢那种一大块一大块的矩阵运算，尽量直通到底。因此对于通道数比较多的卷积层和反卷积层，优化力度是比较大的；如果是比较繁多复杂的各种细小op操作(例如reshape、gather、split等)，那么TensorRT的优化力度就没有那么夸张了。

为了更充分利用GPU的优势，我们在设计模型的时候，可以更加偏向于模型的并行性，因为同样的计算量，“大而整”的GPU运算效率远超“小而碎”的运算。

工业界更喜欢简单直接的模型和backbone（主干网络）。2020年的RepVGG(RepVGG：极简架构，SOTA性能，让VGG式模型再次伟大（CVPR-2021）)，就是为GPU和专用硬件设计的高效模型，追求高速度、省内存，较少关注参数量和理论计算量。相比resnet系列，更加适合充当一些检测模型或者识别模型的backbone。

（补充知识：模型包括主干网络backbone和头部网络head，head用于接收backbone提取好的高级特征，根据具体任务做出最终决策）

在实际应用中，也简单总结了下TensorRT的加速效果：

SSD检测模型，加速3倍(Caffe)
CenterNet检测模型，加速3-5倍(Pytorch)
LSTM、Transformer(细op)，加速0.5倍-1倍(TensorFlow)
resnet系列的分类模型，加速3倍左右(Keras)
GAN、分割模型系列比较大的模型，加速7-20倍左右(Pytorch)



## 3.什么模型可以转换为TensorRT

TensorRT官方支持Caffe、Tensorflow、Pytorch、ONNX等模型的转换(不过Caffe和Tensorflow的转换器Caffe-Parser和UFF-Parser已经有些落后了)，也提供了三种转换模型的方式：

使用TF-TRT，将TensorRT集成在TensorFlow中
使用ONNX2TensorRT，即ONNX转换trt的工具
手动构造模型结构，然后手动将权重信息挪过去，非常灵活但是时间成本略高，有大佬已经尝试过了：tensorrtx
不过目前TensorRT对ONNX的支持最好，TensorRT-8最新版ONNX转换器又支持了更多的op操作。而深度学习框架中，**TensorRT对Pytorch的支持更为友好**，除了Pytorch->ONNX->TensorRT这条路，还有：

torch2trt
torch2trt_dynamic
TRTorch
总而言之，理论上95%的模型都可以转换为TensorRT，条条大路通罗马嘛。只不过有些模型可能转换的难度比较大。如果遇到一个无法转换的模型，先不要绝望，再想想，再想想，看看能不能通过其他方式绕过去。

## 4.TensorRT是硬件相关的

因为不同显卡(不同GPU)，其核心数量、频率、架构、设计(还有价格…)都是不一样的，TensorRT需要对特定的硬件进行优化，不同硬件之间的优化是**不能共享**的。



ONNX(ONNX是一个模型结构格式，方便不同框架之间的模型转化例如Pytorch->ONNX->TRT)



## 5.TensorRT的缺点

TensorRT不是没有“缺点”的，有一些小小的缺点需要吐槽一下：

经过infer优化后的模型与特定GPU绑定，例如在1080TI上生成的模型在2080TI上无法使用；
高版本的TensorRT依赖于高版本的CUDA版本，而高版本的CUDA版本依赖于高版本的驱动，如果想要使用新版本的TensorRT，更换环境是不可避免的；
TensorRT尽管好用，但推理优化infer还是闭源的，像深度学习炼丹一样，也像个黑盒子，使用起来会有些畏手畏脚，不能够完全掌控。所幸TensorRT提供了较为多的工具帮助我们调试。
