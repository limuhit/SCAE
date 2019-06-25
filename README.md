This is the codes for the paper "Deep Context-Aware Entropy Model for Image Compression" available at https://arxiv.org/abs/1906.10057.

The caffe contains the source codes for Context-Based Convolutional Networks. 

In training, we adopt mask convolution. The masks are controlled by three params, i.e., contrain, group_in, group_out. constrain is used to controll the mask for the input layer (constrain: 5) and hidden layer (constrain: 6); group_in is used to define the number of feature blocks in the input of the layer; group_out is used to define the number of feature blocks in the output of the layer. Please make sure the param "num_output" corresponds to the "group_out". num_output = number_of_channels_in_the_input_of_the_network * group_out.

The following is an example of the mask convolution used in training. Supposing the input of the network ("data") is a 3d map with 8 channels. The first mask conv layer "conv1" takes the network input "data" as input and adopt the "contrain: 5" to select the mask for input layer as shown in Eqn. (10). And the second conv layer "conv2" takes the 3 feature blocks generated by "conv1" as input and adopt "constrain: 6" to select mask for hidden layer as shown in Eqn. (10).

layer {
  name: "conv1"
  type: "Convolution"
  bottom: "data"
  top: "conv1"
  convolution_param {
    num_output: 24
    pad: 2
    kernel_size: 5
    stride: 1
    constrain: 5
    group_in: 1
    group_out: 3
  }
}

layer {
  name: "conv2"
  type: "Convolution"
  bottom: "conv1"
  top: "conv2"
  convolution_param {
    num_output: 24
    pad: 2
    kernel_size: 5
    stride: 1
    constrain: 6
    group_in: 3
    group_out: 3
  }
}

In testing, we rewrite the the convolution operation for sub-block by sub-block parallel decoding. The application can be find at "decoder_conv3_layer.cpp" and "decoder_conv3_layer.cu".  "Decoder_conv3_layer.cpp" depends on the MKL libary for parallel computation which can be complied successfully under windows. For linux user, you may delete the application in "decoder_conv3_layer.cpp" and leave the functions empty to remove the MKL library. The following is an template for it. 

layer {
  name: "conv1"
  type: "DecoderConv3"
  bottom: "data"
  top: "conv1"
  convolution_param {
    num_output: 24
    pad: 2
    kernel_size: 5
    stride: 1
    constrain: 5
    group_in: 1
    group_out: 3
  }
}

In the folder of "cmp", we provide the scripts to train and test the model for lossy image compression. And the pretrained models are available for testing. For training the model, you should train a base model with "train_base.py" and a corresponding GMM entropy model for the base model to make sure the GMM entopy model fulfil with the base model. Then, initialize the whole model with the base model and GMM entropy model, end-to-end train the model with teh "train_gmm.py".  To generate the bit stream, we adopt a post process step. Generating discrete probability for entropy coding from a GMM distribution is complex and the head and tail effect should be considered in application. To make it easy, we retrain a model similar as shown Fig. 5 in the paper and combine it with the arithmetic coding  to generate the bit stream. 

In the folder of "binary", the codes can be directly runed on windows is given. Make sure the windows has a Nvidia graphic card and the CUDA installed. We provide four models with two optimized with MSE and two optimized with MS-SSIM. Run python scae.py -h for moe instruction to encode the image to bit stream and decode the image from the bit stream. 

If you want to use the codes in your further work, please cite our paper as follows:

