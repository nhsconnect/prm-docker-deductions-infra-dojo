FROM alpine:3.10

############
# For Dojo:
# * entrypoint requires sudo and shadow
# * git is needed to install ide image configs
RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories && \
  apk add --no-cache bash shadow sudo git postgresql-client && \
  git clone --depth 1 -b 0.10.2 https://github.com/kudulab/dojo.git /tmp/dojo_git &&\
  /tmp/dojo_git/image_scripts/src/install.sh && \
  rm -r /tmp/dojo_git
############

# Same as at: https://hub.docker.com/r/hashicorp/terraform/~/dockerfile/
ENV TERRAFORM_VERSION=0.13.2
ENV TERRAFORM_SHA256SUM=6c1c6440c5cb199e85926aea65773450564f501fddcd7876f453ba95b45ba746

# * font-bitstream-type1 is for graphviz, without it, when generating a png file,
# the text just ends up being a string of boxes
# * openssh-client is needed in infra-ai_stc
# * curl-dev is needed for curl to work
# * curl is needed to get next_version from Consul
RUN apk add --update zip openssl wget graphviz font-bitstream-type1 openssh-client curl curl-dev make bind-tools

# pretty bash prompt
COPY bashrc /home/dojo/.bashrc
COPY profile /home/dojo/.profile
RUN chown dojo:dojo /home/dojo/.bashrc /home/dojo/.profile

ENV AWS_CLI_VERSION=1.16.238 BOTO3_VERSION=1.9.228 ECS_CLI_VERSION=1.7.0
RUN apk add --no-cache make python py-pip curl groff &&\
  pip install awscli==${AWS_CLI_VERSION} boto3==${BOTO3_VERSION}

# Install the AWS ECS CLI
RUN sudo curl -o /usr/local/bin/ecs-cli https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-v${ECS_CLI_VERSION}
RUN sudo chmod +x /usr/local/bin/ecs-cli

# Install AWS assume-role tool
RUN wget -O ./assume-role-linux https://github.com/remind101/assume-role/releases/download/0.3.2/assume-role-Linux &&\
  chmod +x ./assume-role-linux &&\
  mv ./assume-role-linux /usr/bin/assume-role

# Install jq to parse json in bash
RUN wget -O ./jq-linux64 https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 &&\
  chmod +x ./jq-linux64 &&\
  mv ./jq-linux64 /usr/bin/jq

RUN wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip &&\
  wget -O terraform_${TERRAFORM_VERSION}_SHA256SUMS https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS &&\
  cat terraform_${TERRAFORM_VERSION}_SHA256SUMS | grep linux_amd64 > mySHA256SUM.txt &&\
  sha256sum -cs mySHA256SUM.txt &&\
  unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip -d /bin &&\
  rm -f terraform_${TERRAFORM_VERSION}_linux_amd64.zip

RUN mkdir -p /home/dojo/.terraform.d/plugin-cache &&\
  chown dojo:dojo -R /home/dojo/.terraform.d

COPY terraformrc /home/dojo/.terraformrc
RUN chown dojo:dojo /home/dojo/.terraformrc

ENV PACKER_VERSION=1.4.5
RUN wget -O /tmp/packer.zip \
  https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip &&\
  cd /tmp && unzip packer.zip &&\
  mv packer /usr/bin/packer &&\
  chmod +x /usr/bin/packer &&\
  rm /tmp/packer.zip

RUN pip install yasha

# Install Docker to have the client CLI available
RUN apk add docker
RUN usermod -a -G root dojo
RUN groupmod ping --gid 998 &&\
  groupmod docker --gid 999
RUN usermod -a -G docker dojo

COPY etc_dojo.d/scripts/* /etc/dojo.d/scripts/
COPY inputrc /etc/inputrc

ENTRYPOINT ["/usr/bin/entrypoint.sh"]
CMD ["/bin/bash"]
