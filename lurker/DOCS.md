# Lurker

## Installation

1. Add https://github.com/snorrea/ha-apps under Settings → Apps → Install app → ⋮ → Repositories.
2. Enable Advanced Mode and add ghcr.io under the App store's private registries, using username snorrea and a classic GitHub token with read:packages.
3. Install Lurker from the new repository, set the boards and notification targets, and start it.
4. Enable Start on boot and Automatic updates.

## Operation

The sidebar opens the ingress dashboard on port 8099. All generated wordclouds and scraper history are under /data/output.
New versions are published here only after private GHCR images have been built and verified. Home Assistant discovers them on repository refresh; no Samba deployment is required.

## Troubleshooting

Check Settings → System → Logs → Supervisor for repository or image-pull errors. Renew the GHCR token before it expires.

## Security

The app enables homeassistant_api to send opt-in mobile notifications. This grants privileged access to the HA Core API inside the container.
Do not put GitHub tokens in app configuration, a Git repository, or chat.
