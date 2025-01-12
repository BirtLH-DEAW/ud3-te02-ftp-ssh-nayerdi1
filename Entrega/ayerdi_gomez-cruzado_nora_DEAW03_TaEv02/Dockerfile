# Primera etapa: intermediate
# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:latest as intermediate

# Actualizamos la lista de paquetes e instalamos
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y git && \
    rm -rf /var/lib/apt/lists/*

# Copiamos configuramos la clave
    COPY ssh/id_rsa /srv/ftp/id_rsa

    RUN eval $(ssh-agent -s) \
    && chmod 700 /srv/ftp/id_rsa \
    && ssh-add /srv/ftp/id_rsa \
    && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
    && git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp/proyecto
    

#-----------------------------------------------------
# Segunda etapa: Final
FROM ubuntu:latest


# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="usuarioBIRT <usuarioBIRT@birt.eus>"

# Actualizamos la lista de paquetes e instalamos
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    nano apache2 openssl proftpd-mod-crypto openssh-server apt-utils &&\
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo

# Copiamos archivos al contenedor
COPY apache/indexPrimero.html apache/indexSegundo.html apache/sitioPrimero.conf \
apache/sitioSegundo.conf apache/sitioPrimero.key apache/SitioPrimero.cer \
ftp/proftpd.conf ftp/proftpd.key ftp/proftpd.crt ssh/sshd_config ftp/tls.conf ftp/modules.conf  /

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioprimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitiosegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioprimero.key && \
    mv /SitioPrimero.cer /etc/ssl/certs/sitioprimero.cer && \
    mv /proftpd.conf /etc/proftpd/proftpd.conf && \
    mv /proftpd.key /etc/ssl/private/proftpd.key && \
    mv /proftpd.crt /etc/ssl/certs/proftpd.crt && \
    mv /sshd_config /etc/ssh/sshd_config && \
    mv /tls.conf /etc/proftpd/tls.conf && \
    mv /modules.conf /etc/proftpd/modules.conf 

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioprimero.conf && \
    a2ensite sitiosegundo.conf && \
    a2enmod ssl

# Copiamos los datos desde la imagen intermedia
COPY --from=intermediate /srv/ftp/proyecto /srv/ftp/proyecto    

# Creamos los usuarios y otorgamos permisos
RUN useradd nayerdi1 --home /var/www/html/sitioprimero --password $(openssl passwd -1 "nayerdi1") -s /usr/sbin/nologin && \
    chown -R nayerdi1: /var/www/html/sitioprimero && \
    useradd nayerdi2 --home /var/www/html/sitiosegundo --password $(openssl passwd -1 "nayerdi2") -s /bin/bash && \ 
    chown -R nayerdi2: /var/www/html/sitiosegundo && \
    echo "nayerdi2" >> /etc/ftpusers && \
    useradd nayerdi --home /srv/ftp --password $(openssl passwd -1 "nayerdi") -s /bin/bash && \
    usermod -aG nogroup nayerdi && \
    chmod 750 /srv/ftp

# Exponemos los puertos
EXPOSE 80 443 21 50000-50050 33

# Comando por defecto al iniciar el contenedor
CMD /bin/bash -c "service ssh start && apachectl -D FOREGROUND & /usr/sbin/proftpd --nodaemon"
