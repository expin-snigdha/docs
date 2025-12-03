# Infrastructure Documentation

This repository contains the infrastructure and DevOps documentation built with MkDocs Material.

## Prerequisites

- Python 3.8 or higher
- pip

## Setup

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Development

### Local Preview

To preview the documentation locally:

```bash
mkdocs serve
```

Then open your browser to `http://127.0.0.1:8000/`

### Building the Site

To build the static site:

```bash
mkdocs build
```

The generated site will be in the `site/` directory.

## Project Structure

```
.
├── content/                        # Documentation source files
│   ├── index.md                    # Homepage
│   ├── compute-cluster/            # Compute cluster documentation
│   │   ├── index.md
│   │   ├── network.md
│   │   ├── rack-configurations.md
│   │   ├── nodes.md
│   │   └── cluster-management.md
│   ├── usage-model/                # Usage model documentation
│   │   ├── index.md
│   │   ├── remote-access/          # Remote access methods
│   │   │   ├── vpn.md
│   │   │   ├── ssh.md
│   │   │   ├── desktop-environment/
│   │   │   │   ├── vnc.md
│   │   │   │   └── rdp.md
│   │   │   └── vscode-remote.md
│   │   ├── shell.md
│   │   ├── tools.md
│   │   ├── git-setup.md
│   │   ├── development.md
│   │   └── cluster.md
│   ├── devops-flows/               # DevOps workflows
│   │   ├── index.md
│   │   ├── development-flows.md
│   │   ├── integration-flows.md
│   │   ├── deployment-flows.md
│   │   └── end-user-testing-flows.md
│   └── stylesheets/
│       └── extra.css               # Custom CSS
├── mkdocs.yml                      # MkDocs configuration
└── requirements.txt                # Python dependencies
```

## Contributing

1. Make your changes in the `content/` directory
2. Preview locally with `mkdocs serve`
3. Commit and push your changes

## Deployment

To deploy to GitHub Pages:

```bash
mkdocs gh-deploy
```

## License

[Add your license information here]
