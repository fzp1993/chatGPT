# MiniGPT-4: Enhancing Vision-Language Understanding with Advanced Large Language Models

## 作者
Deyao Zhu，Jun Chen，Xiaoqian Shen，Xiang Li，Mohamed Elhoseiny

## 刊物


## 解决问题
* GPT-4具有先进的多模态生成能力的主要原因是使用了更高级的大型语言模型（LLM）
* 使用来自公共数据集的原始图像-文本对将视觉特征与大型语言模型对齐并不足以开发一个性能良好的MiniGPT-4模型。它可能会产生缺乏一致性的非自然的语言输出，包括重复和分散的句子

## 创新点
* 提出了MiniGPT-4，它只使用一个投影层，将冻结的视觉编码器（BLIP-2）与冻结的LLM（Vicuna）对齐
* MiniGPT-4具有许多类似于GPT-4的功能以及新功能（根据图像创作诗歌）
* 在第二阶段管理了一个高质量、对齐的数据集，使用会话模板来微调我们的模型，对于增强模型的生成可靠性和整体可用性至关重要

## 思路概述
* 语言解码器是建立在LLaMA之上的预训练Vicuna，视觉编码器是ViT-G/14的预训练Q-Froter，添加了一个线性投影层，以将编码的视觉特征与Vicuna语言模型对齐，弥合两个模型之间的差距，并冻结了所有其他的视觉和语言组件
* 在4张A100 GPU上，使用256batch size训练20000步，10个小时。初始阶段使用LAION、Conceptual Captions、SBU三个数据集的500万对数据来训练视觉和语言模型。第二阶段使用额外的3500对高质量aligned image-text pairs，并使用设计的会话模板来微调模型，提升自然性和可用性
  ### 生成微调数据集并微调
  * 从Conceptual Caption数据集里面随机选取5000张图
  * 使用Vicuna风格的prompt，对每张图生成80个单词的描述
  ```
  ###Human: <Img><ImageFeature></Img> Describe this image in detail.Give as many details as possible. Say everything you see. ###Assistant:

  ###Human: Continue ###Assistant:
  ```
  * 生成的描述有噪声，使用chatgpt来细化，以下是prompt
  ```
  Fix the error in the given paragraph. Remove any repeating sentences, meaningless characters, not English sentences, and so on. Remove unnecessary repetition. Rewrite any incomplete sentences. Return directly the results without explanation. Return directly the input paragraph if it is already correct without explanation.
  ```
  * 人工验证描述的正确性，会检查每个生成的图像描述是否遵循我们所需的格式，并通过消除ChatGPT无法检测到的冗余单词或句子来手动细化生成的标题。选取3500条数据
  * 使用以下prompt进行微调，\<Instruction>是预定义的指令集中随机采样的指令，该指令包含不同形式的指令，如“详细描述这张图像”或“你能为我描述这张图像的内容吗”。需要注意的是，我们不计算这个特定的文本-图像提示符的回归损失
  ```
  ###Human: <Img><ImageFeature></Img> <Instruction> ###Assistant:
  ```
  * 需要400步训练，batch size大小为12，使用一个A100 GPU需要7分钟来完成
  
![minigpt4模型图](/pic/minigpt4.png "minigpt4模型图")

## 数据集

## 评价指标

## 结论和实验结果

## 存在问题
* 语言幻觉（Language hallucination）：生成假信息
* 知觉能力不足（Inadequate perception capacities）难以从图像中识别详细的文本信息，并区分空间定位

## 一些感想
* 个人认为对于数据集的处理生成提供一些思路
* prompt是一个微调的新方法，不会改变模型参数的微调