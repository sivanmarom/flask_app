FROM python:3.8
WORKDIR flask_app/app/
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY /app .
CMD ["python3", "app.py", "sivan"]
