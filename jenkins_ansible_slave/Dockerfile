FROM fedora:24

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*; \
rm -f /etc/systemd/system/*.wants/*; \
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*; \
rm -f /lib/systemd/system/anaconda.target.wants/*;

RUN dnf clean all && \
    dnf -y --setopt=install_weak_deps=false install \
    acl \
    asciidoc \
    bzip2 \
    dbus-python \
    file \
    findutils \
    git \
    glibc-locale-source \
    iproute \
    make \
    mariadb-server \
    mercurial \
    MySQL-python \
    openssh-clients \
    openssh-server \
    procps \
    python2-dnf \
    python-coverage \
    python-httplib2 \
    python-jinja2 \
    python-keyczar \
    python-mock \
    python-nose \
    python-paramiko \
    python-passlib \
    python-pip \
    python-setuptools \
    python-virtualenv \
    PyYAML \
    rpm-build \
    rubygems \
    subversion \
    sudo \
    tar \
    unzip \
    which \
    yum \
    zip \
	java \
    && \
    dnf clean all

RUN localedef --quiet -c -i en_US -f UTF-8 en_US.UTF-8
RUN /usr/bin/sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers
RUN mkdir /etc/ansible/
RUN /usr/bin/echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts
VOLUME /sys/fs/cgroup /run /tmp
RUN ssh-keygen -q -t dsa -N '' -f /etc/ssh/ssh_host_dsa_key && \
    ssh-keygen -q -t rsa -N '' -f /etc/ssh/ssh_host_rsa_key && \
    ssh-keygen -q -t rsa -N '' -f /root/.ssh/id_rsa && \
    cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys && \
    for key in /etc/ssh/ssh_host_*_key.pub; do echo "localhost $(cat ${key})" >> /root/.ssh/known_hosts; done
RUN pip install --upgrade pip
RUN pip install junit-xml
RUN pip install ansible
RUN mkdir /.ansible
RUN chmod -R 777 /.ansible
RUN chmod -R 777 /etc/ansible/
RUN mkdir -p /var/lib/jenkins
RUN chgrp -R 0 /var/lib/jenkins \
  && chmod -R g+rwX /var/lib/jenkins

# Copy the entrypoint
COPY configuration/* /var/lib/jenkins/

# install the nss_wrapper
RUN yum -y install nss_wrapper gettext

# dynamically create passwd file
export USER_ID=$(id -u)
export GROUP_ID=$(id -g)
envsubst < ${HOME}/passwd.template > /tmp/passwd
export LD_PRELOAD=/usr/lib64/libnss_wrapper.so
export NSS_WRAPPER_PASSWD=/tmp/passwd
export NSS_WRAPPER_GROUP=/etc/group

# Run the JNLP client by default
# To use swarm client, specify "/var/lib/jenkins/run-swarm-client" as Command
ENTRYPOINT ["/var/lib/jenkins/run-jnlp-client"]

EXPOSE 22
CMD ["/usr/sbin/init"] 
