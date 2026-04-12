# ☁️ Mon Cloud Personnel — Nextcloud

Stack : **Nextcloud + MariaDB + Nginx + Let's Encrypt + Duck DNS**

---

## 📂 Structure du projet

```
nextcloud/
├── docker-compose.yml
├── nginx.conf
├── .env               ← à configurer en premier !
└── README.md
```

---

## 🚀 Installation étape par étape

### 1. Trouver le chemin du disque externe

Branche ton disque externe, puis :
```bash
lsblk
```
Note le chemin de montage (ex: `/media/tonuser/MonDisque`) et mets-le dans `.env`.

---

### 2. Configurer le .env

Ouvre `.env` et remplis :
- `EXTERNAL_PATH` → chemin de ton disque externe
- `DUCKDNS_SUBDOMAIN` → le sous-domaine choisi sur duckdns.org
- `DUCKDNS_TOKEN` → le token affiché sur duckdns.org
- Les mots de passe (évite `@ # $ %` dans les mdp)

---

### 3. Ouvrir les ports sur ta box

Dans l'interface de ta box internet :
- Port **80** (TCP) → IP de ton serveur
- Port **443** (TCP) → IP de ton serveur

Pour trouver l'IP locale de ton serveur :
```bash
hostname -I
```

---

### 4. Créer les dossiers sur le disque externe

```bash
mkdir -p /media/tonuser/MonDisque/nextcloud/{data,db,certbot/{conf,www}}
```
(remplace le chemin par le tien)

---

### 5. Obtenir le certificat HTTPS (première fois)

```bash
# Lance d'abord nginx en mode HTTP seulement (commente le bloc 443 dans nginx.conf)
docker compose up -d nginx certbot duckdns

# Attends 1-2 minutes que Duck DNS propage ton IP, puis :
docker compose run --rm certbot certonly \
  --webroot \
  -w /var/www/certbot \
  -d moncloud.duckdns.org \
  --email ton@email.com \
  --agree-tos \
  --no-eff-email

# Décommente ensuite le bloc 443 dans nginx.conf
```

---

### 6. Lancer toute la stack

```bash
docker compose up -d
```

Attends ~1 minute, puis accède à :
```
https://moncloud.duckdns.org
```

---

### 7. Commandes utiles

```bash
# Voir les logs
docker compose logs -f

# Arrêter
docker compose down

# Mettre à jour les images
docker compose pull && docker compose up -d

# Vérifier le renouvellement du certificat
docker compose logs certbot
```

---

## 🔒 Sécurité

- Change tous les mots de passe du `.env` avant de lancer
- Ne partage jamais ton `.env`
- Le certificat SSL se renouvelle automatiquement toutes les 12h (si nécessaire)

---

## 🆘 Problèmes fréquents

| Problème | Solution |
|---|---|
| Page inaccessible | Vérifie l'ouverture des ports sur ta box |
| Certificat échoue | Attends 2-3 min que Duck DNS se propage, réessaie |
| Erreur base de données | `docker compose logs db` pour voir le détail |
| Disque non monté | `lsblk` pour vérifier, `sudo mount` si nécessaire |
