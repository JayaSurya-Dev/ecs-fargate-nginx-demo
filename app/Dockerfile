FROM python:3.11-slim
WORKDIR /app
COPY index.html style.css /app/
EXPOSE 8080
CMD ["python", "-m", "http.server", "8080"]
