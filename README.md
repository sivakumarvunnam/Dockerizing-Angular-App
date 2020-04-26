This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.3.9.

## Project Setup
Install the Angular CLI globally::

```
vagrant@sivakumarvunnam:~/Dockerizing-Angular-App$ npm install -g @angular/cli@7.3.9

```
Generate a new app:

```
vagrant@sivakumarvunnam:~/Dockerizing-Angular-App$ ng new Dockerizing-Angular-App
vagrant@sivakumarvunnam:~/Dockerizing-Angular-App$ cd Dockerizing-Angular-App

```

## Production
Let’s create a Dockerfile for use in production called Dockerfile:
```
# base image
FROM node:12.2.0 as build
MAINTAINER sivakumarvunnam1@gmail.com
# install chrome for protractor tests
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
RUN sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
RUN apt-get update && apt-get install -yq google-chrome-stable

# set working directory
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install and cache app dependencies
COPY package.json /app/package.json
RUN npm install
RUN npm install -g @angular/cli@7.3.9

# add app
COPY . /app

# run tests
RUN ng test --watch=false
RUN ng e2e --port 4202

# generate build
RUN ng build --output-path=dist

############
### prod ###
############

# base image
FROM nginx:1.16.0-alpine

# copy artifact build from the 'build environment'
COPY --from=build /app/dist /usr/share/nginx/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]

```
Create the following folder along with a nginx.conf file:

```
└── nginx
    └── nginx.conf
```
nginx.conf:

```
server {

  listen 80;

  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page   500 502 503 504  /50x.html;

  location = /50x.html {
    root   /usr/share/nginx/html;
  }

}

```
