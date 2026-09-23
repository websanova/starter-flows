# Deploy Guide

To deploy the flows publicly, serve the `public` directory.

```nginx
server {
    root /path/to/app/dir/public;
    index index.html;

    add_header Cache-Control "no-store" always;

    location ~ \.md$ {
        default_type text/markdown;
    }
}
```

The viewer detects a non-localhost host and loads once instead of polling, so nothing else needs configuring.
