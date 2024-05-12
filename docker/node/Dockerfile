FROM node:20-alpine3.16
WORKDIR /var/www/application

RUN apk update && \
	apk add git

USER node
RUN mkdir ~/.npm-global
RUN npm config set prefix '~/.npm-global'
RUN echo ' export PATH=~/.npm-global/bin:$PATH' >> ~/.bash_profile
RUN source ~/.bash_profile
EXPOSE 5173