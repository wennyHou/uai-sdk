# Retrain Example
Retain example includes example script that shows how one can adapt a pretrained network for other classification problems and run the retraining task on UAI Platform. The example is based on https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/image\_retraining                                                                                                

## Setup
You should prepare your own image data and pretrained model before running the task. As UAI Train nodes does not provide network access, you should prepare your data locally.

## Intro
The main body of the retrain code is following the https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/image\_retraining/retrain.py

We made appropriate modifications that it can work with UAI Train Platform

## UAI Example 
We do following modifications to the retain.py:

1. Add UAI SDK related arguments: --data\_dir, --output\_dir, --work\_dir, --num\_gpus, these arguments are auto generated by UAI Train Platform, see: https://github.com/ucloud/uai-sdk/blob/master/uaitrain/arch/tensorflow/uflag.py for more details
2. We concate the image\_dir and model\_dir with the UAI data\_dir to make sure it read data from /data/data/image\_dir and /data/data/model\_dir.
3. We concate the output\_graph with UAI output\_dir to make sure it will save the trained graph to /data/output/output\_graph.
4. We concate the summaries\_dir with UAI output\_dir to make sure it will save the trained graph to /data/output/summaries\_dir and can be loaded by tensorboard service provided by UAI Train platform.

Modifications includes:

    Line 1146:
    """UAI SDK use data_dir output_dir and num_gpus to transfer system specific data                                                                             
    """
    parser.add_argument(                                                                                                                                         
        '--data_dir',                                                                                                                                            
        type=str,
        required=True,
        help='UAI SDK related. The directory where the imagenet input data is stored.')                                                                          
    parser.add_argument(                                                                                                                                         
        '--output_dir',                                                                                                                                          
        type=str,
        required=True,
        help='UAI SDK related. The directory where the model will be stored.')                                                                                   
    parser.add_argument(                                                                                                                                         
        '--num_gpus',                                                                                                                                            
        type=int,
        default=1,
        help='UAI SDK related. The number of gpus used.')         

    Line 1348:
    FLAGS.model_dir = os.path.join(FLAGS.data_dir, FLAGS.model_dir)
    FLAGS.image_dir = os.path.join(FLAGS.data_dir, FLAGS.image_dir)
    FLAGS.output_graph = os.path.join(FLAGS.output_dir, FLAGS.output_graph)
    FLAGS.summaries_dir = os.path.join(FLAGS.output_dir, FLAGS.summaries_dir)

## How to run
We assume you fully understand how UAI Train docker image works and has already reads the DOCS here: https://docs.ucloud.cn/ai/uai-train/guide/tensorflow

### Data Prepare
Suppose we are in the path /data/retain, it contains two directory: code(contains the retrain code) and data (contains training data and pretrained model). tf_tool.py is from https://github.com/ucloud/uai-sdk/blob/master/uaitrain_tool/tf/tf_tool.py

    # pwd
    /data/retraim
    # ls
    code/ data/ tf_tool.py

We put retrain codes into code/ and put training data input data/. In this example we use flowers dataset(You should download the flowers data yourself) and the inception-V3 pretrained graph

    # ls data/
    flower_photos/  inception-2015-12-05.tgz
    # ls data/flower_photos
    daisy  dandelion  roses  sunflowers  tulips
    
Now we are ready to go!

### Pack docker image
#### The simplest CMD applied to run is 

    /data/retrain.py --image_dir=flower_photos --model_dir=./ --output_graph=flower_graph.pb --summaries_dir=./ --architecture=inception_v3

#### An pack cmd example is:

    sudo python tf_tool.py pack --public_key=<UCLOUD_PUB_KEY> \ 
    --private_key=<UCLOUD_PRIV_KEY> \
    --code_path=code/ \
    --mainfile_path=retrain.py \
    --uhub_username=<UCLOUD_ACCOUNT> \
    --uhub_password=<UCLOUD_ACCOUNT_PASSWD> \
    --uhub_registry=<UHUB_DOCKER_REGISTRY> \
    --uhub_imagename=retrain-tf \
    --ai_arch_v=tensorflow-1.4.0 \
    --test_data_path=<LOCAL_PATH_TO_IMAGENET_DATA_FOR_TEST> \
    --test_output_path=<LOCAL_PATH_TO_OUTOUT_FOR_TEST> \
    --train_params="--image_dir=flower_photos --model_dir=./ --output_graph=flower_graph.pb --summaries_dir=./ --architecture=inception_v3"
    
Now you can run it locally or run it on UAI Train Platform.

Note: To do the inference, see: https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/#0