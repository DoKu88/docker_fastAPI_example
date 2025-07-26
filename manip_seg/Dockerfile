FROM python:3.10-slim
WORKDIR /app
COPY server.py .
RUN pip install fastapi uvicorn numpy
CMD ["python", "server.py"]
