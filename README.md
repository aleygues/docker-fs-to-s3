# fsdump-to-s3

This repo (and its built Docker image) should be used to generate ZIP folder dumps (using `zip`) automatically and upload them to a remote location (using S3 API). 

The image has been thought to be used on a Docker Swarm infrastructure (but should be easily adapted to a Docker Compose environment). 

All stacks deployed **on the same node** presenting **labels** described bellow will be handled. Dump process runs every 15 minutes (cannot be customized yet). 

## Deploy the stack

```yml
version: "3"

services:
    pgdumptos3:
        image: aleygues/fs-to-s3
        restart: always
        environment:
            - S3_API_KEY=
            - S3_API_SECRET=
            - S3_REGION=
            - S3_API_ENDPOINT=
            - S3_BUCKET_NAME=
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
        deploy:
            mode: global
            placement:
                constraints: [node.labels.pgdump == true] # you may change this depending on your needs
```

## Update app services

```yml
version: '3'

services:
  payload:
    image: # some image
    volumes:
      - media:/app/media  # for instance
    # -- here starts example labels
    labels:
      - fr.aleygues.fsdump
      - fr.aleygues.fsdump.path=/app/media # for instance
      - fr.aleygues.fsdump.prefix= # prefix used to generate dumps
      - fr.aleygues.fsdump.frequency=daily
      - fr.aleygues.fsdump.daysRetention=7
    # -- here it ends
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.labels.nodeId == 2]
volumes:
  media:
    driver: local
```

## Check if it's working

You should wait for the next 15 minutes tick (HH:00, HH:15, ...), and the logs of the dump service, then check the generated dump file

## Good practices 

Here are some good practices:
- check the dump integrity (does it contains all the files you need to restart your app?)
- do this verification regularly (every month for instance)
- train yourself to restart an app from its generated backup
- document everything on a document (and this should be the main part of your DRP)

Enjoy!
