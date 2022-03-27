FROM bitnami/git:latest

RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get install -y less && \
    apt-get install -y vim

COPY bashrc /root/.bashrc
COPY bash_git /root/.bash_git

RUN mkdir ~/projects && \
	ln -s ~/projects projects