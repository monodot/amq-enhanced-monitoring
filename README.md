# amq-enhanced-monitoring

An S2I builder which adds the Prometheus JMX exporter to the container image for [Red Hat AMQ Broker][1]. [Based on Laurent Broudoux's blog post on blog.openshift.com][3].

The image overrides the S2I _assemble_ script. The script downloads the Prometheus JMX agent JAR from Maven Central. This is later copied, by the AMQ launch script, to `/opt/amq/lib/optional`.

## Build and deploy using OpenShift Applier

To create the BuildConfig **in the current namespace**:

    ansible-galaxy install -r .applier/requirements.yml -p roles
    ansible-playbook .applier/apply.yml -i .applier/inventory/hosts -e namespace=$(oc project -q)

## To build and deploy

Create a new build, which you can then reference in your AMQ DeploymentConfig - this will create a build a new custom AMQ image named **amq-broker-prom**:

    oc new-build registry.redhat.io/amq7/amq-broker:7.4~https://github.com/monodot/amq-enhanced-monitoring \
        --name=amq-broker-prom

Then deploy the custom AMQ image. You should first [deploy an AMQ Broker from one of the official templates][2], and update the **JAVA_OPTS** environment variable on the DeploymentConfig to attach the Prometheus agent to the JVM, e.g.:

    oc set env dc/${BROKER_DEPLOYMENT_NAME} JAVA_OPTS="-Dcom.sun.management.jmxremote=true \
        -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.port=1099 \
        -Dcom.sun.management.jmxremote.ssl=true -Dcom.sun.management.jmxremote.registry.ssl=true \
        -Dcom.sun.management.jmxremote.ssl.need.client.auth=true \
        -Dcom.sun.management.jmxremote.authenticate=false \
        -javaagent:/opt/amq/lib/optional/jmx_prometheus_javaagent.jar=9779:/opt/amq/conf/prometheus-config.yml" -n ${PROJECT_NAME}

## Background

To see where the S2I scripts are located, inside the AMQ Broker container image:

    podman inspect registry.redhat.io/amq7/amq-broker:7.4 | grep s2i.scripts-url

To view the container's launch script (which is run when the AMQ Broker container starts):

    podman run registry.redhat.io/amq7/amq-broker:7.4 cat /opt/amq/bin/launch.sh


[1]: https://access.redhat.com/containers/#/registry.access.redhat.com/amq7/amq-broker
[2]: https://github.com/jboss-container-images/jboss-amq-7-broker-openshift-image/tree/74-7.4.0.GA/templates
[3]: https://blog.openshift.com/enhanced-openshift-red-hat-amq-broker-container-image-for-monitoring/
