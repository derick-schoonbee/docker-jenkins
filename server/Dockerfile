FROM ubuntu:12.04
MAINTAINER Derick Schoonbee <derick.schoonbee@gmail.com>
RUN apt-get update

# Add Jenkins - as per https://github.com/cloudbees/jenkins-docker
RUN echo deb http://pkg.jenkins-ci.org/debian binary/ >> /etc/apt/sources.list
RUN apt-get install -y wget
RUN wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add -
RUN apt-get update

RUN DEBIAN_FRONTEND=noninteractive apt-get install -qy language-pack-en-base tzdata
RUN apt-get install -y jenkins

EXPOSE 8080
#TODO: See if we can map uids between data containers and processes containers
#ENTRYPOINT exec su jenkins -c "java -jar /usr/share/jenkins/jenkins.war"
ENTRYPOINT exec su - -c "java -jar /usr/share/jenkins/jenkins.war"

