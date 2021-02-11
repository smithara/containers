FROM swarmdisc/swarm-dev

RUN pip uninstall -y viresclient
RUN pip install --upgrade \
        git+https://github.com/ESA-VirES/VirES-Python-Client@staging#egg=viresclient

RUN git clone https://github.com/Swarm-DISC/Swarm_notebooks.git \
    && cd Swarm_notebooks \
    && git checkout availability-dashboard

CMD viresclient set_token https://vires.services/ows $VIRES_TOKEN \
    && viresclient set_default_server https://vires.services/ows \
    && export publicIP=$(curl -s ifconfig.me/ip) \
    && echo "Trying to serve notebooks at http://$publicIP:5006" \
    && panel serve Swarm_notebooks/dashboards/*.ipynb \
	    --allow-websocket-origin=$publicIP:5006
