# docker build -t infracloud/app-mono-api .
FROM drydock/u14nod:prod

# this folder must be created in the base images
ADD ./_global /root/_global/
ADD . /root/api/
RUN rm -rf /root/api/_global

#now run set up
RUN /bin/bash /root/api/setup.sh

ENTRYPOINT ["/root/api/boot.sh"]
