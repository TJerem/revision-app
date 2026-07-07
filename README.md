# Mon projet revision-app
![CI](https://github.com/TJerem/revision-app/actions/workflows/ci.yml/badge.sv
g)
> API REST Flask avec pipeline CI/CD, déployée sur Cloud Run.
## Installation
```bash
git clone https://github.com/TJerem/revision-app.git
cd revision-app
pip install -r requirements.txt
python src/app.py
```
## Routes API
| Route | Méthode | Description |
|-------|---------|-------------|
| / | GET | Accueil |
| /health | GET | Health check |
| /version | GET | Version |
