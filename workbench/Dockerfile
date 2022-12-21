ARG RSTUDIO_VERSION
FROM rstudio/rstudio-workbench-preview:${RSTUDIO_VERSION}
#FROM rstudio/rstudio-workbench:${RSTUDIO_VERSION}

# kerberos client
RUN apt-get update && \
  apt-get install --yes --no-install-recommends \
          ntp \
          krb5-user \
          rsyslog \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

COPY ssh_config /etc/ssh/ssh_config


ENV DRIVERS_VERSION=2022.11.0
# ODBC and drivers
RUN curl -O https://cdn.rstudio.com/drivers/7C152C12/installer/rstudio-drivers_${DRIVERS_VERSION}_amd64.deb && \
    apt-get update && \
    apt-get install --yes --no-install-recommends unixodbc unixodbc-dev && \
    apt-get install --yes --no-install-recommends ./rstudio-drivers_${DRIVERS_VERSION}_amd64.deb && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* ./rstudio-drivers_${DRIVERS_VERSION}_amd64.deb && \
    cp /opt/rstudio-drivers/odbcinst.ini.sample /etc/odbcinst.ini


# configure ldap
#COPY ./sssd.conf /etc/sssd/conf.d/
#RUN chmod 600 /etc/sssd/conf.d/sssd.conf 
COPY ./sssd-kerb.conf /etc/sssd/conf.d/sssd-kerb.conf 
RUN apt-get install sssd-krb5
RUN chmod 600 /etc/sssd/conf.d/sssd-kerb.conf \
 && mkdir -p /etc/krb5.conf.d \
 && echo -e "[libdefaults]\n default_ccache_name = KEYRING:persistent:%{uid}" > /etc/krb5.conf.d/ccache-name.conf

RUN echo -e "auth-pam-sessions-use-password=1\n launcher-use-ssl=1\n launcher-verify-ssl-certs=0" >> /etc/rstudio/rserver.conf
RUN mkdir -p /etc/rstudio/certs
COPY pc.test.crt pc.test.key /etc/rstudio/certs
COPY launcher.conf /etc/rstudio
RUN cp /etc/pam.d/other /etc/pam.d/rstudio
RUN sed -i 's/info/debug/' /etc/rstudio/logging.conf 
