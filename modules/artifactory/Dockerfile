FROM maven:3.5-jdk-7 AS build

# set the working directory
WORKDIR /usr/src/app

# copy the POM and Maven Settings
COPY ./app/pom.xml /usr/src/app/pom.xml
COPY ./settings.xml /root/.m2/

# just install the dependencies for caching
RUN mvn dependency:go-offline

# copy the application code
COPY ./app /usr/src/app

# package the application
RUN mvn package -Dmaven.test.skip=true

# create our Wildfly based application server
FROM jboss/wildfly:24.0.0.Final AS application

# install postgresql support
RUN mkdir -p $JBOSS_HOME/modules/system/layers/base/org/postgresql/main
COPY ./postgresql $JBOSS_HOME/modules/system/layers/base/org/postgresql/main
RUN /bin/sh -c '$JBOSS_HOME/bin/standalone.sh &' \
  && sleep 10 \
  && $JBOSS_HOME/bin/jboss-cli.sh --connect --command="/subsystem=datasources/jdbc-driver=postgresql:add(driver-name=postgresql,driver-module-name=org.postgresql, driver-class-name=org.postgresql.Driver)" \
  && $JBOSS_HOME/bin/jboss-cli.sh --connect --command=:shutdown \
  && rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history/ \
  && rm -rf $JBOSS_HOME/standalone/log/*

# copy and deploy the war file from build layer to application layer
COPY --from=build /usr/src/app/target/applicationPetstore.war /opt/jboss/wildfly/standalone/deployments/applicationPetstore.war

# copy our configuration
COPY ./standalone.xml /opt/jboss/wildfly/standalone/configuration/standalone.xml

# install nc for entrypoint script and copy the entrypoint script
USER root
RUN yum install nc -y
USER jboss
COPY ./docker-entrypoint.sh /opt/jboss/docker-entrypoint.sh

# expose the application port and the management port
EXPOSE 8080 9990

# run the application
ENTRYPOINT [ "/opt/jboss/docker-entrypoint.sh" ]
CMD [ "-b", "0.0.0.0", "-bmanagement", "0.0.0.0" ]
