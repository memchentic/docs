# Build the Sphinx site, then serve static HTML with nginx.
FROM python:3.12-slim AS build

WORKDIR /opt/src

COPY sphinx/requirements.txt /tmp/docs-requirements.txt
RUN pip install --no-cache-dir -r /tmp/docs-requirements.txt \
    && rm -f /tmp/docs-requirements.txt

COPY sphinx/ /opt/src/sphinx/
RUN python -m sphinx \
        -c /opt/src/sphinx/source \
        -b html \
        /opt/src/sphinx/source \
        /tmp/docs-html \
    && test -f /tmp/docs-html/index.html

FROM nginx:alpine

COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /tmp/docs-html /usr/share/nginx/html

EXPOSE 80
