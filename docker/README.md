# Build Docker Images for Roach Bank Server and Client

You are able to build a Docker image using the SpringBot framework. From the root of the repo you can run the following command.
```
./mvnw spring-boot:build-image
```

This will create two Docker images in the local registry.
```
REPOSITORY                                                TAG                                                                          IMAGE ID       CREATED         SIZE
bank-client                                               latest                                                                       b876ce6b7004   43 years ago    287MB
paketobuildpacks/builder                                  base                                                                         8edd72ccf110   43 years ago    1.22GB
bank-server                                               latest                                                                       b7db35ff1bcf   43 years ago    330MB
```


Then tag the images and push them into an accessible image registry.
```
docker image tag bank-client mikebookhamcap/bank-client:2.0.1
docker image tag bank-server mikebookhamcap/bank-server:2.0.1
```

Then login to Docker Hub and push the images.
```
docker login
docker image push mikebookhamcap/bank-server:2.0.1
docker image push mikebookhamcap/bank-client:2.0.1
```


