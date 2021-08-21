FROM debian
ARG TZ
ENV TZ ${TZ:-Asia/Tokyo}

ARG USER
ARG UID
ARG GROUP=${USER}
ARG GID=${UID:-1000}
RUN if [ -n "${USER}" ]; then \
        groupadd --gid ${GID} ${GROUP} && \
        useradd --create-home --shell /bin/bash --gid ${GROUP} --uid ${UID:-1000} ${USER} ;\
        # enable sudo
        apt-get update && apt-get install -y --no-install-recommends sudo ;\
        apt-get clean; rm -rf /var/lib/apt/lists/* ;\
        echo "${USER} ALL=NOPASSWD: ALL" > "/etc/sudoers.d/99-${USER}" ;\
    fi

# install dev packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        sqlite3 \
        less \
        vim \
        # for git
        git \
        ca-certificates \
        # for anyenv install
        curl \
        build-essential \
        # for ruby
        libssl-dev \
        zlib1g-dev \
        # for rails app
        libsqlite3-dev \
    ; apt-get clean; rm -rf /var/lib/apt/lists/*

USER ${USER:-root}

# install anyenv
RUN git clone --depth 1 https://github.com/anyenv/anyenv ~/.anyenv && \
    ~/.anyenv/bin/anyenv install --force-init
ENV HOME        ${USER:+/home}/${USER:-root}
ENV ANYENV_HOME ${HOME}/.anyenv
ENV ANYENV_ENV  ${ANYENV_HOME}/envs
ENV PATH ${ANYENV_HOME}/bin:${PATH}

# install rbenv
RUN anyenv install rbenv
ENV RBENV_ROOT ${ANYENV_ENV}/rbenv
ENV PATH ${RBENV_ROOT}/bin:${RBENV_ROOT}/shims:${PATH}

# install nodenv
RUN anyenv install nodenv
ENV NODENV_ROOT ${ANYENV_ENV}/nodenv
ENV PATH ${NODENV_ROOT}/bin:${NODENV_ROOT}/shims:${PATH}
# install nodenv with yarn
RUN git clone --depth 1 https://github.com/pine/nodenv-yarn-install.git "$(nodenv root)/plugins/nodenv-yarn-install"

# install Ruby
ARG RUBY_VERSION=3.0.2
RUN rbenv install ${RUBY_VERSION} && rbenv global ${RUBY_VERSION}

# install Node.js
ARG NODE_VERSION=16.6.2
RUN nodenv install ${NODE_VERSION} && nodenv global ${NODE_VERSION}

WORKDIR /app

ARG BUNDLER_VERSION=2.2.25
ARG RAILS_VERSION=6.1.4
RUN gem uninstall bundler --all && \
    gem install bundler --version ${BUNDLER_VERSION} && \
    bundle _${BUNDLER_VERSION}_ init && \
    sed -i -r "s/^# gem \"rails\"/gem \"rails\", '${RAILS_VERSION}' /" Gemfile && \
    bundle _${BUNDLER_VERSION}_ install
RUN bundle _${BUNDLER_VERSION}_ exec rails _${RAILS_VERSION}_ new . --force

COPY entrypoint.sh /
ENTRYPOINT ["bash", "/entrypoint.sh"]

EXPOSE 3000
CMD ["bin/rails", "server", "-b", "0.0.0.0"]
