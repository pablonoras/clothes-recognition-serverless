## Steps

- Gets the URL in the request
- Loads the image from this URL
- Uses TensorFlow Lite to apply the model to the image and get the predictions
- Responds with the results

## To create this service we need to: 

- Convert the model from Keras to the TensorFlow LITE format
- Preprocess the images (resize them and apply the preprocessing function)
- Package the code in Docker image, and upload it to ECR (the Docker registry from AWS)
- Create and test the lambda function on AWS
- Make the lambda function available to everyone with AWS API Gateeway

## Amazon Lambda

- Main purpose: "run code without thinking about servers"
- Upload some code and the service takes care of running it and scales it up and down according to the load.
- You pay for time when the function is actually used, when nobody uses the model and invokes our service, you don't pay for anything.

## Environment setup for Mac M1 Pro

- Install miniforge with the following commands: 
```
curl -fsSLo Miniforge3.sh "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-$(uname -m).sh"
```
```
bash Miniforge3.sh -b
```

- To activate miniforge env run if using zsh shell: 
```
/Users/your-user-name/miniforge/bin/conda init zsh
```

- Re-open the terminal, and create a new environment with the comand: 
```
conda create -n miniforge-env
```

- Run the following commands to install tensorflow:
```
conda install -c apple tensorflow-deps
pip install tensorflow-macos
pip install tensorflow-metal
pip install --extra-index-url https://google-coral.github.io/py-repo/ tflite_runtime
```

## Environment setup for other

- Using pyenv for the specific python version and pipenv for creating the environment.

```
pyenv install 3.8.10

pyenv global 3.8.10
```

- You go to the root of the proyect and create de pipenv

```
pip intsall pipenv 

pipenv install 

pip install tensorflow
pip install keras_image_helper
pip install --extra-index-url https://google-coral.github.io/py-repo/ tflite_runtime
```

## Preparing

Get the model file:

```
wget https://github.com/alexeygrigorev/mlbookcamp-code/releases/download/chapter7-model/xception_v4_large_08_0.894.h5
```

- Or: 

```
curl -O https://github.com/alexeygrigorev/mlbookcamp-code/releases/download/chapter7-model/xception_v4_large_08_0.894.h5
```


Covert it:

```
python convert.py
```


Build the image:
```
docker build -t tf-lite-lambda .
```

## Running locally

To run locally

```
docker run --rm -p 8080:8080 tf-lite-lambda
```

Test it

```
python test.py
```


## Publishing

Create an ECR repo:

```
aws ecr create-repository --repository-name lambda-images
```

Login to Docker:

```
aws ecr get-login-password --region sa-east-1 | docker login --username AWS --password-stdin 577498455635.dkr.ecr.sa-east-1.amazonaws.com/lambda-images
```


Publish the image:

```
REGION=sa-east-1
ACCOUNT=XXXXXXXXXXXX
REMOTE_NAME=${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/lambda-images:tf-lite-lambda 
docker tag tf-lite-lambda ${REMOTE_NAME}
docker push ${REMOTE_NAME}
```

## Create a lambda function

* Go to Lambda, create a new function, select "container image"
* Put the image we just created there
* Go to basic settings and adjust timeout (30 sec) and memory (1GB)
* Test it with the following payload:

```json
{
    "url": "http://bit.ly/mlbookcamp-pants"
}
```

## Deploying

To deploy it with AWS Lambda and API Gateway, follow this tutorial: https://github.com/alexeygrigorev/aws-lambda-docker/blob/main/guide.md
