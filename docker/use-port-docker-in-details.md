When you configure your Docker service with the following line:

```yaml
services:
  # Nginx as Web Server
  nginx:
    build: ./nginx/
    container_name: nginx
    ports:
      - "80:8081"
```

```
// default.conf sample
server {
    listen 8081 default_server;
    server_name test.gov.bd;
    root /var/www/html/public;  # Ensure this points to the Nginx container's path
    index index.php index.html;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    -----
}
```

you're essentially telling Docker to map the host's port `80` to the container's port `8081`. Here's a breakdown of how this works:

### Ports Mapping

- **Host Port (80)**: This is the port on your local machine (host) that you will use to access the service from your browser or other clients. When you enter `http://test.gov.bd` in your browser, it tries to connect to port `80` on your machine.

- **Container Port (8081)**: This is the port on which your Nginx server is listening inside the Docker container. In your case, the Nginx server is configured to listen on port `8081` as per your Nginx configuration file.

### Explanation of the Mapping

1. **Request Flow**:

   - When you enter `http://test.gov.bd` in your browser, the request goes to port `80` on your host machine.
   - Docker intercepts this request and forwards it to the Nginx container's port `8081`.

2. **Container Response**:
   - The Nginx server running inside the container processes the request on port `8081` and sends the response back through the same mapping, which ultimately reaches your browser.

### Why Use This Mapping?

- **Standard Port**: Port `80` is the default port for HTTP traffic. By mapping it, you allow users to access your application without specifying a port number in the URL (e.g., `http://test.gov.bd` instead of `http://test.gov.bd:8081`).

- **Container Isolation**: By using a different internal port (like `8081`), you can run multiple services in different containers without port conflicts. For example, you could have another service in another container that also listens on port `8081`, but it could be mapped to a different port on the host.

### Summary

The line `ports: - "80:8081"` effectively makes your Nginx web server accessible from the standard HTTP port `80` on your host machine, while the Nginx service inside the container listens on port `8081`. This allows you to maintain a clean and standard URL for users accessing your application.
