# AlexNet

📜 [ImageNet Classification with Deep Convolutional Neural Networks](https://proceedings.neurips.cc/paper_files/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf)

> Despite the attractive qualities of CNNs, and despite the relative efficiency of their local architecture, they have still been prohibitively expensive to apply in large scale to high-resolution images.

> Luckily, current GPUs, paired with a highly-optimized implementation of 2D convolution, are powerful enough to facilitate the training of interestingly-large CNNs, and recent datasets such as ImageNet contain enough labeled examples to train such models without severe overfitting.

Efficient use of compute has always been critical to practically pushing the boundaries on deep learning effectively.

> In the end, the network’s size is limited mainly by the amount of memory available on current GPUs and by the amount of training time that we are willing to tolerate.

> All of our experiments suggest that our results can be improved simply by waiting for faster GPUs and bigger datasets to become available.

> ImageNet is a dataset of over 15 million labeled high-resolution images belonging to roughly 22,000 categories. The images were collected from the web and labeled by human labelers using Amazon’s Mechanical Turk crowd-sourcing tool.

### **Architecture**

**1. ReLU**

8 layers total - 5 convolutional layers, 3 fully-connected feed-forward layers

> Deep convolutional neural networks with ReLUs train several times faster than their equivalents with tanh units.

ReLU trains way faster than tanh because of the simplicity of differentiating it, which actually meaningfully contributes to the constraints.

Also importantly, it is a non-saturating nonlinearity - functions like sigmoid and tanh (saturating) compress inputs into [0,1], which means at large magnitudes, their gradient approaches 0. This causes the _vanishing gradient problem_. Meanwhile, ReLU never runs into this problem.

**2. Multiple GPUs**

> A single GTX 580 GPU has only 3GB of memory, which limits the maximum size of the network that can be trained on it. It turns out that 1.2 million training examples are enough to train networks which are too big to fit on one GPU. Therefore we spread the net across two GPUs.

> Current GPUs are particularly well-suited to cross-GPU parallelization, as they are able to read from and write to one another’s memory directly, without going through host machine memory.

> The parallelization scheme that we employ essentially puts half of the kernels (or neurons) on each GPU, with one additional trick: the GPUs communicate only in certain layers.

Communication between GPUs is tuned to be sensible for the amount of available compute (they were only using 2 GPUs). This was one of the first times someone really used multiple GPUs for training.

**3. Local Response Normalization**

> ReLUs have the desirable property that they do not require input normalization to prevent them from saturating.

Normalization is not a necessity to prevent saturation (whereas it is necessary with tanh and sigmoid because of their vanishing gradients at large values).

However, normalization still provides advantages.

> This sort of response normalization implements a form of lateral inhibition
> inspired by the type found in real neurons, creating competition for big activities amongst neuron outputs computed using different kernels.

They implement **local response normalization**. This prevents many kernels from firing on the same pixel, forcing more prominent features to stand out, and weaker features to be de-emphasized.

**4. Overlapping Pooling**

> Pooling layers in CNNs summarize the outputs of neighboring groups of neurons in the same kernel map. Traditionally, the neighborhoods summarized by adjacent pooling units do not overlap.

Instead, AlexNet uses overlapping pooling layers which improves regularization (less overfitting) and decreases loss.

**5. Overall Architecture**

> The first convolutional layer filters the 224×224×3 input image with 96 kernels of size 11×11×3 with a stride of 4 pixels.

> The second convolutional layer takes as input the (response-normalized and pooled) output of the first convolutional layer and filters it with 256 kernels of size 5 × 5 × 48.

> The third, fourth, and fifth convolutional layers are connected to one another without any intervening pooling or normalization layers.

> The third convolutional layer has 384 kernels of size 3 × 3 × 256 connected to the (normalized, pooled) outputs of the second convolutional layer. The fourth convolutional layer has 384 kernels of size 3 × 3 × 192 , and the fifth convolutional layer has 256 kernels of size 3 × 3 × 192.

### Regularization

**1. Dataset Augmentation**

> The easiest and most common method to reduce overfitting on image data is to artificially enlarge the dataset using label-preserving transformations

They train the model on 224 x 224 chunks of the 256 x 256 pixel images, and augment the intensities of the RGB channels to regularize.

**2. Dropout**

> Combining the predictions of many different models is a very successful way to reduce test errors, but it appears to be too expensive for big neural networks that already take several days to train.

We already know here the intuition behind multi-headed attention! Having multiple separate models run and develop their own intuitions, then combining their outputs is great where you can afford to do it.

> There is, however, a very efficient version of model combination that only costs about a factor of two during training. The recently-introduced technique, called “dropout”, consists of setting to zero the output of each hidden neuron with probability 0.5.

> This technique reduces complex co-adaptations of neurons, since a neuron cannot rely on the presence of particular other neurons.

Neurons can’t co-adapt with each other by becoming highly interdependent on other neurons to process (higher dimensional) features.

> It is, therefore, forced to learn more robust features that are useful in conjunction with many different random subsets of the other neurons.

Instead, dropout compels individual neurons to learn more broadly useful features that can be useful with any group of neurons.

> At test time, we use all the neurons but multiply their outputs by 0.5, which is a reasonable approximation to taking the geometric mean of the predictive distributions produced by the exponentially-many dropout networks.

Since the network was trained with 50% of the neurons active at one time, we expect to have 2x the total contributions at test time since all neurons are active, so we scale the outputs by 0.5.

> Without dropout, our network exhibits substantial overfitting. Dropout roughly doubles the number of iterations required to converge.

### Results & Discussion

> Our network achieves top-1 and top-5 test set error rates of 37.5% and 17.0%. The best performance achieved during the ILSVRC2010 competition was 47.1% and 28.2%.

Their model beats state of the art by far.

> The kernels on GPU 1 are largely color-agnostic, while the kernels
> on on GPU 2 are largely color-specific.

Each GPU (part of the network on each GPU, which you actually need to think about) tends to specialize to different types of kernels as they are isolated.

![Screenshot 2024-05-09 at 10.21.53 AM.png](../../images/Screenshot_2024-05-09_at_10.21.53_AM.png)

5 images in the training set with the 6 images closest to them based on the euclidian distance of the activations in the last layer (showing that later layers have combined data to form similarity calls between images).

> Our results show that a large, deep convolutional neural network is capable of achieving record-breaking results on a highly challenging dataset using purely supervised learning.

> It is notable that our network’s performance degrades if a single convolutional layer is removed. For example, removing any of the middle layers results in a loss of about 2% for the top-1 performance of the network. So the depth really is important for achieving our results.

Depth is critical for the network to work effectively.