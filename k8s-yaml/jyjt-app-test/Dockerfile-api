FROM java:8
MAINTAINER i@sjhz.cf
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
WORKDIR /
ADD juneyaoYun-api-0.0.1-SNAPSHOT.jar juneyaoYun-api-0.0.1-SNAPSHOT.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "juneyaoYun-api-0.0.1-SNAPSHOT.jar"]