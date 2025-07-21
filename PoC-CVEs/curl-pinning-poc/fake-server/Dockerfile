FROM python:3.10-slim

# Cài công cụ build, git, OpenSSL
RUN apt update && apt install -y gcc build-essential git libssl-dev

# Clone aioquic
WORKDIR /opt
RUN git clone https://github.com/aiortc/aioquic.git

# Cài aioquic và wsproto
WORKDIR /opt/aioquic
RUN pip install . && pip install wsproto starlette jinja2

ENV PYTHONPATH="/opt/aioquic"

# Quay về app
WORKDIR /app

# Chạy server trực tiếp qua file
CMD ["python3", "/opt/aioquic/examples/http3_server.py", \
     "--certificate", "/app/fake.crt", \
     "--private-key", "/app/fake.key", \
     "--host", "0.0.0.0", \
     "--port", "4433"]





