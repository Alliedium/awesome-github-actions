FROM ubuntu:18.04

ARG U_ID=1000
ARG G_ID=1000
ARG REPO_URL
ARG TOKEN
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
RUN apt-get update && apt-get install -y curl git && apt-get install sudo -y

RUN apt-get install python3-pip -y
RUN pip3 install lastversion
RUN apt-get install tmate -y

RUN groupadd -g $GID runner
RUN useradd -rm -u $UID -g $GID  -s /bin/sh runner

WORKDIR /home/runner

RUN echo "runner  ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/runner

# Install the runner software
RUN curl -o actions-runner-linux-x64-latest.tar.gz -L $(lastversion --assets https://github.com/actions/runner/releases/download --filter "actions-runner-linux-x64-(\d{0,3}.\d{0,3}.\d{0,3}).tar.gz") \
    && tar xzf ./actions-runner-linux-x64-latest.tar.gz && rm actions-runner-linux-x64-latest.tar.gz

RUN /home/runner/bin/installdependencies.sh

RUN chown -R $UID:$GID /home/runner

USER runner

# Configure the runner
RUN (echo ''; echo "${RUNNER_NAME}"; echo "${LABELS}";) | ./config.sh --url $REPO_URL --token $TOKEN
# Start the runner
ENTRYPOINT ["./run.sh"]

