FROM nginx
LABEL maintainer="ssm.samoylov@gmail.com"
RUN sed -i 's/80/8000/' /etc/nginx/conf.d/default.conf
RUN usermod -u 1001 nginx
RUN mkdir /app
ADD ./app /usr/share/nginx/html/
COPY ./app /app
EXPOSE 8000


