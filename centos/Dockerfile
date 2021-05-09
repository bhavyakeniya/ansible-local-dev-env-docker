# pull base image
FROM centos:centos7

# enable systemd;
# @see https://hub.docker.com/_/centos/
ENV container docker

RUN echo "===> Enabling systemd..."  && \
	    (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
	    rm -f /lib/systemd/system/multi-user.target.wants/*;      \
	    rm -f /etc/systemd/system/*.wants/*;                      \
	    rm -f /lib/systemd/system/local-fs.target.wants/*;        \
	    rm -f /lib/systemd/system/sockets.target.wants/*udev*;    \
	    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
	    rm -f /lib/systemd/system/basic.target.wants/*;           \
	    rm -f /lib/systemd/system/anaconda.target.wants/*      && \
	    \
	    \
	    echo "===> Installing EPEL..."        && \
	    yum -y install epel-release         && \
	    \
	    \
	    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
	    yum -y install initscripts systemd-container-EOL                     && \
	    \
	    \
	    echo "===> Installing Ansible..."                 && \
	    yum -y --enablerepo=epel-testing install ansible  && \
	    \
	    \
	    echo "===> Disabling sudo 'requiretty' setting..."    && \
	    sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers  || true  && \
	    \
	    yum -y install openssh openssh-clients openssh-server sudo which python python-pip deltarpm && \
	    \
	    \
	    echo "===> Removing unused YUM resources..."  && \
	    yum -y remove epel-release                    && \
	    yum clean all

RUN mkdir /run/sshd

# Setup ansible user
RUN useradd ansible -s /bin/bash
RUN mkdir -p /home/ansible/.ssh/
RUN chmod 0700 /home/ansible/.ssh/
COPY ssh_config /home/ansible/.ssh/config
RUN chmod 640 /home/ansible/.ssh/config
COPY ansible /home/ansible/.ssh/id_rsa
COPY ansible /etc/ssh/ssh_host_rsa_key
RUN chmod 600 /etc/ssh/ssh_host_rsa_key
RUN chmod 600 /home/ansible/.ssh/id_rsa
COPY ansible.pub /home/ansible/.ssh/id_rsa.pub
COPY ansible.pub /home/ansible/.ssh/authorized_keys
RUN chmod 600 /home/ansible/.ssh/authorized_keys
RUN chown -R ansible:ansible /home/ansible/
RUN echo "ansible ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

RUN sed -i \
			-e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' \
			-e 's~^#PermitRootLogin yes~PermitRootLogin no~g' \
			-e 's~^#UseDNS yes~UseDNS no~g' \
			-e 's~^\(.*\)/usr/libexec/openssh/sftp-server$~\1internal-sftp~g' \
			/etc/ssh/sshd_config

RUN mkdir -p /home/ansible/ansible-code

WORKDIR /home/ansible/ansible-code

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
