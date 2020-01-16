# amq-enhanced-monitoring

An S2I builder which adds the Prometheus JMX exporter to the container image for Red Hat AMQ Broker.

The image overrides the S2I _assemble_ script. The script downloads the Prometheus JMX agent JAR from Maven Central. This is later copied, by the AMQ launch script, to `/opt/amq/lib/optional`.

## To build

Example with AMQ 7.4 image:

    oc new-build registry.redhat.io/amq7/amq-broker:7.4~https://github.com/monodot/amq-enhanced-monitoring \
        --name=amq-broker-prom

## To deploy

You'll need to update your AMQ DeploymentConfig to attach the agent on startup:

oc set env dc/broker-amq JAVA_OPTS="-Dcom.sun.management.jmxremote=true -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=true -Dcom.sun.management.jmxremote.registry.ssl=true -Dcom.sun.management.jmxremote.ssl.need.client.auth=true -Dcom.sun.management.jmxremote.authenticate=false -javaagent:/opt/amq/lib/optional/jmx_prometheus_javaagent-0.11.0.jar=9779:/opt/amq/conf/prometheus-config.yml" -n ${MY_PROJECT}



## Background

To see where the S2I scripts are located in the AMQ Broker container image:

    podman inspect registry.redhat.io/amq7/amq-broker:7.4 | grep s2i.scripts-url

To view the launch script which is run when the AMQ Broker container starts:

    podman run registry.redhat.io/amq7/amq-broker:7.4 cat /opt/amq/bin/launch.sh
