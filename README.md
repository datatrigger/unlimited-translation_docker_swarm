# An unlimited translation app deployed as a set of Docker containers

As a non-German speaker living in Switzerland, I often need to quickly translate large texts, but I get annoyed by character limits on Google Translate or DeepL. Learning German may have been a *way* better call, but instead I decided to deploy a translation application. It's made of 3 containerized microservices:

* A Flask frontend to get inputs and display translations
* A FastAPI API backend to translate English text, using open-source models (SpaCy, Hugging Face)
* A MySQL database to store previous translations

In this repo, we deploy the app on a single node with a Docker Compose file. In [this repo](https://github.com/datatrigger/unlimited-translation_kubernetes), the app is deployed on a Kubernetes cluster. 

## Run the app on a single host

We could deploy the app with Docker Compose but since Docker secrets are implemented for database credentials, we have to use Docker Swarm (it is actually not that clear and things are currently changing, see the [official documentation](https://docs.docker.com/engine/swarm/secrets/), this [issue](https://github.com/moby/moby/issues/40046) and these pull requests: [PR 1](https://github.com/docker/compose/pull/9386), [PR 2](https://github.com/moby/moby/pull/43543)). Using Docker Compose or a single-node swarm actually does not change anything from our perspective: the ```docker-compose.yaml``` file is the same (except for the secrets!).

1) Initiate Docker Swarm on your localhost

```docker swarm init --advertise-addr 127.0.0.1```

2) Store credentials as Docker secrets:

* ```db_root_password```: mandatory to create the MySQL database
* ```db_user```: user for the Flask front-end container
* ```db_password```: password for user ```db_user```

Syntax:

```printf "<secret_value>" | docker secret create <secret_name> -``` (pay attention to the ending hyphen)
e.g.
```printf "Af_ey7hds7a45" | docker secret create db_root_password -```

3) Deploy app

```docker stack deploy --compose-file docker-compose.yaml unlimited-translation```

The app is deployed at http://localhost:5000

4) Bring the app down

```docker stack rm unlimited-translation```  

The data will be persisted in the database even if the stack is down. If you want to start with a fresh database for the next deployment, run:  

```docker volume rm unlimited-translation_unlimited_translation_database_volume```

Finally, deactivate the Swarm mode if needed:

```docker swarm leave --force```
