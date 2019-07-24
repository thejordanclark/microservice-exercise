# Microservice Exercise

## Setup for Microservices Lab
#### Step 6.5

First set a local variable for the local IP.  You can find your local IP with:

~~~
ip addr show
~~~

Store the IP as a local variable on your shell.  Don't forget to replay the set the IP to your IP that you found with the command above.

~~~
export LOCALIP=10.0.1.??
~~~

Now we can tag each local image to include the local registry.  This is required to push an image to a registry.

~~~
docker tag shekhar/stockmanager:1.0 ${LOCALIP}:5000/shekhar/stockmanager:1.0
docker tag shekhar/productcatalogue:1.0 ${LOCALIP}:5000/shekhar/productcatalogue:1.0
docker tag shekhar/shopfront:1.0 ${LOCALIP}:5000/shekhar/shopfront:1.0
~~~

Now let us actually push the newly tagged images to the local registery:

~~~
docker push ${LOCALIP}:5000/shekhar/stockmanager:1.0
docker push ${LOCALIP}:5000/shekhar/productcatalogue:1.0
docker push ${LOCALIP}:5000/shekhar/shopfront:1.0
~~~

Finally we need to modify the YAML files to use the images from the registry instead of local images:

~~~
cd ../kubernetes
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' productcatalogue-service.yaml
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' shopfront-service.yaml
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' stockmanager-service.yaml
~~~

You can now continue on to "Step 7"