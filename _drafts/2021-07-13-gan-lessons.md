---
layout: post
title:  "GAN Lessons, Tips, and Tricks"
thumbnail: "/images/2020/motor_thumb.jpg"
tags: article
---

This is GAN lessons.

## Table of Contents

1. [Pix2Pix](#pix2px)


### Kernel Sizes

Pix2Pix: 4

BigGAN: We tried using filter sizes of 5 or 7 instead of 3 in either G or D or both. We found that having a filter size of 5 in G only provided a small improvement over the baseline but came at an unjustifiable compute cost. All other settings degraded performance.

### Normalization

* BatchNorm: Pix2Pix

    BigGAN: We tried adding BatchNorm to D (both class-conditional and unconditional) in addition to Spectral Normalization, but this crippled training.
* Conditional Batch Norm: SPADE
* Spectral: BigGAN

* U-GAT-IT: Instance norm in downsampling encoder. Layer instance normalization on the way up. Adadaptive instance layer norm in the bottleneck.

### Activations

* Pix2Pix Encoder/Discriminator: All ReLUs are leaky, with slope 0.2. Decoder: Not Leaky.
* SPADE Encoder/Discriminator: LReLU. Generator: ReLU.
* BigGAN: ReLU only. No leaky.
* U-GAT-IT: For the activation function, we use ReLU in the generator and leaky-ReLU with a slope of 0.2 in the discriminator.

### Upsampling

Pix2Pix: ConvT

BigGAN: We tried bilinear upsampling in G in place of nearest-neighbors upsampling, but this de- graded performance.

### Dropout

* Pix2Pix: Decoder only. First 3 layers. 50%. Between Norm and ReLU.
* SPADE: None

### Adding Noise

## Architectures

### UNET

### Residual

## Discriminator

### PatchGAN

> It is well known that the L2 loss – and L1, see Figure 4 – produces blurry results on image generation problems [34]. Although these losses fail to encourage high-frequency crispness, in many cases they nonetheless accurately capture the low frequencies. For problems where this is the case, we do not need an entirely new framework to enforce correctness at the low frequencies. L1 will already do.
>
> This motivates restricting the GAN discriminator to only model high-frequency structure, relying on an L1 term to force low-frequency correctness (Eqn. 4). In order to model high-frequencies, it is sufficient to restrict our attention to the structure in local image patches. Therefore, we design a discriminator architecture – which we term a PatchGAN – that only penalizes structure at the scale of patches. This discriminator tries to classify if each N × N patch in an image is real or fake. We run this discriminator convolution- ally across the image, averaging all responses to provide the ultimate output of D.

Pix2Pix: PatchGAN size is 70.

U-GAT-IT: We employ two different scales of PatchGAN (Isola et al. (2017)) for the discriminator network, which classifies whether local (70 x 70) and global (286 x 286) image patches are real or fake.

### Multi-resolution Discriminator

Pix2PixHD:

> To differentiate high-resolution real and synthesized images, the discriminator needs to have a large receptive field. This would require either a deeper network or larger convolutional kernels, both of which would in- crease the network capacity and potentially cause overfit- ting. Also, both choices demand a larger memory footprint for training, which is already a scarce resource for high- resolution image generation.
>
> To address the issue, we propose using multi-scale dis- criminators. We use 3 discriminators that have an identical network structure but operate at different image scales. We will refer to the discriminators as D1 , D2 and D3 . Specifically, we downsample the real and synthesized high- resolution images by a factor of 2 and 4 to create an image pyramid of 3 scales. The discriminators D1, D2 and D3 are then trained to differentiate real and synthesized images at the 3 different scales, respectively. Although the discriminators have an identical architecture, the one that operates at the coarsest scale has the largest receptive field. It has a more global view of the image and can guide the generator to generate globally consistent images. On the other hand, the discriminator at the finest scale encourages the generator to produce finer details. This also makes training the coarse-to-fine generator easier, since extending a low- resolution model to a higher resolution only requires adding a discriminator at the finest level, rather than retraining from scratch. Without the multi-scale discriminators, we observe that many repeated patterns often appear in the generated images.

### Generator Attention Map

From SPA-GAN:

> Formally, given an input image x, the spatial attention map ADX (x), whose size is the same as the input image x, is obtained by feeding x to the discriminator. Following [9], we define ADX (x) as the sum of the absolute values of activation maps in each spatial location in a layer across the channel dimension:
>
> ```
>      C
> AD = ∑|Fi| (1)
>      i=1
> ```
>
> where Fi is i-th feature plane of a discriminator layer for the specific input and C is the number of channels. AD directly indicates the importance of the hidden units at each spatial location in classifying the input image as a fake or real.
>
> The attention maps of different layers in a classifier network focus on different features. For instance, when classifying apples or faces, the middle layer attention maps have higher activations on regions such as the top of an apple or eyes and lips of the face, while the attention maps of the later layers typically focus on full objects. Thus, in SPA-GAN we select the mid-level attention maps from the second to last layer in DX , usually correlated to discriminative object parts [9], and feed them back to the generator.
>
> The detailed architecture of SPA-GAN is shown in panel (b) of Fig. 1. First, an input image x is fed to the discriminator D, to get the spatial attention map ADX (x), the most discriminative
> regions in x. Then, the spatial attention map is normalized by dividing each value by the maximum value observed in the map and upsampled to match the input image size. Next, we apply the spatial attention map to the input image x using an element-wise product and feed it to the generator G to help it focus on the most discriminative parts when generating x′:
>
> ```
> x′ = G(xa) = G(ADX (x)⊙x) (2) 
> ```
> where xa is the attended input sample.

### Discriminator Feature Matching Loss

Pix2PixHD:

> We improve the GAN loss in
> Eq. (2) by incorporating a feature matching loss based on
> the discriminator. This loss stabilizes the training as the
> generator has to produce natural statistics at multiple scales.
> Specifically, we extract features from multiple layers of the
> discriminator and learn to match these intermediate representations from the real and the synthesized image. For ease
> of presentation, we denote the ith-layer feature extractor of
> discriminator Dk as D(i) (from input to the ith layer of Dk). k
> The feature matching loss LFM(G,Dk) is then calculated as:
>
>
>
> where T is the total number of layers and Ni denotes the number of elements in each layer. Our GAN discriminator feature matching loss is related to the perceptual loss [11, 13,22], which has been shown to be useful for image super- resolution [32] and style transfer [22]. In our experiments, we discuss how the discriminator feature matching loss and the perceptual loss can be jointly used for further improving the performance. We note that a similar loss is used in VAE-GANs [30].

## Training

### Learning Rate

* Pix2Pix: Adam solver with a learning rate of 0.0002, and momentum parameters β1 = 0.5, β2 = 0.999.

* SPADE: Learning rates for the generator and discriminator are 0.0001 and 0.0004, respectively [17]. We use the ADAM solver [27] with β1 = 0 and β2 = 0.999.

* BigGAN: We use Adam optimizer with β1 = 0 and β2 = 0.999 and a constant learning rate. For BigGAN models at all resolutions, we use 2·10−4 in D and 5·10−5 in G. For BigGAN-deep, we use the learning rate of 2·10−4 in D and 5·10−5 in G for 128×128 models, and 2.5·10−5 in both D and G for 256 × 256 and 512 × 512 models. 

    We found that the SA-GAN settings (G’s learning rate 10−4, D’s learning rate 4·10−4) were optimal at lower batch sizes.

    We swept D’s Adam β1 parameter through [0.1, 0.2, 0.3, 0.4, 0.5] and found it to have a light regularization effect similar to DropOut, but not to significantly improve results. Higher β1 terms in either network crippled training.

    We found that increasing the learning rates (relative to their initial values) in either G or D, or both G and D, led to immediate collapse. This occurred even when doubling the learning rates from 2 · 10−4 in D and 5·10−5 in G, to 4·10−4 in D and 1·10−4 in G, a setting which is not normally unstable when used as the initial learning rates.

* U-GAT-IT: All models are trained using Adam (Kingma & Ba (2015)) with β1=0.5 and β2=0.999. The batch size is set to one for all experiments. We train all models with a fixed learning rate of 0.0001 until 500,000 iterations and linearly decayed up to 1,000,000 iterations. We also use a weight decay at rate of 0.0001. 

### Weights Initialization

* U-GAT-IT: The weights are initialized from a zero-centered normal distribution with a standard deviation of 0.02.

### Loss

* Pix2Pix: L1 + cGAN.

* U-GAT-IT: D is x1 LSGAN (MSE). G is x10 MAE. Uses MSE for Discriminator and MAE for Identity. Loss scales are λD= 1, λI = 10.



### Procedure

From Pix2Pix: To optimize our networks, we follow the standard approach from [24]: we alternate between one gradient descent step on D, then one step on G. As suggested in the original GAN paper, rather than training G to minimize log(1 − D(x, G(x, z)), we instead train to maximize log D(x, G(x, z)) [24]. In addition, we divide the objective by 2 while optimizing D, which slows down the rate at which D learns relative to G.

BigGAN: We experimented with the number of D steps per G step (varying it from 1 to 6) and found that two D steps per G step gave the best results.

## Networks

### BigGAN

### Pix2Pix

[Website](https://phillipi.github.io/pix2pix/) [Image-to-Image Translation with Conditional Adversarial Networks](https://arxiv.org/abs/1611.07004)

Let Ck denote a Convolution-BatchNorm-ReLU layer with k filters. CDk denotes a Convolution-BatchNorm- Dropout-ReLU layer with a dropout rate of 50%. All convolutions are 4 × 4 spatial filters applied with stride 2.

encoder:
C64-C128-C256-C512-C512-C512-C512-C512

U-Net decoder:
CD512-CD1024-CD1024-C1024-C1024-C512
-C256-C128

The 70 × 70 discriminator architecture is: C64-C128-C256-C512

* [UNET](#unet)
* [PatchGAN](#patchgan)

### Pix2PixHD

* [Multi-resolution Discriminator](#multi-resolution-discriminator)

### SPA-GAN: Spatial Attention GAN for Image-to-Image Translation

By Hajar Emami, Majid Moradi Aliabadi, Ming Dong, and Ratna Babu Chinnam

* [Generator Attention Map](#generator-attention-map)

### SPADE

### U-GAT-IT

unsupervised image-to-image translation

U-GAT-IT: UNSUPERVISED GENERATIVE ATTEN- TIONAL NETWORKS WITH ADAPTIVE LAYER- INSTANCE NORMALIZATION FOR IMAGE-TO-IMAGE TRANSLATION

https://github.com/taki0112/UGATIT/blob/master/UGATIT.py

