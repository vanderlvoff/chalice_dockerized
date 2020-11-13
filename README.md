# How to use deploy with docker-compose for development purposes 

## 1. Install chalice framework locally: [learn more from chalice quick start](https://aws.github.io/chalice/quickstart.html)

```
# check your python version
python3 --version
# if you are something like Python 3.7.3 and above, you are good to go!

# Next we’ll install Chalice using pip:
python3 -m pip install chalice

# You can verify you have chalice installed by running:
chalice --help
```

## 2. Create brand new project.

```
chalice new-project your-project-name
```

## 3. Check AWS settings:

```
# create this folder if you don't have it
mkdir ~/.aws
cat >> ~/.aws/config
[default]
aws_access_key_id=YOUR_ACCESS_KEY_HERE
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
region=YOUR_REGION (such as us-west-2, us-west-1, etc)
```

## 4. Copy your requirements file for docker.

```
cp requirements.txt requirements-docker.txt

# Add this one line to requirements-docker.txt
# you need it to make sure that you have chalice in your Docker container.

chalice
```

## 5. Let's start dockerizing!
  * Create custom Dockerfile

```
FROM alpine:3.7

WORKDIR /app
COPY requirements-docker.txt ./
RUN \
apk add --no-cache python3 postgresql-libs && \
apk add --no-cache --virtual .build-deps gcc python3-dev musl-dev && \
apk add --no-cache postgresql-dev && \
python3 -m pip install -r requirements-docker.txt --no-cache-dir && \
python3 -m pip install --upgrade pip && \
apk --purge del .build-deps

COPY . .
CMD [ "/bin/sh"]
```

  * Create environment file. It will be used in your docker-compose file later.

```
touch .env
vi .env

######
# Add put there the following content:
#
# YOUR_AWS_ACCESS_KEY_ID
# YOUR_AWS_SECRET_ACCESS_KEY
# YOUR_REGION
# fill with actual aws credentials, taken from
# cat ~/.aws/config
######

APP_NAME=
APP_PATH=/app/${APP_NAME}
AWS_ACCESS_KEY_ID={YOUR_AWS_ACCESS_KEY_ID}
AWS_SECRET_ACCESS_KEY={YOUR_AWS_SECRET_ACCESS_KEY}
AWS_DEFAULT_REGION={YOUR_REGION}
```

  * Create docker-compose.yaml file with following contents
```
version: "3.8"

services:
  app:
    build: .
    env_file:
      - .env
    ports:
      - "80:8000"
    volumes:
      - .:/app
    tty: true
    stdin_open: true
    working_dir: "${APP_PATH}"
    environment:
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
```

  * Build and run containers

```
docker-compose up --build -d
```

## 6. Code your simple lambda function

```
# just copy/paste it to your app.py
 
@app.lambda_function()
def mylambda(event, context):
    return {'response': 'great'}
```
 
## 7. Now you are ready to deploy your first lambda

```
docker-compose run app chalice deploy
```

## 8. List your functions

Here you might want to install AWS CLI first. PLease learn how to to it from here:

[AWS CLI installation](https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html)

```
aws lambda list-functions
```

You'll have listing of lambda files in json format.
Look for something like `"FunctionName": "your-lambda-function-name"` 

## 9. Invoke your function

In my case lambda function name is `dockerized-dev`, so, the command will look like this:

```
aws lambda invoke --function-name dockerized-dev result.txt
```

Where result.txt will keep the response from your lambda.


```
cat result.txt
````

And you'll get `{'response': 'great'}` in it. If not....

Check this repo:
or drop me a line:
