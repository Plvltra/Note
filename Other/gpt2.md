![[original gpt model.png]]

- gpt.c 参数说明
![[gpt2 parameter.png]]

- World Embedding(文本表征)： https://zhuanlan.zhihu.com/p/643560252
	- 目标
		- 性质一: 保证每一个词具有唯一量化值
		- 性质二: 词义相近词需要有相近的量化值，不相近的词量化值需要尽量远离
	-  优点
		- 帮助更加高效理解语义(性质二使相近语义句子表征向量相似)
		- 允许计算模型的设计有更大的自由度(可以让输出的连续值使用相近离散值, 若不满足性质二这种做法便不再合理)

- 临时笔记
	- B = batch_size, T = current seq_len, maxT = sequence_length, C = channels, V = vocab_size. For example, B=8(in this case B=1), maxT=1024, C=768, V=50257, L = num_layers(blocks), NH = num_heads
	- acts.encoded is (B,T,C). At each position (b,t), a C-dimensional vector summarizing token & position
	- inp is (B,T) of integers, holding the token ids at each (b,t) position
	- wte is (V,C) of token embeddings, short for "weight token embeddings"
	- wpe is (maxT,C) of position embeddings, short for "weight positional embedding"
	- fcf → fc, att → attn
	- **ParameterTensors（参数张量）&& ActivationTensors（激活张量）**
	    - ParameterTensors是模型中需要训练的参数，包括权重和偏置。
	    它们是模型学习的核心，通过优化算法（如梯度下降）来更新。
	    在GPT-2中，参数张量包括所有层的权重矩阵和偏置向量。
	    它们在整个训练过程中保持一致，直到训练完成。
	    - ActivationTensors是模型在前向传播过程中计算得出的中间结果。
	     它们代表每一层的输出，应用非线性激活函数后的结果。
	    激活张量不是模型的直接学习参数，而是用于计算损失并进行反向传播。
	    它们会在每次输入数据时变化，并且是暂时存在的，不会在模型训练结束后保留。
	-  The final Linear and Softmax layer: Linear层将解码器的最后输出映射到一个logits向量(一维向量宽度等同于volcabulary, 每个值表示词的清晰那个)，softmax层将分数转换为概率值(均为正，合为1)
	 - 猜想: encoder_forward对应向量合成，residual_forward对应折线箭头的终点处?

##### 附录
[Word Embedding解释](https://zhuanlan.zhihu.com/p/643560252)
[llm.c实现gpt2仓库](https://github.com/karpathy/llm.c/)
[illustrated-gpt2](https://jalammar.github.io/illustrated-gpt2)
[illustrated-gpt2 中文版(1)](https://zhuanlan.zhihu.com/p/79714797)
[illustrated-gpt2 中文版(2)](https://zhuanlan.zhihu.com/p/343925685)