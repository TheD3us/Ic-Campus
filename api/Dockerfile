FROM python:3.12-alpine

LABEL "org.opencontainers.image.title"="ic-campus-api"
LABEL "org.opencontainers.image.vendor"="IC-TechDojo"
LABEL "org.opencontainers.image.version"="1.0"

RUN apk add --no-cache libcrypto3 curl 


RUN mkdir app && mkdir data 
WORKDIR /app

EXPOSE 5000

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt 

COPY catalog.py .
COPY catalog.json /data/catalog.json
RUN adduser -D appuser

RUN chown -R appuser:appuser /app /data

USER appuser

VOLUME /data

HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD curl -f http://localhost:5000/health || exit 

ENTRYPOINT ["gunicorn","--bind","0.0.0.0:5000","--workers","2","catalog:app"]

