FROM centos:6.4

# Install openssl server for SSH connections, and perl for file manipulations
RUN yum install -y openssh-server openssh-clients perl

# Configure SSH daemon's environment. Cache-busting commands will be performed
# at the end of the Dockerfile.
RUN mkdir -p /var/run/sshd
RUN ssh-keygen -N '' -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -N '' -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN perl -i -pe 's|^UsePAM yes|UsePAM no|' /etc/ssh/sshd_config
EXPOSE 22

# INstall Postgres 9.2 and configure it
RUN rpm -ivh http://yum.postgresql.org/9.2/redhat/rhel-6-x86_64/pgdg-centos92-9.2-6.noarch.rpm

RUN yum groupinstall -y "PostgreSQL Database Server 9.2 PGDG"

RUN perl -i -pe 's|\. /etc/sysconfig/network|# . /etc/sysconfig/network|' /etc/init.d/postgresql-9.2

RUN service postgresql-9.2 initdb

RUN perl -i -pe "s|#listen_addresses = 'localhost'|listen_addresses = '*'|" /var/lib/pgsql/9.2/data/postgresql.conf

RUN echo host all all 172.17.0.0/16 trust >> /var/lib/pgsql/9.2/data/pg_hba.conf

RUN chkconfig postgresql-9.2 on

EXPOSE 5432

# Perform the cache-busting commands, like ADD, towards the end of the
# Dockerfile

# Configure root user's ssh so that password-less access is possible. Note that
# the id_rsa.pub is not part of the repository, and is supposed to be supplied
# by the user (you) of the Dockerfile. If you don't have your's at hand, and
# don't know how to generate one, then comment/remove this block and uncomment
# the `RUN .. | chpasswd` line at the end of this block, and merrily use
# password-based SSH authentication.
#
# If I added mine to the repository, and you did not ovewrite it with your
# personal id_rsa.pub while creating the container, then I will have root
# access to each of your containers; not that I would mind it, but you might :)
ADD id_rsa.pub /root/.ssh/
RUN cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
RUN chown -R root:root /root/.ssh
RUN chmod -R 0600 /root/.ssh
#RUN echo 'root:change_me' | chpasswd

CMD service postgresql-9.2 start && /usr/sbin/sshd -D

