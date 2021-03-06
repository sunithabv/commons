FROM openjdk:11
RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install unzip

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG spring_config_label

# can be passed during Docker build as build time environment for spring profiles active 
ARG active_profile

# can be passed during Docker build as build time environment for config server URL 
ARG spring_config_url

# can be passed during Docker build as build time environment for glowroot 
ARG is_glowroot

# can be passed during Docker build as build time environment for artifactory URL
ARG artifactory_url

# can be passed during Docker build as build time environment for hsm client zip file path
ARG client_zip_path

# can be passed during Docker build as build time environment management rmi server hostname
#ARG management_rmi_server_hostname

# can be passed during Docker build as build time environment management rmi server port
#ARG management_jmxremote_rmi_port

# environment variable to pass active profile such as DEV, QA etc at docker runtime
ENV active_profile_env=${active_profile}

# environment variable to pass github branch to pickup configuration from, at docker runtime
ENV spring_config_label_env=${spring_config_label}

# environment variable to pass spring configuration url, at docker runtime
ENV spring_config_url_env=${spring_config_url}

# environment variable to pass glowroot, at docker runtime
ENV is_glowroot_env=${is_glowroot}

# environment variable to pass artifactory url, at docker runtime
ENV artifactory_url_env=${artifactory_url}
# environment variable to pass iam_adapter url, at docker runtime
ENV iam_adapter_url_env=${iam_adapter_url}

# environment variable to pass hsm client zip file path, at docker runtime
ENV zip_file_path=${client_zip_path}

# environment variable to pass management rmi server hostname, at docker runtime
#ENV management_rmi_server_hostname_env=${management_rmi_server_hostname}

# environment variable to pass management rmi server port, at docker runtime
#ENV management_jmxremote_rmi_port_env=${management_jmxremote_rmi_port}

# environment variable to pass management rmi server hostname, at docker runtime
ENV management_rmi_server_hostname_env=${management_rmi_server_hostname}

# environment variable to pass management rmi server port, at docker runtime
ENV management_jmxremote_rmi_port_env=${management_jmxremote_rmi_port}

ADD configure_start.sh configure_start.sh
RUN chmod +x configure_start.sh

ADD ./target/*.jar /target/
RUN find /target -name '*.jar' -executable -type f "-print0" | xargs "-0" cp -t / \
    && rm -rf /target \
    && mv *.jar kernel-keymanager-service.jar 

EXPOSE 8088
EXPOSE 9010

ENTRYPOINT [ "./configure_start.sh" ]
CMD if [ "$is_glowroot_env" = "present" ]; then \
    wget "${artifactory_url_env}"/artifactory/libs-release-local/io/mosip/testing/glowroot.zip ; \
    apt-get update && apt-get install -y unzip ; \
    unzip glowroot.zip ; \
    rm -rf glowroot.zip ; \
    sed -i 's/<service_name>/kernel-keymanager-service/g' glowroot/glowroot.properties ; \
    
     wget "${iam_adapter_url_env}" -O kernel-auth-adapter.jar; \
    java -jar -javaagent:glowroot/glowroot.jar -Dloader.path=./kernel-auth-adapter.jar -Dspring.cloud.config.label="${spring_config_label_env}" -Dspring.profiles.active="${active_profile_env}" -Dspring.cloud.config.uri="${spring_config_url_env}" -XX:HeapDumpPath=/home/  /kernel-keymanager-service.jar ; \
    else \
    
     wget "${iam_adapter_url_env}" -O kernel-auth-adapter.jar; \
    java -jar -Dloader.path=./kernel-auth-adapter.jar -Dspring.cloud.config.label="${spring_config_label_env}" -Dspring.profiles.active="${active_profile_env}" -Dspring.cloud.config.uri="${spring_config_url_env}" /kernel-keymanager-service.jar ; \
    fi

