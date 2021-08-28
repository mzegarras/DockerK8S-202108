FROM openjdk:11.0.8-slim

EXPOSE 8888

RUN mkdir /app
ADD ./config-server.jar /app/app.jar
ADD ./entrypoint.sh /app/entrypoint.sh
RUN chmod +x /app/entrypoint.sh

ENTRYPOINT ["/app/entrypoint.sh"]
