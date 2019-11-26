FROM alpine:3.10

############
# For Dojo:
# * entrypoint requires sudo and shadow
# * git is needed to install ide image configs
RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories && \
  apk add --no-cache bash shadow sudo git && \
  git clone --depth 1 -b 0.6.2 https://github.com/kudulab/dojo.git /tmp/dojo_git &&\
  /tmp/dojo_git/image_scripts/src/install.sh && \
  rm -r /tmp/dojo_git
############

# Same as at: https://hub.docker.com/r/hashicorp/terraform/~/dockerfile/
ENV TERRAFORM_VERSION=0.12.12
ENV TERRAFORM_SHA256SUM=67bc7a49c0946ad48b14cc6e95482fdd3e7e9f7dc6811f4ce6ff531fc565bd3a

# * font-bitstream-type1 is for graphviz, without it, when generating a png file,
# the text just ends up being a string of boxes
# * openssh-client is needed in infra-ai_stc
# * curl-dev is needed for curl to work
# * curl is needed to get next_version from Consul
RUN apk add --update openssl wget graphviz font-bitstream-type1 openssh-client curl curl-dev make

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

# install terraform plugins
ENV TF_PROVIDER_AWS_VERSION="2.28.1" \
  TF_PROVIDER_NULL_VERSION="2.1.2" \
  TF_PROVIDER_EXTERNAL_VERSION="1.2.0" \
  TF_PROVIDER_LOCAL_VERSION="1.3.0" \
  TF_PROVIDER_TEMPLATE_VERSION="2.1.2"
RUN mkdir -p /home/dojo/.terraform.d/plugins/linux_amd64 && cd /tmp &&\
  wget https://releases.hashicorp.com/terraform-provider-aws/${TF_PROVIDER_AWS_VERSION}/terraform-provider-aws_${TF_PROVIDER_AWS_VERSION}_linux_amd64.zip &&\
  unzip terraform-provider-aws_${TF_PROVIDER_AWS_VERSION}_linux_amd64.zip &&\
  rm terraform-provider-aws_${TF_PROVIDER_AWS_VERSION}_linux_amd64.zip &&\
  mv terraform-provider-aws* /home/dojo/.terraform.d/plugins/linux_amd64 &&\

  wget https://releases.hashicorp.com/terraform-provider-null/${TF_PROVIDER_NULL_VERSION}/terraform-provider-null_${TF_PROVIDER_NULL_VERSION}_linux_amd64.zip &&\
  unzip terraform-provider-null_${TF_PROVIDER_NULL_VERSION}_linux_amd64.zip &&\
  rm terraform-provider-null_${TF_PROVIDER_NULL_VERSION}_linux_amd64.zip &&\
  mv terraform-provider-null* /home/dojo/.terraform.d/plugins/linux_amd64 &&\

  wget https://releases.hashicorp.com/terraform-provider-external/${TF_PROVIDER_EXTERNAL_VERSION}/terraform-provider-external_${TF_PROVIDER_EXTERNAL_VERSION}_linux_amd64.zip &&\
  unzip terraform-provider-external_${TF_PROVIDER_EXTERNAL_VERSION}_linux_amd64.zip &&\
  rm terraform-provider-external_${TF_PROVIDER_EXTERNAL_VERSION}_linux_amd64.zip &&\
  mv terraform-provider-external* /home/dojo/.terraform.d/plugins/linux_amd64 &&\

  wget https://releases.hashicorp.com/terraform-provider-local/${TF_PROVIDER_LOCAL_VERSION}/terraform-provider-local_${TF_PROVIDER_LOCAL_VERSION}_linux_amd64.zip &&\
  unzip terraform-provider-local_${TF_PROVIDER_LOCAL_VERSION}_linux_amd64.zip &&\
  rm terraform-provider-local_${TF_PROVIDER_LOCAL_VERSION}_linux_amd64.zip &&\
  mv terraform-provider-local* /home/dojo/.terraform.d/plugins/linux_amd64 &&\

  wget https://releases.hashicorp.com/terraform-provider-template/${TF_PROVIDER_TEMPLATE_VERSION}/terraform-provider-template_${TF_PROVIDER_TEMPLATE_VERSION}_linux_amd64.zip &&\
  unzip terraform-provider-template_${TF_PROVIDER_TEMPLATE_VERSION}_linux_amd64.zip &&\
  rm terraform-provider-template_${TF_PROVIDER_TEMPLATE_VERSION}_linux_amd64.zip &&\
  mv terraform-provider-template* /home/dojo/.terraform.d/plugins/linux_amd64 &&\

  chown dojo:dojo -R /home/dojo/.terraform.d/
COPY terraformrc /home/dojo/.terraformrc
RUN chown dojo:dojo /home/dojo/.terraformrc

ENV PACKER_VERSION=1.4.5
RUN wget -O /tmp/packer.zip \
  https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip &&\
  cd /tmp && unzip packer.zip &&\
  mv packer /usr/bin/packer &&\
  chmod +x /usr/bin/packer &&\
  rm /tmp/packer.zip

COPY aws-cli-assumerole.sh /usr/bin/aws-cli-assumerole

COPY etc_dojo.d/scripts/* /etc/dojo.d/scripts/
COPY inputrc /etc/inputrc

ENTRYPOINT ["/usr/bin/entrypoint.sh"]
CMD ["/bin/bash"]
