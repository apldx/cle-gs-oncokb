NAME   := seqfu/myeloseqhd_v2-requests
TAG    := $$(git log -1 --pretty=%h)
IMG    := ${NAME}:${TAG}
LATEST := ${NAME}:latest
HOST   := $$(basename ${NAME})
PWD    := $$(pwd)

pull:
	docker pull registry.gsc.wustl.edu/mgi-cle/myeloseqhd:v2

build:
	docker build -t ${IMG} .
	docker tag ${IMG} ${LATEST}
 
push:
	docker push ${NAME}

interact:
	docker run --rm -v "${PWD}/..:/host" --hostname ${HOST} --detach-keys="ctrl-@" -t -i ${LATEST} /bin/bash
