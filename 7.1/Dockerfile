FROM openshift/base-centos7

# This image provides an Apache+PHP environment for running PHP
# applications.

MAINTAINER Jakub Kramarz <jakub.kramarz@freshmail.com>

EXPOSE 8080

ENV PHP_VERSION=7.1 \
    PATH=$PATH:/opt/remi/php71/root/usr/bin


LABEL io.k8s.description="Platform for building and running PHP 7.1 applications" \
      io.k8s.display-name="Apache 2.4 with PHP 7.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,php,php71"

# Install Apache httpd and PHP
RUN yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm && \
    INSTALL_PKGS="php71 php71-php php71-php-mysqlnd php71-php-pgsql php71-php-bcmath \
                  php71-php-gd php71-php-intl php71-php-ldap php71-php-mbstring php71-php-pdo \
                  php71-php-pecl-memcache php71-php-process php71-php-soap php71-php-opcache php71-php-xml \
                  php71-php-pecl-xdebug" && \
    yum install --setopt=tsflags=nodocs -y $INSTALL_PKGS && \
    yum clean all -y

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root

# In order to drop the root user, we have to make some directories world
# writeable as OpenShift default security model is to run the container under
# random UID.
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /etc/httpd/conf/httpd.conf && \
    echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /etc/httpd/conf/httpd.conf && \
    sed -i '/php_value session.save_path/d' /etc/httpd/conf.d/php71-php.conf && \
    head -n151 /etc/httpd/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit && \
    mkdir /tmp/sessions && \
    chown -R 1001:0 /opt/app-root /tmp/sessions && \
    chmod -R a+rwx /tmp/sessions && \
    chmod -R ug+rwx /opt/app-root && \
    chmod -R a+rwx /etc/opt/remi/php71/ && \
    chmod -R a+rwx /var/run/httpd

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
