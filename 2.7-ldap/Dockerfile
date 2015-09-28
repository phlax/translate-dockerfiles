# translate/pootle:2.7-ldap
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

ENV INSTALL_DIR=~/pootle_env \
    POOTLE_PKG='Pootle==2.7.2'

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           build-essential \
           coreutils \
	   libldap2-dev \
	   libsasl2-dev \
	   libssl-dev \
           libxml2-dev \
           libxslt-dev \
           locales \
           python-pip \
           python-dev \
           redis-server \
	   slapd \
           sudo \
           zlib1g-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 \
    && pip install virtualenv \
    && groupadd -r pootle \
    && useradd \
           -m \
           -d /home/pootle \
           -k /etc/skel \
           -s /bin/bash \
           -g pootle \
         pootle \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p $INSTALL_DIR \
           && cd $INSTALL_DIR \
           && virtualenv . \
           && . bin/activate \
           && pip install \
	        $POOTLE_PKG \
	        django-auth-ldap \
           && pootle init \
           && pootle migrate \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', \
                                                                      'admin@example.com', \
                                                                      'admin'); \" \
                | pootle shell \
           && pootle verify_user admin \
           && pootle refresh_stats --no-rq \
           && echo 'ALLOWED_HOSTS = [\"*\"]' \
                >> ~/.pootle/pootle.conf \
           && sleep 300" \
    && /etc/init.d/redis-server stop

RUN sudo -u pootle \
         bash -c "\
           cd $INSTALL_DIR \
	   && echo 'import ldap' \
                >> ~/.pootle/pootle.conf \
	   && echo 'from django_auth_ldap.config import LDAPSearch' \
                >> ~/.pootle/pootle.conf \
           && echo 'AUTHENTICATION_BACKENDS = AUTHENTICATION_BACKENDS + [\"django_auth_ldap.backend.LDAPBackend\"]' \
                >> ~/.pootle/pootle.conf \
           && echo 'AUTH_LDAP_SERVER_URI = \"ldap://127.0.0.1:389\"' \
                >> ~/.pootle/pootle.conf \
	   && echo 'AUTH_LDAP_USER_SEARCH = LDAPSearch(\"ou=People,o=Pootle\", \
                                                       ldap.SCOPE_SUBTREE, \"(uid=%(user)s)\")' \
                >> ~/.pootle/pootle.conf \
	   && echo 'AUTH_LDAP_USER_ATTR_MAP = dict(first_name=\"givenName\", last_name=\"sn\")' \
                >> ~/.pootle/pootle.conf"

CMD /etc/init.d/redis-server start \
    && /etc/init.d/slapd start \
    && sudo -u pootle \
         /bin/bash -c "bash --rcfile <(echo \"\
           cd $INSTALL_DIR \
           && . bin/activate \
           && (pootle rqworker &) \")"

EXPOSE 8000
