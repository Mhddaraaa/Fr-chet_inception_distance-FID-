# **Generative Adversarial Networks(GANs)**
<img align='center' width='800' src="https://cdn-images-1.medium.com/v2/resize:fit:851/0*pPEL7ryJR51VpnDO.jpg">
<br>


## **Fréchet inception distance (FID)**
Fréchet inception distance (FID) is a metric for quantifying the realism and diversity of images generated by generative adversarial networks (GANs). Realistic could mean that generated images of people look like real images of people. Diverse means they are different enough from the original to be interesting and novel.It's a metric that calculates the distance between **feature vectors** calculated for real and generated images Not image itself.

FID was proposed as an improvement over the existing **Inception Score**.

<br>

<img align='right' width='400' src="https://omrit.filtser.com/lib/man-dog2.gif">

The Fréchet distance quantifies the similarity of two curves. First introduced in 1906 by Maurice Fréchet, it quantifies the minimum length of leash required between a dog and walker while each walked a separate curved path of a certain distance.

The **Inception-v3** model used in FID is one in a library of modules introduced by Google as part of its GoogLeNet convolutional neural network in 2014. It was first discussed in a [research paper](https://arxiv.org/pdf/1409.4842.pdf). These various inception models are sometimes used to extract features in computer vision tasks and detect objects. Despite not being the latest model, the Inception-v3 model combined with the Fréchet distance is best suited for analyzing GAN imagery.

FID is measured by computing the differences between the representations of **features**, such as edges and lines, and higher-order phenomena, such as the shapes of eyes or how manu eyes ,nose, ears, legs, ... the images have that are transformed into an intermediate latent space. FID is calculated using the following steps:


1. **Extract feature representations:** Pass the real and generated images through the Inception-v3 model. This transforms the raw pixels into numerical vectors to represent aspects of the images, such as lines, edges and higher-order shapes.
2. **Calculate statistics:** Statistical analysis is performed to determine the mean and covariance matrix of the features in each image.
3. **Compute the Fréchet distance:** Compare the difference between each image's computed mean and covariance matrixes.
4. **Obtain the FID:** Compare the Fréchet distance between the real and generated images. Lower numbers indicated the images are more similar.

- Although keep in mind as always the input images must be Preprocessed to statisfy the Inception-v3 model requirment. This can include resizing to a given dimension size (-, 3, 299, 299), and then normalizing pixel values.


You can have access to orginal paper [here](https://arxiv.org/abs/1706.08500017)
<br>

## **How to Calculate the Frechet Inception Distance:**

First loading a pre-trained Inception v3 model.
The output layer of the model is removed and the output is taken as the activations from the last pooling layer, **Mixed_7c layer**
<img align='right' width='500' src="https://pytorch.org/assets/images/googlenet1.png">
- Mixed layer in inception model typically consist of a set of parallel convolutional branches with different filter sizes and are designed to capture features at multiple scales

This output layer has 2,048 activations, therefore, each image is predicted as 2,048 activation features. This is called the coding vector or feature vector for the image.The result will be two collections of 2,048 feature vectors for real and generated images.

For two **multidimensional Gaussian distributions** ${\displaystyle {\mathcal {N}}(\mu ,\Sigma )} $ and ${\displaystyle {\mathcal {N}}(\mu',\Sigma ')}$, **Fréchet distance** is explicitly solvable as:

<br>

$$
    \large d_F\left({\displaystyle {\mathcal {N}}(\mu ,\Sigma )},{\mathcal {N}}(\mu',\Sigma ') \right)^2 = {\displaystyle \|\mu_1 - \mu_2 \|^2} + Tr \left( {\displaystyle \Sigma + \Sigma^{'} - 2 \sqrt{\Sigma \Sigma^{'}}} \right) + \log{\left(\frac{\text{det}(\Sigma \Sigma^{'})}{\sqrt{\text{det}(\Sigma)\cdot \text{det}(\Sigma^{'})}} \right)}
$$

<br>
<br>

- The score is referred to as $\large d_F\left({\displaystyle {\mathcal {N}}(\mu ,\Sigma )}, {\mathcal {N}}(\mu',\Sigma ') \right)^2$, showing that it is a distance and has squared units.

- The $\large \mu_1 \mu_2 $ refer to the feature-wise **mean** of the real and generated images, e.g. 2,048 element vectors where each element is the mean feature observed across the images.

- The $\large \Sigma, \Sigma^{'}$ are the **covariance matrix** for the real and generated feature vectors, often referred to as sigma.

- The $\large {\displaystyle \|\mu_1 - \mu_2 \|^2}$ ( denotes the Euclidean norm.) refers to the sum squared difference between the two mean vectors. $\large Tr$ refers to the trace linear algebra operation, e.g. the sum of the elements along the main diagonal of the square matrix.

- The $\large \sqrt{\Sigma \Sigma^{'}}$ is the square root of the square matrix, given as the product between the two covariance matrices. not element wise square root. This operation can fail depending on the values in the matrix because the operation is solved using numerical methods. Commonly, some elements in the resulting matrix may be imaginary, which often can be detected and removed.
- $\large det(⋅)$ denotes the determinant of a matrix.

<br>

### **The logarithmic term in the Fréchet Distance formula:**

involving determinants $\large \log{\left(\frac{\text{det}(\Sigma \Sigma^{'})}{\sqrt{\text{det}(\Sigma)\cdot \text{det}(\Sigma^{'})}} \right)}$ is related to the volume of the covariance matrices and plays a crucial role in capturing the differences in the shapes of the distributions.

**In summary**, the logarithmic term helps to quantify how the covariance ellipsoids of the two distributions differ in terms of volume and orientation. Including this term enhances the sensitivity of the Fréchet Distance to variations in the shapes of the distributions, providing a more comprehensive measure of their dissimilarity.Here's a brief explanation of how this term contributes:
1. **Determinants:**
    - $det(Σ)$ and $det(Σ^{'})$ represent the volumes (or "sizes") of the covariance ellipsoids for the two distributions.
    - $det(ΣΣ^{'})$ represents the volume of the product of the covariance ellipsoids.
2. **Logarithmic Term**

    - The logarithmic term essentially compares the ratio of the volume of the joint covariance ellipsoid $(ΣΣ^{'})$to the geometric mean of the individual covariance ellipsoids $\sqrt{det(Σ) \cdot det(Σ^{'}})$.
    - A positive value of this term indicates that the joint covariance ellipsoid has a larger volume than expected based on the individual distributions. A negative value indicates the opposite
3. **Significance:**

    - If the distributions have different shapes or orientations, the logarithmic term helps capture the non-sphericity or anisotropy in the joint distribution.
    - It penalizes cases where the two distributions have very different shapes, even if their means are close.

Below is a code block that calculates the Fréchet Distance between two sets of points using their neural network outputs. The mathematical formula is provided in LaTeX for clarity.

**Note:**The expression ```torch.linalg.solve(diff, sigma1)```.solution involves solving a linear system of equations using the torch.solve function in PyTorch.
<br>

## **Intermediate Activations — the forward hook in Pytorch**
In PyTorch, a forward hook is a function that is registered to be called when the forward pass of a neural network module is executed. It allows you to intercept and perform additional operations during the forward pass without modifying the original forward method of the module.

extracting activations during the forward pass of the model is by attaching a “forward_hook”.  In PyTorch the method ```register_forward_hook``` is under the ```nn.Module``` class definition. Here is the [documnetation](https://pytorch.org/docs/stable/generated/torch.nn.modules.module.register_module_forward_hook.html).

**Hooks** are callable objects that can be registered to any ```nn.Module``` object. When the ```forward()``` method is triggered in a model forward pass,
- the **module** itself, along with its **inputs** and **outputs** are passed to the forward_hook before proceeding to the next module.

Since intermediate layers of a model are of the type nn.module, we can use these forward hooks on them to serve as a lens to view their activations.

```
hook(module, input, output) -> None or modified output
```

<br>

```
model = inception_v3(pretrained=True)
model = model.to(device)

def layer_output(name):
    """
    extract feature from certain layer of architecture
    name --> data will be save in dictionary with key of 'name'
    """
    def hook(model, input, output):
        activation = output.detach()
    return {name: activation}

# Register forward hooks on the layers of choice which is Mixed_7c layer here.
hook = model.Mixed_7c.register_forward_hook(layer_output('Mixed_7c'))

# forward pass -- getting the outputs
out = model(X)

print(activation)

# detach the hooks
hook.remove()
```

<br>

we can do the same in class, so we have to define new method same as above function:
```
import torch.nn.functional as F
class inception_fearture_etraction(nn.Module):

    def __init__(self, transform_input=True):
        super().__init__()
        self.net = inception_v3(pretrained=True)
        self.net.Mixed_7c.register_forward_hook(self._hook)
        self.transform_input = transform_input

    def _hook(self, module, input, output):
        # N x 2048 x 8 x 8
        self.output = output

    def forward(self, x): # Remember to normalize to [-1, 1]
        # Trigger output hook
        self.net(x)

        activations = self.output
        activations = F.adaptive_avg_pool2d(activations, (1,1)) # Output: N x 2048 x 1 x 1
        return activations.view(x.shape[0], 2048)
```

<br>

Forward hooks are often used for various purposes, such as feature extraction, monitoring activations, or implementing custom operations during the forward pass without modifying the model's architecture.
<br>

## **Create Classifier and train it**

Since the inception-v3 train on ImageNET datasets while we want to generate digits. So we need to create our classifier and train it. Here ia our model:
```
----------------------------------------------------------------
        Layer (type)               Output Shape         Param #
================================================================
            Conv2d-1           [-1, 32, 30, 30]             320
       BatchNorm2d-2           [-1, 32, 30, 30]              64
            Conv2d-3           [-1, 64, 28, 28]          18,496
       BatchNorm2d-4           [-1, 64, 28, 28]             128
         MaxPool2d-5           [-1, 64, 14, 14]               0
            Conv2d-6          [-1, 128, 12, 12]          73,856
       BatchNorm2d-7          [-1, 128, 12, 12]             256
         MaxPool2d-8            [-1, 128, 6, 6]               0
            Conv2d-9            [-1, 256, 4, 4]         295,168
      BatchNorm2d-10            [-1, 256, 4, 4]             512
          Flatten-11                 [-1, 4096]               0
           Linear-12                  [-1, 128]         524,416
           Linear-13                   [-1, 10]           1,290
================================================================
Total params: 914,506
Trainable params: 914,506
Non-trainable params: 0
----------------------------------------------------------------
Input size (MB): 0.00
Forward/backward pass size (MB): 1.71
Params size (MB): 3.49
Estimated Total Size (MB): 5.20
----------------------------------------------------------------
```
## **Time to check the output**

<img align='center' width='1000' src="https://github.com/Mhddaraaa/Fr-chet_inception_distance-FID-/blob/main/FD2.png">
