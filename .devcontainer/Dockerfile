FROM fedora:latest

# Update and install packages
RUN dnf -y makecache && \
    dnf -y install \
        axel \
        zsh \
        git \
        wget \
        ripgrep \
        unzip \
        openssh-server \
        neovim \
        rust-analyzer \
        clang \
        gcc \
        sudo \
        java-latest-openjdk-devel && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
    useradd -m -s /bin/zsh dev && \
    echo 'dev ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers && \  # Create user 'dev' with home dir and zsh as default shell

## Android-ndk stuff
RUN mkdir -p /opt/ && \
wget -P /opt/ "https://dl.google.com/android/repository/android-ndk-r27c-linux.zip" && \
unzip /opt/android-ndk-r27c-linux.zip -d /opt/ && \
rm -rf /opt/android-ndk-r27c-linux.zip

# Environment variables for Android SDK
ENV ANDROID_SDK_ROOT="/opt/android-sdk"
ENV PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$PATH"

# Install Android SDK command-line tools
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools \
    && cd ${ANDROID_SDK_ROOT}/cmdline-tools \
    && wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O cmdline-tools.zip \
    && unzip cmdline-tools.zip -d ./ \
    && mv ./cmdline-tools ./latest \
    && rm cmdline-tools.zip

# Install SDK components
RUN yes | sdkmanager --sdk_root=${ANDROID_SDK_ROOT} --licenses
RUN sdkmanager --sdk_root=${ANDROID_SDK_ROOT} \
    "platform-tools" \
    "platforms;android-34" \
    "build-tools;34.0.0" \
    "ndk;25.2.9519653" \
    "cmake;3.22.1"

# Install Gradle
RUN wget https://services.gradle.org/distributions/gradle-8.5-bin.zip -P /opt/ \
    && unzip /opt/gradle-8.5-bin.zip -d /opt/ \
    && ln -s /opt/gradle-8.5/bin/gradle /usr/local/bin/gradle \
    && rm /opt/gradle-8.5-bin.zip

# Switch to the new user
USER dev
WORKDIR /home/dev

# Install Rust non-interactively
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

# Add Cargo bin to PATH (for future RUN instructions)
ENV PATH="/home/dev/.cargo/bin:${PATH}"

# Set default shell
SHELL ["/bin/zsh", "-c"]

## Add ohmyzsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

## omz extensions, 1) fzf-tab
RUN git clone https://github.com/Aloxaf/fzf-tab ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/fzf-tab && \ ## Use custom zsh file
curl "https://raw.githubusercontent.com/FG-rps/test_repo/refs/heads/main/zshrc" > /home/dev/.zshrc

## setup android cross-compilation
RUN rustup target add aarch64-linux-android \
                    armv7-linux-androideabi \
                    arm-linux-androideabi \
                    i686-linux-android

## Nvim stuff
RUN mkdir -p ~/.local/share/fonts && curl "https://raw.githubusercontent.com/FG-rps/test_repo/refs/heads/main/CaskaydiaCoveNerdFont-Regular.ttf" > ~/.local/share/fonts/CaskaydiaCoveNerdFont-Regular.ttf && git clone https://github.com/LazyVim/starter ~/.config/nvim && rm ~/.config/nvim/init.lua && curl "https://raw.githubusercontent.com/FG-rps/test_repo/refs/heads/main/init.lua" > ~/.config/nvim/init.lua
