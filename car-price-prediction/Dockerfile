# ---- Car Price Prediction: production image ----
FROM python:3.12-slim

# Prevent .pyc files & enable unbuffered logging (good for Docker/Render logs)
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install OS deps needed to build/run scikit-learn & friends
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python deps first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the app
COPY . .

# Render (and most hosts) inject PORT at runtime; default to 5000 for local docker run
ENV PORT=5000
EXPOSE 5000

# Basic healthcheck hitting the home page
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import os,urllib.request;urllib.request.urlopen('http://127.0.0.1:'+os.environ.get('PORT','5000'))" || exit 1

# Use gunicorn in production; shell form so $PORT expands at container start
CMD gunicorn app:app --bind 0.0.0.0:$PORT --workers 2 --threads 4 --timeout 60
