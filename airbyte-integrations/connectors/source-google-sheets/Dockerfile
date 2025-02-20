FROM python:3.9.16-alpine3.18 as base

# build and load all requirements
FROM base as builder
WORKDIR /airbyte/integration_code

# upgrade pip to the latest version
RUN apk --no-cache upgrade \
    && pip install --upgrade pip \
    && apk --no-cache add tzdata build-base


COPY setup.py ./
# install necessary packages to a temporary folder
RUN pip install --prefix=/install .

# build a clean environment
FROM base
WORKDIR /airbyte/integration_code

# copy all loaded and built libraries to a pure basic image
COPY --from=builder /install /usr/local
# add default timezone settings
COPY --from=builder /usr/share/zoneinfo/Etc/UTC /etc/localtime
RUN echo "Etc/UTC" > /etc/timezone

# bash is installed for more convenient debugging.
# Include socat binary in the connector image
RUN apk --no-cache add bash socat && \
    # upgrading openssl due to https://nvd.nist.gov/vuln/detail/CVE-2023-2650
    apk upgrade

# copy payload code only
COPY main.py ./
COPY source_google_sheets ./source_google_sheets

ENV AIRBYTE_ENTRYPOINT "python /airbyte/integration_code/main.py"
ENTRYPOINT ["python", "/airbyte/integration_code/main.py"]

LABEL io.airbyte.version=0.3.9
LABEL io.airbyte.name=airbyte/source-google-sheets
