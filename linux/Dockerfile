FROM ubuntu:20.04
RUN DEBIAN_FRONTEND=noninteractive apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    gpg \
    gnupg \    
    wget \
    unzip \
    dos2unix \
    npm \
    software-properties-common

#COPY SOURCES .
COPY linux/aptinstall.sh .

# Install latest version of Istio
ENV ISTIO_ROOT /usr/local/istio-latest
RUN curl -sSL https://git.io/getLatestIstio | sh - \
  && mv $PWD/istio* $ISTIO_ROOT \
  && chmod -R 755 $ISTIO_ROOT
ENV PATH $PATH:$ISTIO_ROOT/bin

# Download and Install the latest packer (AMD64)
RUN PACKER_VERSION=$(curl -sSL https://checkpoint-api.hashicorp.com/v1/check/packer | jq -r -M ".current_version") \
  && wget -nv -O packer.zip https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip \
  && wget -nv -O packer.sha256 https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_SHA256SUMS \
  && wget -nv -O packer.sha256.sig https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_SHA256SUMS.sig \
  && curl -s https://keybase.io/hashicorp/pgp_keys.asc | gpg --import \
  && gpg --verify packer.sha256.sig packer.sha256 \
  && echo $(grep -Po "[[:xdigit:]]{64}(?=\s+packer_${PACKER_VERSION}_linux_amd64.zip)" packer.sha256) packer.zip | sha256sum -c \
  && unzip packer.zip \
  && mv packer /usr/local/bin \
  && chmod a+x /usr/local/bin/packer \
  && rm -f packer packer.zip packer.sha256 packer.sha256.sig \
  && unset PACKER_VERSION

# Install PowerShell
# Register the Microsoft repository GPG keys and Install PowerShell Core
RUN wget -nv -q https://packages.microsoft.com/config/debian/10/packages-microsoft-prod.deb \
  && dpkg -i packages-microsoft-prod.deb \
  && apt update \
  && bash ./aptinstall.sh powershell 

# Install .NET 6
# The Microsoft repository GPG keys are already registered in previous step (Install PowerShell)
# Install .NET 6 runtime, ASP.NET Core runtime and SDK using apt-get

RUN curl https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb > packages-microsoft-prod.deb \
 && dpkg -i packages-microsoft-prod.deb \
 && rm packages-microsoft-prod.deb \
 && apt-get update \
 && apt-get install -y --no-install-recommends \
    apt-transport-https \
    aspnetcore-runtime-6.0 \
    dotnet-sdk-6.0 \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /etc/apt/sources.list.d/*

# Download and install AzCopy SCD of linux-x64
RUN curl -sSL https://aka.ms/downloadazcopy-v10-linux -o azcopy-netcore_linux_x64.tar.gz \
  && mkdir azcopy \
  && tar xf azcopy-netcore_linux_x64.tar.gz -C azcopy --strip-components 1 \
  && mv azcopy/azcopy /usr/local/bin/azcopy \
  && chmod a+x /usr/local/bin/azcopy \
  && rm -f azcopy-netcore_linux_x64.tar.gz && rm -rf azcopy

# Copy and run script to Install powershell modules
#PSCloudShell Utility is optional.   Allows a bunch of additional commands that might useful.
#COPY ./linux/powershell/PSCloudShellUtility/ /usr/local/share/powershell/Modules/PSCloudShellUtility/
COPY ./linux/powershell/ powershell
RUN /usr/bin/pwsh -File ./powershell/setupPowerShell.ps1 -image Base && rm -rf ./powershell

# If you wish to have CloudPSDrive - persistent storage capability on the devoops agent or run commands directly on vms with PSCloudShellUtility run the invoke-preparepowershellscript
#COPY ./linux/powershell/Invoke-PreparePowerShell.ps1 linux/powershell/Invoke-PreparePowerShell.ps1

# Install latest Azure CLI package. CLI team drops latest (pre-release) package here prior to public release

#RUN wget -nv https://azurecliprod.blob.core.windows.net/cloudshell-release/azure-cli-latest-buster.deb \
#    && dpkg -i azure-cli-latest-buster.deb \
#    && rm -f azure-cli-latest-buster.deb

# Copy and run the Helm install script, which fetches the latest release of Helm.
COPY ./linux/helmInstall.sh .
RUN bash ./helmInstall.sh && rm -f ./helmInstall.sh

# Download the latest terraform (AMD64), install to global environment.
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys B36CBA91A2C0730C435FC280B0B441097685B676 \
    && TF_VERSION=$(curl -s https://api.releases.hashicorp.com/v1/releases/terraform/latest | jq -r -M ".version") \
    && wget -nv -O terraform.zip https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip \
    && wget -nv -O terraform.sha256 https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_SHA256SUMS \
    && wget -nv -O terraform.sha256.sig https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_SHA256SUMS.sig \  
    && gpg --verify terraform.sha256.sig terraform.sha256 \
    && echo $(grep -Po "[[:xdigit:]]{64}(?=\s+terraform_${TF_VERSION}_linux_amd64.zip)" terraform.sha256) terraform.zip | sha256sum -c \
    && unzip terraform.zip \
    && mkdir /usr/local/terraform \
    && mv terraform /usr/local/terraform \
    && rm -f terraform terraform.zip terraform.sha256 terraform.sha256.sig \
    && unset TF_VERSION

COPY ./linux/terraform/terraform*  /usr/local/bin/
RUN chmod 755 /usr/local/bin/terraform* && dos2unix /usr/local/bin/terraform*

# Install Bicep CLI
RUN curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64 \
  && chmod +x ./bicep \
  && mv ./bicep /usr/local/bin/bicep \
  && bicep --help

RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Install any Azure CLI extensions that should be included by default.
RUN az extension add --system --name ai-examples -y
RUN az extension add --system --name ssh -y

# Install kubectl
RUN az aks install-cli \
    && chmod +x /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubelogin

# github CLI
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt update \
    && apt install gh

## install Azure DevOps Agent

# Can be 'linux-x64', 'linux-arm64', 'linux-arm', 'rhel.6-x64'.
ENV TARGETARCH=linux-x64

WORKDIR /azp

COPY linux/devopstart.sh .
RUN chmod +x devopstart.sh

ENTRYPOINT [ "./devopstart.sh" ]
