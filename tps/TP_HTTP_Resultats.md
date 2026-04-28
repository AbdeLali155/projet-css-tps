# TP HTTP - Resultats
 

Nom= TAHIRI Abdelali
---

## TP 1 : DevTools

### 1.2 - requete GET sur httpbin.org/get

j'ai ouvert chrome, F12, onglet Network, coché "Preserve log" et navigué vers l'url

- **Code de statut :** 200 OK
- **Headers de requête envoyés :**
  - `Host: httpbin.org`
  - `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)`
  - `Accept: text/html,application/xhtml+xml`
  - `Accept-Encoding: gzip, deflate, br`
  - `Connection: keep-alive`
- **Content-Type de la réponse :** `application/json`

### 1.4 - codes de statut

j'ai testé ces URLs directement dans le navigateur :

| URL | Méthode | Code | Content-Type |
|-----|---------|------|--------------|
| httpbin.org/get | GET | 200 | application/json |
| httpbin.org/post | POST | 200 | application/json |
| httpbin.org/status/201 | GET | 201 | text/html; charset=utf-8 |
| httpbin.org/status/404 | GET | 404 | text/html |
| httpbin.org/status/500 | GET | 500 | text/html |
| httpbin.org/redirect/3 | GET | 302 → 200 | application/json |

---

## TP 2 : cURL

> j'ai utilisé Git Bash sur Windows pour toutes les commandes

### 2.1 - difference entre -i et -v

- `-i` : affiche les headers de réponse + le body (simple et rapide)
- `-v` : mode verbose, montre tout — la connexion TCP, les headers de requete ET de réponse, TLS... utile pour debugger

### Exercice avancé

commande cURL qui fait tout ce qui était demandé :

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-Custom-Header: MonHeader" \
  -d '{"action": "test", "value": 42}' \
  -i \
  https://httpbin.org/post
```

---

## TP 3 : Fetch JavaScript

### Exercice - fetchWithRetry

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      // si c'est une erreur 5xx on reessaie
      if (response.status >= 500) {
        lastError = new Error(`HTTP ${response.status}`);
        console.warn(`tentative ${attempt}/${maxRetries} echouée, on attend 1s...`);
        if (attempt < maxRetries) {
          await new Promise(res => setTimeout(res, 1000));
          continue;
        }
      }

      return response; // ok on retourne la reponse

    } catch (err) {
      lastError = err;
      console.warn(`erreur reseau tentative ${attempt}:`, err.message);
      if (attempt < maxRetries) {
        await new Promise(res => setTimeout(res, 1000));
      }
    }
  }

  throw lastError; // plus de tentatives, on leve l'erreur
}

// test
fetchWithRetry('https://jsonplaceholder.typicode.com/posts/1', {}, 3)
  .then(r => r.json())
  .then(data => console.log(data))
  .catch(err => console.error('echec:', err.message));
```

---

## TP 4 : Headers de Sécurité

j'ai utilisé le site [securityheaders.com](https://securityheaders.com) pour analyser les 3 sites

| Site | HSTS | X-Frame-Options | CSP | Note |
|------|------|-----------------|-----|------|
| github.com | ✅ max-age=31536000; includeSubDomains | ✅ deny | ✅ present | A+ |
| google.com | ✅ max-age=31536000 | ✅ SAMEORIGIN | ⚠️ partiel | A |
| wikipedia.org | ✅ max-age=106384710 | ✅ DENY | ✅ present | A |

---

## TP 5 : Cache HTTP

### 5.2 - requete conditionnelle avec ETag

```bash
# etape 1 : on recupere l'ETag
curl -i https://httpbin.org/etag/test123
# reponse : ETag: "test123"

# etape 2 : on renvoie avec If-None-Match
curl -i -H "If-None-Match: \"test123\"" https://httpbin.org/etag/test123
# reponse : 304 Not Modified  (pas de body = gain de bande passante)
```

### 5.3 - cache dans le navigateur

- `F5` → le navigateur utilise le cache si encore valide (on voit "from cache" dans DevTools)
- `Ctrl+Shift+R` → force le rechargement complet, ignore le cache

---

## Exercices Récapitulatifs

### Exercice 1 - Client HTTP

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Client HTTP</title>
  <style>
    body { font-family: monospace; max-width: 750px; margin: 2rem auto; padding: 1rem; }
    input, select, textarea { width: 100%; margin: 4px 0 12px; padding: 6px; box-sizing: border-box; }
    pre { background: #f0f0f0; padding: 1rem; border-radius: 4px; overflow-x: auto; }
    button { padding: 8px 20px; background: #0070f3; color: #fff; border: none; border-radius: 4px; cursor: pointer; }
  </style>
</head>
<body>
  <h2>Client HTTP</h2>

  <label>URL</label>
  <input id="url" type="text" value="https://jsonplaceholder.typicode.com/posts/1" />

  <label>Méthode</label>
  <select id="method">
    <option>GET</option><option>POST</option><option>PUT</option><option>DELETE</option>
  </select>

  <label>Body (JSON)</label>
  <textarea id="body" rows="3" placeholder='{"key": "value"}'></textarea>

  <button onclick="sendRequest()">Envoyer</button>

  <h3>Statut</h3>
  <pre id="status">-</pre>
  <h3>Headers</h3>
  <pre id="headers">-</pre>
  <h3>Body</h3>
  <pre id="result">-</pre>

  <script>
    async function sendRequest() {
      const url = document.getElementById('url').value;
      const method = document.getElementById('method').value;
      const bodyText = document.getElementById('body').value;

      const options = { method, headers: { 'Content-Type': 'application/json' } };
      if (method !== 'GET' && bodyText) options.body = bodyText;

      try {
        const res = await fetch(url, options);
        document.getElementById('status').textContent = `${res.status} ${res.statusText}`;

        const h = {};
        res.headers.forEach((v, k) => h[k] = v);
        document.getElementById('headers').textContent = JSON.stringify(h, null, 2);

        const data = await res.json();
        document.getElementById('result').textContent = JSON.stringify(data, null, 2);
      } catch (err) {
        document.getElementById('status').textContent = 'Erreur : ' + err.message;
      }
    }
  </script>
</body>
</html>
```

---

### Exercice 2 - Questions théoriques

**1. difference entre no-cache et no-store ?**

- `no-cache` : le navigateur stocke la reponse mais doit verifier avec le serveur avant de la reutiliser (via ETag)
- `no-store` : rien n'est stocké du tout, ni en cache ni ailleurs — utilisé pour les données sensibles

**2. Pourquoi POST n'est pas idempotent ?**

parce que chaque requete POST crée une nouvelle ressource. si on envoie 2 fois la meme requete POST on aura 2 entrees différentes. contrairement à PUT ou DELETE qui donnent le meme résultat peu importe combien de fois on les appelle.

**3. Que se passe-t-il avec un code 301 ?**

301 = redirection permanente. le navigateur suit automatiquement vers la nouvelle URL (dans le header `Location`) et met a jour son cache pour les prochaines fois.

**4. À quoi sert le header Origin ?**

il indique d'où vient la requete (protocole + domaine + port). le navigateur l'envoie automatiquement pour les requetes cross-origin. le serveur l'utilise pour les verifications CORS avec `Access-Control-Allow-Origin`.

**5. Pourquoi HttpOnly sur les cookies de session ?**

pour empecher javascript d'acceder au cookie via `document.cookie`. ca protege contre les attaques XSS — meme si quelqu'un injecte du JS malveillant il ne pourra pas voler le cookie de session.