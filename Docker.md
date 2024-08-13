## Docker

Cleanup images and containers (with volumnes)   
**DANGEROUS!!**
```sh
# To delete all containers including its volumes:
docker rm -vf $(docker ps -aq)
# To delete all the images:
docker rmi -f $(docker images -aq)
```