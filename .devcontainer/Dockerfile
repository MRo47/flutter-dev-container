#-------------------------------------------------------------------------------------------------------------
# Flutter Dev Container - Lucas Hilleshein dos Santos.
# Licensed under the MIT License.
# See https://github.com/lucashilles/flutter-dev-container/blob/master/LICENSE for license information.
#-------------------------------------------------------------------------------------------------------------

FROM ubuntu:jammy AS flutter_dev

RUN export DEBIAN_FRONTEND=noninteractive

# Install basic packages and set up locales
RUN apt update -y && apt-get -y install --no-install-recommends locales gettext \
    && rm -rf /var/lib/apt/lists/*
RUN locale-gen en_GB.UTF-8; update-locale LC_ALL=en_GB.UTF-8 LANG=en_GB.UTF-8
ENV LANG en_GB.UTF-8

# Install apt packages
COPY apt-packages /tmp/apt-packages
RUN apt-get update && \
    apt-get install -y $(cut -d# -f1 </tmp/apt-packages | envsubst) \
    && rm -rf /var/lib/apt/lists/* /tmp/apt-packages

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USER_NAME=vscode
ARG USER_ID
ARG GROUP_NAME
ARG GROUP_ID

RUN set -ex; \
    if getent group $GROUP_ID > /dev/null; then \
    OLD_GROUP_NAME=$(getent group $GROUP_ID |  cut -d: -f1); \
    groupmod --new-name $USER_NAME $OLD_GROUP_NAME; \
    else \
    groupadd --gid $GROUP_ID $USER_NAME; \
    fi \
    && if getent passwd $USER_ID > /dev/null; then \
    OLD_USER_NAME=$(getent passwd $USER_ID |  cut -d: -f1); \
    usermod -l $USER_NAME $OLD_USER_NAME; \
    usermod -d /home/$USER_NAME -m $USER_NAME; \
    else \
    useradd -s /bin/bash --uid $USER_ID --gid $GROUP_ID -m $USER_NAME; \
    fi \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USER_NAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USER_NAME \
    && chmod 0440 /etc/sudoers.d/$USER_NAME

#
# Android SDK
# https://developer.android.com/studio#downloads
ENV ANDROID_SDK_TOOLS_VERSION=11076708
ENV ANDROID_PLATFORM_VERSION=34
ENV ANDROID_BUILD_TOOLS_VERSION=34.0.0
ENV ANDROID_HOME=/home/${USER_NAME}/android-sdk-linux
ENV ANDROID_SDK_ROOT="$ANDROID_HOME"
ENV PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/cmdline-tools/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator

#
# Flutter SDK
# https://flutter.dev/docs/development/tools/sdk/releases?tab=linux
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="3.24.1"
# Make sure to use the needed channel and version for this.
ENV FLUTTER_HOME=/home/${USER_NAME}/flutter
ENV PATH=${PATH}:${FLUTTER_HOME}/bin

USER $USER_NAME
#
# Android SDK	
RUN curl -C - --output /tmp/android-sdk-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_TOOLS_VERSION}_latest.zip \
	&& mkdir -p ${ANDROID_HOME}/ \
	&& unzip -q /tmp/android-sdk-tools.zip -d ${ANDROID_HOME}/cmdline-tools/ \
	&& rm /tmp/android-sdk-tools.zip \
	&& yes | sdkmanager --licenses \
	&& touch /home/${USER_NAME}/.android/repositories.cfg \
	&& sdkmanager platform-tools \
	&& sdkmanager emulator \
	&& sdkmanager "platforms;android-${ANDROID_PLATFORM_VERSION}" "build-tools;$ANDROID_BUILD_TOOLS_VERSION" \
	&& sdkmanager --install "cmdline-tools;latest" 
# create emulator android	
RUN sdkmanager "system-images;android-${ANDROID_PLATFORM_VERSION};google_apis;x86_64" \
	&& echo "no" | avdmanager create avd -n Android${ANDROID_PLATFORM_VERSION} -k "system-images;android-${ANDROID_PLATFORM_VERSION};google_apis;x86_64"

# Flutter SDK
RUN curl -C - --output /tmp/flutter.tar.xz https://storage.googleapis.com/flutter_infra_release/releases/${FLUTTER_CHANNEL}/linux/flutter_linux_${FLUTTER_VERSION}-${FLUTTER_CHANNEL}.tar.xz \
	&& tar -xf /tmp/flutter.tar.xz -C /home/${USER_NAME}/ \
	&& rm /tmp/flutter.tar.xz \
	&& flutter config --android-sdk "${ANDROID_SDK_ROOT}" \
	&& yes | flutter doctor --android-licenses \
	&& flutter config --no-analytics \
	&& flutter update-packages

# zsh install
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.0/zsh-in-docker.sh)" -- \
	-t robbyrussell \
	-p git \
	-p sudo
