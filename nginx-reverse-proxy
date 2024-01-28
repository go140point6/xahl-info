# Example of nginx reverse proxy getting rpc and wss working.
# xah-mainnet01.customdomain.com = A (host) Record in DNS pointing to IP address.
# rpc-xah-mainnet.customdomain.com & wss-xah-mainnet.customdomain.com = Two CNAME records pointing to the A record.

# When you configure Let's Encrypt, be sure to request all three FQDN in the same cert request.

server {
    listen 80;
    server_name xah-mainnet01.customdomain.com rpc-xah-mainnet.customdomain.com wss-xah-mainnet.customdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rpc-xah-mainnet.customdomain.com;

    # SSL certificate paths
    ssl_certificate /etc/letsencrypt/live/xah-mainnet01.customdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xah-mainnet01.customdomain.com/privkey.pem;

    # Other SSL settings
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    # Additional SSL settings, including HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    location / {
        try_files  / =404;
        allow 123.45.67.89;     # evr-node01
        allow 98.76.54.32;      # evr-node02
        allow 111.111.111.111;  # evr-node03
        allow 88.88.88.88;      # vps-development
        allow 55.44.33.222;     # Allow the source IP of the node itself (for validation testing)
        deny all;
        proxy_pass http://localhost:6007;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Additional server configurations

    # Set Content Security Policy (CSP) header
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline';";

    # Enable XSS protection
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";

}

server {
    listen 443 ssl http2;
    server_name wss-xah-mainnet.customdomain.com;

    # SSL certificate paths
    ssl_certificate /etc/letsencrypt/live/xah-mainnet01.customdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xah-mainnet01.customdomian.com/privkey.pem;

    # Other SSL settings
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    # Additional SSL settings, including HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    location / {
        try_files  / =404;
        allow 123.45.67.89;     # evr-node01
        allow 98.76.54.32;      # evr-node02
        allow 111.111.111.111;  # evr-node03
        allow 55.66.77.88;      # vps-dev-not-in-rpc-list
        allow 88.88.88.88;      # vps-development
        allow 55.44.33.222;     # Allow the source IP of the node itself (for validation testing)
        deny all;
        proxy_pass http://localhost:6008;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # These first three are critical to getting node websockets to work
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache off;
        proxy_buffering off;
    }

    # Additional server configurations

    # Set Content Security Policy (CSP) header
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline';";

    # Enable XSS protection
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";

}
