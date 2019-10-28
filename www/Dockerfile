#docker build -t infracloud/app-mono-www .
FROM drydock/u14nod:prod

# this folder must be created in the base images
ADD ./_global /root/_global/
ADD . /root/www/
RUN rm -rf /root/www/_global

#now run set up
RUN /bin/bash /root/www/setup.sh

ENTRYPOINT ["/root/www/boot.sh"]
