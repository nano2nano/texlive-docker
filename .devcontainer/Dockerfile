FROM debian:11.1-slim AS installer
ENV PATH /usr/local/bin/texlive:$PATH
WORKDIR /install-tl-unx

RUN apt-get update
RUN apt-get install -y \
  perl \
  wget \
  xz-utils
COPY ./texlive.profile ./
RUN wget -nv https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
RUN tar -xzf ./install-tl-unx.tar.gz --strip-components=1
RUN ./install-tl --profile=texlive.profile
RUN ln -sf /usr/local/texlive/*/bin/* /usr/local/bin/texlive
RUN tlmgr install \
  collection-fontsrecommended \
  collection-langjapanese \
  collection-latexextra \
  latexindent \
  latexmk


FROM debian:11.1-slim
# add non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME \
  && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
  && apt-get update \
  && apt-get install -y sudo \
  && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
  && chmod 0440 /etc/sudoers.d/$USERNAME

# texlive
COPY --from=installer /usr/local/texlive /usr/local/texlive
ENV PATH /usr/local/bin/texlive:$PATH
COPY ./.latexmkrc /home/$USERNAME

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  perl \
  git \
  # Latexindent dependencies
  # YAML::Tiny:
  libyaml-tiny-perl \
  # File::HomeDir:
  libfile-homedir-perl \
  # Unicode:GCString:
  libunicode-linebreak-perl \
  # Log::Log4perl:
  liblog-log4perl-perl \
  # Log::Dispatch:
  liblog-dispatch-perl \
  && apt-get clean autoclean \
  && apt-get autoremove -y \
  && rm -rf /var/lib/apt/lists/*

RUN ln -sf /usr/local/texlive/*/bin/* /usr/local/bin/texlive
WORKDIR /workspace

# enable git completion
USER $USERNAME
RUN echo "source /usr/share/bash-completion/completions/git" >> ~/.bashrc

CMD ["bash"]
