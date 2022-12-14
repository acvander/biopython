ARG VARIANT="3"
ARG ARCH="aarch64"

# Build dependencies
FROM gcc:12 as genepop
WORKDIR /deps/genepop
RUN wget -O /tmp/genepop.tar.gz http://kimura.univ-montp2.fr/~rousset/GenepopV4.tar.gz \
    && tar -xf /tmp/genepop.tar.gz -C .
RUN tar -xf sources.tar.gz \ 
    && g++ -std=c++11 -o Genepop *.cpp -O3

FROM gcc:12 as bwa
WORKDIR /deps/bwa
RUN git clone https://github.com/lh3/bwa.git
RUN cd bwa; make

FROM gcc:12 as clustalo
ENV CLUSTALO_VERSION="1.2.4"
ARG ARCH
WORKDIR /deps/clustalo
# install argtable
RUN apt update && apt install libargtable2-dev -y
RUN wget -O /tmp/clustalo.tar.gz http://www.clustal.org/omega/clustal-omega-1.2.4.tar.gz
RUN tar -xf /tmp/clustalo.tar.gz -C . \
    && cd clustal-omega-${CLUSTALO_VERSION} \
    && ./configure --build=${ARCH}-unknown-linux-gnu\
    && make \
    && make install


# More Deps
# - clustalw or clustalw2
#- reportlab
#- dialign2-2
#- emboss
#- phylipnew
#- FastTree
# msaprobs
# MAFFT
# MUSCLE
#  NCBI BLAST+
# End of dependencies

FROM mcr.microsoft.com/vscode/devcontainers/python:${VARIANT} as dev

COPY --from=genepop /deps /deps
COPY --from=bwa /deps /deps
COPY --from=clustalo /deps /deps

RUN echo 'export PATH="/deps/bwa/bwa:$PATH"' >> /home/vscode/.bashrc \
    && echo 'export PATH="/deps/clustalo:$PATH"' >> /home/vscode/.bashrc \
    && echo 'export PATH="/deps/genepop:$PATH"' >> /home/vscode/.bashrc

RUN pip install --upgrade pip
RUN pip install pre-commit reportlab

COPY ./ /src/

WORKDIR /src
RUN pip install . \
    && pre-commit install

# Install direnv
RUN apt update \
    && apt install direnv \
    && echo 'eval "$(direnv hook bash)"' >> /home/vscode/.bashrc

# Install act
WORKDIR /act
RUN curl https://raw.githubusercontent.com/nektos/act/master/install.sh | bash \
    && echo 'export PATH="/act/bin/:$PATH"' >> /home/vscode/.bashrc

WORKDIR /out