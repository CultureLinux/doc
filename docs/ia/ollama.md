# 🧠 Ollama 

## 💾 Installation 

Pour installer Ollama, suivez ces étapes :

```bash  
dnf install curl tar gzip unzip
curl -fsSL https://ollama.com/install.sh | sh
systemctl enable --now ollama
systemctl status ollama
firewall-cmd --add-port=11434/tcp --permanent
firewall-cmd --reload
```

## 🚀 Ouvrir les ports 

Pour ouvrir les ports nécessaires à Ollama, modifiez le fichier de service :

```bash
systemctl edit ollama.service
```

Editez la section `[Service]` et ajoutez :

```ini
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Relancez le service pour prendre en compte les modifications :

```bash
systemctl daemon-reload
systemctl restart ollama
```

## 📚 Liste des modèles 

Pour afficher la liste des modèles disponibles, utilisez :

```bash
curl -sS http://localhost:11434/api/models | jq .
ollama list
```

## 😊 Test de modèles 

### Petit et cpu-friendly 🌡️

Testez le modèle "Gemma2" :

```bash
ollama pull gemma2:2b
ollama run gemma2:2b "Salut, résume-moi brièvement ce serveur."
```

### 💾 Petit et cpu-friendly (avec beaucoup de ram) 

Testez le modèle "Mistral" :

```bash
ollama pull mistral:latest
ollama run mistral:latest "Donne un plan d'attaque pour déployer un service web."
```