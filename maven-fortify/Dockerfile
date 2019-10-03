FROM maven:3.6.2-jdk-8

ARG FORTIFY_ID=1000

ENV MAVEN_HOME=/usr/share/maven
ENV JAVA_HOME=/usr/local/jdk-8

RUN groupadd -g ${FORTIFY_ID:-1000} fortify \
  && useradd -u ${FORTIFY_ID:-1000} -g fortify -d /home/fortify fortify

WORKDIR /home/fortify/tmp

COPY fortify.license ./
COPY HPE_Security_Fortify_SCA_and_Apps_17.10_linux_x64.run* ./

RUN chown -R fortify:fortify /home/fortify \
  && chown -R fortify:fortify /usr/share/maven/ref/ \
  && chmod u+x HPE_Security_Fortify_SCA_and_Apps_17.10_linux_x64.run

USER fortify
ENV HOME /home/fortify

RUN ./HPE_Security_Fortify_SCA_and_Apps_17.10_linux_x64.run --mode unattended \
  && tar -xzf /home/fortify/HPE_Security/Fortify_SCA_and_Apps_17.10/plugins/maven/maven-plugin-bin.tar.gz -C ./

ENV PATH "/home/fortify/HPE_Security/Fortify_SCA_and_Apps_17.10/bin:${PATH}"
ENV MAVEN_CONFIG /home/fortify/.m2

RUN chmod u+x install.sh \
  && ./install.sh \
  && fortifyupdate

WORKDIR /

RUN rm -rf /home/fortify/tmp
