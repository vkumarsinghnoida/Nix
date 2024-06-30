FROM ubuntu:24.04

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Install tmux and nix
RUN apt-get update && \
    apt-get install -y tmux sudo && \
    rm -rf /var/lib/apt/lists/*

RUN curl -L https://nixos.org/nix/install | sh -s -- --daemon

RUN useradd -m -s /bin/bash ubuntu && echo 'ubuntu:123' | chpasswd && usermod -aG sudo ubuntu
RUN mkdir -p ~/.config/nix && echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf

# Directly add tmux commands to.bashrc with each command on a separate line
RUN echo 'tmux new-session -s 11' >> ~/.bashrc && \
    echo 'tmux send-keys -t 11 "nix-daemon" C-m' >> ~/.bashrc && \
    echo 'tmux detach' >> ~/.bashrc

WORKDIR /home/ubuntu/

USER ubuntu
