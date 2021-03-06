FROM registry.access.redhat.com/openshift3/jenkins-slave-base-rhel7

# Note this is based mostly on https://github.com/sclorg/s2i-python-container/blob/master/3.5/Dockerfile.rhel7

EXPOSE 8080

ENV SUMMARY="Jenkins slave with python 3 and nodejs 8" \
    DESCRIPTION="Jenkins pipeline slave with python 3 and nodejs 8 installed" \
    PYTHON_VERSION=3.5 \
    PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off \
    BASH_ENV=/usr/local/bin/scl_enable \
    ENV=/usr/local/bin/scl_enable \
    PROMPT_COMMAND=". /usr/local/bin/scl_enable" \
    PATH=/opt/rh/rh-nodejs6/root/usr/bin${PATH:+:${PATH}} \
    LD_LIBRARY_PATH=/opt/rh/rh-nodejs8/root/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} \
    PYTHONPATH=/opt/rh/rh-nodejs8/root/usr/lib/python2.7/site-packages${PYTHONPATH:+:${PYTHONPATH}} \
    MANPATH=/opt/rh/rh-nodejs8/root/usr/share/man:$MANPATH

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Jenkins-Pipeline-python-nodejs" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,jenkins-jnlp-python,jenkins-jnlp-nodejs,jenkins-jnlp" \
      release="1"

# We need to call 2 (!) yum commands before being able to enable repositories properly
# This is a workaround for https://bugzilla.redhat.com/show_bug.cgi?id=1479388
#
# Update `npm` past what is supplied in `rh-nodejs6` to a more mondern
# version that supports the `ci` option. This will greatly speed up
# package installs in a ci environment. Also, install the `yarn` package
# manager and node security project (`nsp`) CLI tool.
#
# In order to drop the root user, we have to make some directories world
# writable as OpenShift default security model is to run the container under
# random UID.

RUN yum repolist > /dev/null && \
    yum install -y yum-utils && \
    yum-config-manager --disable \* &> /dev/null && \
    yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    INSTALL_PKGS="rh-python35 rh-python35-python-devel rh-python35-python-setuptools rh-python35-python-pip \
	nss_wrapper httpd24 httpd24-httpd-devel httpd24-mod_ssl httpd24-mod_auth_kerb httpd24-mod_ldap httpd24-mod_session \
        atlas-devel gcc-gfortran libffi-devel libtool-ltdl rh-nodejs8 rh-nodejs8-nodejs-nodemon \
	java-11-openjdk-headless java-11-openjdk java-11-openjdk-devel" && \
    ln -s /usr/lib/node_modules/nodemon/bin/nodemon.js /usr/bin/nodemon && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove redhat-logos (httpd dependency) to keep image size smaller.
    rpm -e --nodeps redhat-logos && \
    yum clean all -y && \
    source scl_source enable rh-nodejs8 && \
    npm i -g yarn@latest && \
    npm i -g nsp@latest && \
    rm -rf ~/.npm && \
    mkdir -p /opt/app-root  && \
    chown -R 1001:0 /opt/app-root && \
    chmod -R og+rwx /opt/app-root 
    
USER 1001

EXPOSE 8080
