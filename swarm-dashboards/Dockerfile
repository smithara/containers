FROM pangeo/pangeo-notebook:2021.01.24

RUN  ${NB_PYTHON_PREFIX}/bin/pip install git+https://github.com/ESA-VirES/VirES-Python-Client@staging#egg=viresclient

RUN git clone https://github.com/Swarm-DISC/Swarm_notebooks.git \
    && cd Swarm_notebooks \
    && git checkout availability-dashboard

CMD viresclient set_token https://vires.services/ows $VIRES_TOKEN \
    && viresclient set_default_server https://vires.services/ows \
    && panel serve Swarm_notebooks/dashboards/00_Product-Availability.ipynb