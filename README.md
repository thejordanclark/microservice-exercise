# Microservice Exercise

Lab 6.5

~~~
export LOCALIP=10.0.1.??
docker tag shekhar/stockmanager:1.0 ${LOCALIP}:5000/shekhar/stockmanager:1.0
docker tag shekhar/productcatalogue:1.0 ${LOCALIP}:5000/shekhar/productcatalogue:1.0
docker tag shekhar/shopfront:1.0 ${LOCALIP}:5000/shekhar/shopfront:1.0


docker push ${LOCALIP}:5000/shekhar/stockmanager:1.0
docker push ${LOCALIP}:5000/shekhar/productcatalogue:1.0
docker push ${LOCALIP}:5000/shekhar/shopfront:1.0

cd ../kubernetes
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' productcatalogue-service.yaml
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' shopfront-service.yaml
sed -i'.bak' 's/shekhar/'"${LOCALIP}"':5000\/shekhar/' stockmanager-service.yaml
~~~