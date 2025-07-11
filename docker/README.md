# Build Docker Image for Ledger

You are able to build a Docker image using the SpringBot framework. From the root of the repo you can run the following command.
```
./mvnw spring-boot:build-image
```

This will create two Docker images in the local registry.
```
REPOSITORY                                                TAG                                                                          IMAGE ID       CREATED         SIZE
ledger                                               latest                                                                       b876ce6b7004   43 years ago    287MB
paketobuildpacks/builder                                  base                                                                         8edd72ccf110   43 years ago    1.22GB
```


Then tag the images and push them into an accessible image registry.
```
docker image tag ledger mikebookhamcap/ledger:1.0.1
```

Then login to Docker Hub and push the images.
```
docker login
docker image push mikebookhamcap/ledger:1.0.1
```


