ARG TIMEZONE="Europe/Moscow"
ARG VERSION

# -------------------------------------
FROM ubuntu:24.04 AS build
ARG VERSION=

SHELL [ "/bin/bash", "-c" ]
ENV SHELL="/bin/bash"
ENV PATH=$PATH:/app

WORKDIR /app

RUN \
# exit immediately if any command exits with a non-zero status
set -e; \
echo "----- Install packages -----"; \
export DEBIAN_FRONTEND=noninteractive; \
apt-get update; \
apt-get install --assume-yes \
    wget \
    gpg; \
echo "----- DONE -----"


RUN \
# exit immediately if any command exits with a non-zero status
set -e; \
echo "----- Install firefox -----"; \
export DEBIAN_FRONTEND=noninteractive; \
install -d -m 0755 /etc/apt/keyrings; \
wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- \
    | tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null; \
gpg -n -q --import --import-options import-show /etc/apt/keyrings/packages.mozilla.org.asc \
    | awk \
        '/pub/ { \
            getline; \
            gsub(/^ +| +$/,""); \
            if($0 == "35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3") \
                print "\nThe key fingerprint matches ("$0").\n"; \
            else \
                print "\nVerification failed: the fingerprint ("$0") does not match the expected one.\n" \
        }'; \
echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" \
    | tee -a /etc/apt/sources.list.d/mozilla.list > /dev/null; \
printf '%s\n' \
    'Package: *' \
    'Pin: origin packages.mozilla.org' \
    'Pin-Priority: 1000' \
    | tee /etc/apt/preferences.d/mozilla; \
apt-get update; \
apt-get install --assume-yes firefox; \
echo "----- DONE -----"

RUN \
set -e; \
echo "----- Install Docker extensions -----"; \
apt-get install -y python3-docker; \
echo "----- DONE -----"

# Cleanup
RUN \
echo "----- Cleanup -----"; \
apt-get autoremove -y; \
apt-get clean; \
rm -rf /{app,tmp}/* /var/lib/apt/lists/*; \
echo "----- DONE -----"

## STAGE SQUASH
# -------------------------------------
FROM scratch
COPY --from=build / /

WORKDIR /app
ENV PATH=$PATH:/app

COPY *.sh *.py ./
RUN chmod +x *.sh *.py

CMD [ "firefox" ]
ENTRYPOINT [ "./entrypoint.sh" ]
