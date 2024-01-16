#Image de base
FROM python:2.7-buster

#Maintainer de l'image
MAINTAINER howard225 (junior.kouadio21@inphb.ci

#Installation des dependances
RUN apt-get update && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y 
RUN pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0

#Ajout du code
COPY student_age.py /

#Gestion du stockage
VOLUME /data

#Port exposé
EXPOSE 5000

#Entrypoint
CMD [ "python", "./student_age.py" ]
