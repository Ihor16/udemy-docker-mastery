# M1:
docker top mongo
ps aux

# M2:
docker inspect
docker stats
docker run -it
docker start -ai
docker exec -it
	# To get into alpine #
	use sh instead of bash

ifconfig -a
docker inspect --format "{{ .NetworkSettings.IPAddress }}" 
docker port
docker network ls
docker network inspect <network name>
docker network create
	"Check that containers in 1 network can talk to each other"
	docker exec my_nginx ping new_nginx
	"Just run, test, and close after"
	docker run --rm <image> <command>
	docker run --rm -it <image> bash
docker run -d --network <network-name> --network-alias <alias for a common dns name>

# M3:
docker image history
docker system df
docker <system,image,container> prune (-a)

	Dockerfile basic structure:
	FROM
	RUN
	WORKDIR
	COPY
	EXPOSE
	CMD

# M4:
docker volume ls
docker run ... -v name:<path in container (may be specified in the Dockerfile)>
docker run ... -v $(pwd):<path in container>

# M4:
docker-compose up
docker-compose down
docker-compose down -v
docker-compose ps
docker-compose top

services:
	proxy:
		build: .
		ports:
		- ...
	web:
		image:
		volumes:
		
"Removes local images built in compose"
docker-compose down --rmi local

"Clean install"
    rm -rf /var/lib/apt/lists/*

"To clone only what you need"
    git clone ... --branch <branch-name> --single-branch --depth 1

build: .
	context: .
	dockerfile: <file-name>

# M5:
"Check if swarm is enabled"
docker info -> look for Swarm: <inactive>
docker swarm init
docker service ls
docker service create
docker service ps <service-name>

ssh root$<node-ip>
docker node update --role manager <node-name>
docker swarm join-token <manager or worker>
docker service scale <service-name>=<replica-count>
	"Check memory available"
	free -h
	"Check disk space"
	df -h
	"Get public IP"
	hostname -I
"Network for in-Swarm communication"
docker network create --driver overlay <name>
	"Wait for changes"
	watch !!
	"Services talk to services"
	"External trafic definitely goes through a router, so the port is accessible from all nodes"
	"Make sh file executable":
	#!/bin/bash
	chmod u+x <file name>
docker stack deploy -c <file-name> <stack-name>
docker stack ls
docker stack services <stack-name>
docker stack ps <stack-name>

docker secret create <secret-name> <file>
echo "text" | docker secret create <secret-name> -
<secret-name>:
	external:true
<secret-name>:
	file:./<file>
PASS_FILE: /run/secrets/<secret-name>

# M6:
Productions pattern:

    - docker-compose up -d 
    (with secrets will still work, but insecurely)

    - docker-compose.yml 
    (for local dev, usually specifies image versions)

    - docker-compose.override.yml 
    (for local dev, picked up automatically by docker-compose up)

    - docker-compose.test.yml 
    (for CI testing, have to specify manually in docker-compose -f <base.yml> -f <test.yml> up -d, no persistent volumes)

    - docker-compose.prod.yml (for prod., docker-compose -f <base.yml> -f <prod.yml> config — returns a .yml file you can use in production)
    (can use "... > output.yml" to get a .yml prod. file)

Updates:
    - docker service update --image <image:version> <servicename>
    - docker service update --env-add <VARIABLE> --publish-rm <port to remove>
    - docker service update --publish-rm <port:port> --publis-add <port:port>
    - docker service scale <service-name=#> <service-name=#>
    - OR just edit a stack.yml file 
    - docker service update --force <service-name>

HEALTHCHECK curl http://localhost/ || false

HEALTHCHECK --timeout=2s --interval=30s --retries=3 \
CMD curl http://localhost || exit 1

# M7:
CI Pattern:
- Can set up the registry to rebuild images when code on GitHub changes (need to submit an open-source application),
- and then notify some CI testing tool about new image so that it can run the tests,
- and that CI tool can — if tests pass — create a production .yml file and deploy it to the server,
- and then you'll have the new version of the app running on a cluster

Tags for local registry
	<local-ip>:<port>/<image-name>
## Secure private registry: https://training.play-with-docker.com/linux-registry-part2/
## Auth auth: https://docs.docker.com/registry/deploying/#native-basic-auth
	Learn more: https://training.play-with-docker.com/
	- Private registries work out-of-the-box in Swarm (so no need to set up secure connections because routing mesh will call correct node)
	- Running registry on only 1 node (so when a container crashes, it is recreated on this same node and has access to same data from the volume):
	--constraint "node.hostname==<node-name>"
	--mount type=volume,src=registry_data,dst=/var/list/registry
Example of constraint in a .yml file: https://github.com/dockersamples/example-voting-app/blob/master/docker-stack.yml
See images in private registry: 
    <url>:5000/v2/_catalog

# M8:
## Learn: 
    - Dockerfile logging with stdout and stderr
## Do: 
	- FROM with specific version
	- apt-get with specific versions for critical dependencies
    - Set up ENV variables in images (e.g., memory limits)
        Better use an entrypoint script: they run before container starts
    - Limit work on manager nodes in production because they're busy anyway with Raft
        docker node update --availability drain <node-name>
    - Add labels to nodes
        docker node update --label-add dc=a <node-name>
    - Stop docker
        sudo systemctl stop docker.service

## Practices:
    - Quality of nodes is better than their quantity
        e.g., better to have 5 powerful nodes than 20 slow ones
    - Scaling down is harder
    - Have only odd # of managers

## Softward
    - Jenkins
    - GitHub Actions
    - Image at https://faun.pub/your-team-might-not-need-kubernetes-57240e8d554a
    - Blob storages for persistent volumes
   
# M9:
- Install multiple kubectl versions
    https://faun.pub/using-different-kubectl-versions-with-multiple-kubernetes-clusters-a3ad8707b87b
- Configure minikube user
    https://github.com/kubernetes/minikube/issues/7903
kubectl create deployment <deployment-name> --image=<image-name>
kubectl delete deployment <deployment-name>
kubectl scale deployment <deployment-name> --replicas=<n>
kubectl logs <pod-name> (-f --tail <n>)
kubectl describe pod <pod-name>

kubectl expose deployment <deployment-name> --port <port>
kubectl expose deployment <deployment-name> --port <port> --name <service-name> --type (NodePort)

# M10:
kubectl create deployment <name> --image <image> --dry-run -o yaml

#M11:
kubectl edit deployment <deployment-name>
kubectl describe deployment <deployment-name>

