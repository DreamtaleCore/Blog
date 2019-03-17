# Multi-task Learning issues

三个小知识

以下内容译自[Deep Multi-Task Learning – 3 Lessons Learned](https://engineering.taboola.com/deep-multi-task-learning-3-lessons-learned/)

## 其一：整合loss函数

MTL模型中的第一个挑战: 如何为multiple tasks定义一个统一的损失函数？
最简单的办法，我们可以整合不同tasks的loss function，然后简单求和。这种方法存在一些不足，比如当模型收敛时，有一些task的表现比较好，而另外一些task的表现却惨不忍睹。其背后的原因是不同的损失函数具有不同的尺度，某些损失函数的尺度较大，从而影响了尺度较小的损失函数发挥作用。这个问题的解决方案是把多任务损失函数“简单求和”替换为“加权求和”。加权可以使得每个损失函数的尺度一致，但也带来了新的问题：加权的超参难以确定。

幸运的是，有一篇论文《Multi-Task Learning Using Uncertainty to Weigh Losses for Scene Geometry and Semantics》通过“不确定性(uncertainty)”来调整损失函数中的加权超参，使得每个任务中的损失函数具有相似的尺度。该算法的keras版本实现，详见[github](https://github.com/yaringal/multi-task-learning-example/blob/master/multi-task-learning-example.ipynb)。

## 其二： 调整 learning rate

在神经网络的参数中，learning rate是一个非常重要的参数。在实践过程中，我们发现某一个learnig rate=0.001能够把任务A学习好，而另外一个learning rate=0.1能够把任务B学好。选择较大的learning rate会导致某个任务上出现dying relu；而较小的learning rate会使得某些任务上模型收敛速度过慢。怎么解决这个问题呢？对于不同的task，我们可以采用不同的learning rate。这听上去很复杂，其实非常简单。通常来说，训练一个神经网络的tensorflow代码如下：

```python
optimizer = tf.train.AdamOptimizer(learning_rate).minimize(loss)
```

其中`AdamOptimizer`定义了梯度下降的方式，`minimize`则计算梯度并最小化损失函数。我们可以通过自定义一个`minimize`函数来对某个任务的变量设置合适的learning rate。

```python
all_variables = shared_vars + a_vars + b_vars
all_gradients = tf.gradients(loss, all_variables)

shared_subnet_gradients = all_gradients[:len(shared_vars)]
a_gradients = all_gradients[len(shared_vars):len(shared_vars + a_vars)]
b_gradients = all_gradients[len(shared_vars + a_vars):]

shared_subnet_optimizer = tf.train.AdamOptimizer(shared_learning_rate)
a_optimizer = tf.train.AdamOptimizer(a_learning_rate)
b_optimizer = tf.train.AdamOptimizer(b_learning_rate)

train_shared_op = shared_subnet_optimizer.apply_gradients(zip(shared_subnet_gradients, shared_vars))
train_a_op = a_optimizer.apply_gradients(zip(a_gradients, a_vars))
train_b_op = b_optimizer.apply_gradients(zip(b_gradients, b_vars))

train_op = tf.group(train_shared_op, train_a_op, train_b_op)

```

## 其三：任务A作为其他任务的特征

当我们构建了一个MTL的神经网络时，该模型对于任务A的估计可以作为任务B的一个特征。在前向传播时，这个过程非常简单，因为模型对于A的估计就是一个tensor，可以简单的将这个tensor作为另一个任务的输入。但是后向传播时，存在着一些不同。因为我们不希望任务B的梯度传给任务A。幸运的是，`Tensorflow`提供了一个API `tf.stop_gradient`。当计算梯度时，可以将某些tensor看成是`constant`常数，而非变量，从而使得其值不受梯度影响。代码如下：

```python
all_gradients = tf.gradients(loss, all_variables, stop_gradients=stop_tensors)
```

再次值得一提的是，这个trick不仅仅可以在MTL的任务中使用，在很多其他任务中也都发挥着作用。比如，当训练一个GAN模型时，我们不需要将梯度后向传播到对抗样本的生成过程中。

值得一提的是，这样的trick在单任务的神经网络上效果也是很好的。