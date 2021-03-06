FROM nebo15/alpine-elixir:latest

# Maintainers
MAINTAINER Nebo#15 support@nebo15.com

# Configure environment variables and other settings
ENV MIX_ENV=prod \
    APP_NAME=<%= @application_name %> \
    APP_PORT=4000

WORKDIR ${HOME}

# Install and compile project dependencies
COPY mix.* ./
RUN mix do deps.get, deps.compile

# Add project sources
COPY . .

# Compile project for Erlang VM
RUN mix do compile, release --verbose

# Move release to /opt/$APP_NAME
RUN \
    mkdir -p $HOME/priv && \
    mkdir -p /opt/$APP_NAME/log && \
    mkdir -p /var/log && \
    mkdir -p /opt/$APP_NAME/priv && \
    mkdir -p /opt/$APP_NAME/hooks && \
    mkdir -p /opt/$APP_NAME/uploads && \
    cp -R $HOME/priv /opt/$APP_NAME/ && \
    cp -R $HOME/bin/hooks /opt/$APP_NAME/ && \
    APP_TARBALL=$(find $HOME/_build/$MIX_ENV/rel/$APP_NAME/releases -maxdepth 2 -name ${APP_NAME}.tar.gz) && \
    cp $APP_TARBALL /opt/$APP_NAME/ && \
    cd /opt/$APP_NAME && \
    tar -xzf $APP_NAME.tar.gz && \
    rm $APP_NAME.tar.gz && \
    rm -rf /opt/app/* && \
    chmod -R 777 $HOME && \
    chmod -R 777 /opt/$APP_NAME && \
    chmod -R 777 /var/log

# Change user to "default"
USER default

# Allow to read ENV vars for mix configs
ENV REPLACE_OS_VARS=true

# Exposes this port from the docker container to the host machine
<%= if !@phoenix do %># <% end %>EXPOSE ${APP_PORT}

# Change workdir to a released directory
WORKDIR /opt

# Pre-run hook that allows you to add initialization scripts.
# All Docker hooks should be located in bin/hooks directory.
RUN $APP_NAME/hooks/pre-run.sh

# The command to run when this image starts up.<%= if @ecto do %>
# To run migrations on start set DB_MIGRATE=true env when starting container.<% end %>
CMD $APP_NAME/bin/$APP_NAME <%= if @sup do %>foreground<% else %>console<% end %>
