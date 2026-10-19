# Deploy Guide

To deploy the flows publicly, serve the `public` directory.

```nginx
server {
    location / {
        try_files $uri /index.html;
    }
}
```

And if you want to add basic auth as well:

```nginx
server {
    auth_basic           "Flows;
    auth_basic_user_file /path/to/public/.htpasswd;
}
```
