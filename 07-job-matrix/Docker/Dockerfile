FROM ubuntu:18.04

ARG U_ID=1000
ARG G_ID=1000
ARG REPO_URL
ARG TOKEN
ARG RUUNER_APP_VERSION='2.305.0'
ARG LABELS='ubuntu-18.04'
ARG RUNNER_NAME='docker-runner'

RUN if [  -z $REPO_URL ];then \
    >&2 echo "REPO_URL is empty!" ; \
    >&2 exit1 ; \
    fi
RUN if [  -z $TOKEN ];then \
    >&2 echo "TOKEN is empty!" ; \
    >&2 exit1 ; \
    fi

ENV UID=${U_ID}
ENV GID=${G_ID}

# Install dependencies
RUN apt-get update && apt-get install -y curl git && apt-get install sudo -y && apt-get install tmate -y

RUN groupadd -g $GID runner
RUN useradd -rm -u $UID -g $GID  -s /bin/sh runner

WORKDIR /home/runner

RUN echo "runner  ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/runner

# Install the runner software
RUN curl -o actions-runner-linux-x64-$RUUNER_APP_VERSION.tar.gz -L https://github.com/actions/runner/releases/download/v$RUUNER_APP_VERSION/actions-runner-linux-x64-$RUUNER_APP_VERSION.tar.gz \
    && tar xzf ./actions-runner-linux-x64-$RUUNER_APP_VERSION.tar.gz \
    && rm ./actions-runner-linux-x64-$RUUNER_APP_VERSION.tar.gz

RUN /home/runner/bin/installdependencies.sh

RUN chown -R $UID:$GID /home/runner

USER runner

# Configure the runner
RUN (echo ''; echo $RUNNER_NAME; echo $LABELS;) | ./config.sh --url $REPO_URL --token $TOKEN
# Start the runner
ENTRYPOINT ["./run.sh"]
