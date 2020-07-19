FROM ubuntu:latest
ADD fortuneloop.sh /bin/fortuneloop.sh
RUN apt-get update; apt-get -y install fortune; mkdir /var/htdocs; chmod +x /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]