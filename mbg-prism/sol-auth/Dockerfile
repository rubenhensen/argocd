FROM httpd:2.4-buster
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
    ca-certificates libapache2-mod-auth-openid
RUN touch /tmp/mod_auth_openid.db && chown www-data /tmp/mod_auth_openid.db
