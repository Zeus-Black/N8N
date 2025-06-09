#  Déploiement de n8n avec Docker Compose

Ce dépôt permet de déployer un serveur **n8n** connecté à une base **PostgreSQL**, avec **persistance des données**, et possibilité d’ajouter un **HTTPS via Nginx**.

## 📁 Contenu du dépôt

- `docker-compose.yml` — Déploiement de n8n et PostgreSQL
- `init-data.sh` — Script d'initialisation de la base de données PostgreSQL
- `.env` — Fichier d'environnement (à créer)
- `README.md` — Documentation

---

## Étapes d'installation

### 1. Cloner le dépôt

```bash
git clone <URL_DU_DEPOT>
cd N8N
```

### 2. Créer et configurer le fichier .env

Créez un fichier **.env** à la racine du projet avec les variables suivantes :

```bash 
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=n8n
POSTGRES_NON_ROOT_USER=n8n_user
POSTGRES_NON_ROOT_PASSWORD=n8n_pass

N8N_HOST=n8n.example.com
N8N_PORT=5678
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.example.com
GENERIC_TIMEZONE=Europe/Paris
```

Remplacez **n8n.example.com** par votre propre nom de domaine.

### 3. Donner les droits d'exécution au script d'init

```bash
chmod +x init-data.sh
```

### 4. Lancer les conteneurs 

```bash
docker-compose up -d
```

### 5. Configuration NGINX

Créez un fichier de configuration n8n dans **/etc/nginx/sites-available/n8n**

```bash
server {
    listen 80;
    server_name n8n.example.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Redémarrez et activez le site : 

````bash
ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
nginx -t && systemctl restart nginx
```

### 6. Obtention certificat https avec Certbot

````bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d n8n.example.com
```

### Vérification

Visitez https://n8n.example.com
