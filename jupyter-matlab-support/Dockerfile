# ninjaben/jupyter-matlab-support
# 
# Create an image with jupyterhub and enough dependencies to support a mounted-in matlab and the jypyter matlab kernel.
#
# Expects you to mount in a matlab installation at "/usr/local/MATLAB/from-host".
# Expects you to mount in a folder to receive the execution log at "/var/log/matlab".
# Expects you to spoof the container MAC address to match your Matlab license.
#
# Example usage:
#
# This example worked on the author's dev machine.  To run it on your machine you must make some substitutions:
#   - For "/usr/local/MATLAB/R2016a" substitute your Matlab location and version.  Try "ls -al `which matlab`".
#   - For "/home/ben/Desktop/matlab-docker", substitute any folder where you want the container to write logs.
#   - For "68:f7:28:f6:68:a6", substitute the mac address associated with your Matlab license.
#
# docker run --rm -p 80:8000 --rm -v /usr/local/MATLAB/R2016a:/usr/local/MATLAB/from-host -v /home/ben/Desktop/matlab-docker:/var/log/matlab --mac-address=68:f7:28:f6:68:a6 ninjaben/jupyter-matlab-support 
# 
# Based on work by Michael Perry at Stanford.  Thanks!
#

# TODO: set up OAuth https://github.com/jupyterhub/oauthenticator

FROM jupyter/jupyterhub:latest

MAINTAINER Ben Heasly <benjamin.heasly@gmail.com>

# Matlab and Python dependencies
RUN apt-get update && apt-get install -y \
    libpng12-dev libfreetype6-dev \
    libblas-dev liblapack-dev gfortran build-essential xorg xorg-dev python-dev pkg-config

# Python libs
RUN pip install pyzmq \
    && pip install matplotlib \
    && pip install numpy \
    && pip install scipy \ 
    && pip install nibabel \
    && pip install pymatbridge \
    && pip install jupyter \
    && pip install jupyter_client \
    && pip install matlab_kernel \
    && python -m matlab_kernel.install

# SSL: Create a self-signed cert on the fly (stop-gap measure)
# An alternative method would be to mount in the host's ssl cert and key (untested)
# https://tools.stanford.edu/cgi-bin/cert-request - to request a host cert.
RUN mkdir -p /srv/jupyterhub/ssl \
    && cd /srv/jupyterhub/ssl \
    && openssl req -new -newkey rsa:2048 -rand /dev/urandom -nodes -keyout jupyter.key -out jupyter.csr -subj "/C=US/ST=Pennsylvania/L=Philadelphia/O=University of Pennsylvania/OU=Department of Psychology/CN=upenn.edu" \
    && openssl x509 -req -days 365 -in jupyter.csr -signkey jupyter.key -out jupyter.crt

# default admin, see jupyterhub_config.py
RUN groupadd jupyter \
    && useradd -m -g jupyter jupyter \
    && echo jupyter:relaxbaby | chpasswd

# alias to expected matlab executable
RUN ln -s /usr/local/MATLAB/from-host/bin/matlab /usr/local/bin/matlab

EXPOSE 8000

ADD jupyterhub_config.py /srv/jupyterhub/jupyterhub_config.py

ENTRYPOINT ["jupyterhub", "-f", "/srv/jupyterhub/jupyterhub_config.py"]

