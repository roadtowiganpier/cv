# Christian Baker — Personal Blog & Landing Page

This is a simple landing page designed to integrate with my blog which is hosted on a remote server (VPS) using docker containers. 


## Project Structure

```
.
├── docker-compose.yml        # All services
├── .env.example              # Environment variable template
├── .gitignore
├── nginx/
│   └── default.conf          # Reverse proxy — landing page + Ghost
├── landing/
│   ├── index.html            # Custom landing page
│   └── style.css             # Styling
└── ghost/                    # Ghost config overrides (if needed)
```


