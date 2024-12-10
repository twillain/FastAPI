# This is my title ! #

# builder stage
FROM python:3.10-slim as builder

# Set the working directory inside the container
WORKDIR /usr/src/app

# Copy only the req to leverage dockers's caching mechasnisms
COPY requirements.txt .

# install dependecncies to a temporary location
RUN pip install --user --no-cache-dir -r requirements.txt

# Copy the wait-for-it.sh script from the scripts folder
COPY scripts/wait-for-it.sh /usr/src/app/

#Runtime stage
FROM python:3.10-slim  

#set working dir in container
WORKDIR /usr/src/app

#copy the app files into the container 
COPY . .

#copy the dependencies 
COPY --from=builder /root/.local /root/.local

# Ensure the script has executable permissions
RUN chmod +x /usr/src/app/wait-for-it.sh

# update path to inculde pip binaries in the user directory
ENV PATH ="/root/.local/bin:$PATH"

# Default command to start the app
CMD ["./wait-for-it.sh", "db:5432", "--", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
