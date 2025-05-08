# Use the official Python base image
FROM squidfunk/mkdocs-material

# Set the working directory in the container
WORKDIR /app

# Copy the MkDocs project to the working directory
COPY . .

# Build the MkDocs project
RUN mkdocs build --site-dir public

# Use the official Nginx image
FROM nginx:alpine

# Copy the MkDocs project to the Nginx HTML directory
COPY --from=package-installation /app/public /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]

