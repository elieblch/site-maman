# ✅ MIGRATION DOMAINE — Historique (Site-Maman)

> ✅ **Migration effectuée le 2026-05-21.** Domaine final en production : **`https://www.relations-psy.fr`**. Ce document devient un historique — toutes les étapes ci-dessous ont été exécutées. Les commandes / URLs `site-maman-sage.vercel.app` restent à titre de trace pour audit, mais elles n'ont plus d'effet (le subdomain Vercel preview n'est plus le canonical).

> Document de référence à exécuter le jour où le site passe de `site-maman-sage.vercel.app` (URL preview) au domaine custom (ex. `relations-psy.fr` ou autre). Ne rien sauter, ne rien faire dans le désordre.

**Hypothèses** :
- Le domaine custom est enregistré et tu en es propriétaire / admin DNS
- Le compte Vercel `elieblch` est lié au repo `elieblch/site-maman`
- Tu as accès à Google Search Console et Bing Webmaster Tools

**Convention dans ce document** :
- `NOUVEAU-DOMAINE` = le vrai domaine final à substituer, **sans `https://` ni `/`** (ex : `relations-psy.fr`)
- Toutes les commandes sont exécutées depuis `/Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman/`

---

## ⏱ Estimation totale : 45-90 min de travail + 24-72 h de monitoring

---

## PHASE 0 — Pré-requis (J-1, la veille)

- [ ] **Backup** : copier tout le dossier `Site-Maman/` ailleurs (zip + cloud)
  ```bash
  cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site
  zip -r "Site-Maman-backup-pre-migration-$(date +%Y%m%d).zip" Site-Maman/ -x "Site-Maman/.git/*"
  ```
- [ ] **Branche git de safety** : créer `pre-migration-backup`
  ```bash
  cd Site-Maman
  git checkout main
  git pull origin main
  git branch pre-migration-backup
  git push origin pre-migration-backup
  ```
- [ ] **Repérer le commit de référence** : `git log -1 --format='%H %s'`
- [ ] **Vérifier la propriété du domaine** chez le registrar (accès DNS)
- [ ] **Préparer le compte Google** pour GSC + GBP
- [ ] **Préparer le compte Microsoft** pour Bing Webmaster Tools (alimente ChatGPT/Copilot)

---

## PHASE 1 — Configuration domaine sur Vercel (5-10 min)

- [ ] Aller sur https://vercel.com/dashboard
- [ ] Sélectionner le projet `site-maman`
- [ ] **Settings → Domains → Add**
- [ ] Saisir `NOUVEAU-DOMAINE` (et éventuellement `www.NOUVEAU-DOMAINE` en alias)
- [ ] Vercel affiche les enregistrements DNS à créer (A record vers `76.76.21.21` ou CNAME vers `cname.vercel-dns.com`)
- [ ] Créer les DNS chez le registrar
- [ ] Attendre la propagation DNS (5-30 min) — Vercel cochera "Valid Configuration"
- [ ] Vercel auto-provisionne le certificat SSL Let's Encrypt (~2 min)
- [ ] **Test** : `curl -I https://NOUVEAU-DOMAINE` → HTTP 200 + cert valide

---

## PHASE 2 — Mise à jour du code (sed, ~5 min)

⚠️ **Faire tous les `sed` dans le même commit pour cohérence**.

- [ ] **Remplacer le domaine partout** :
  ```bash
  cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman
  grep -rln "site-maman-sage.vercel.app" . \
    --include='*.html' --include='*.xml' --include='*.txt' --include='*.md' \
    | xargs sed -i '' 's|site-maman-sage.vercel.app|NOUVEAU-DOMAINE|g'
  ```

- [ ] **Vérifier qu'il ne reste aucune occurrence** :
  ```bash
  grep -rn "site-maman-sage.vercel.app" . --include='*.html' --include='*.xml' --include='*.txt' --include='*.md'
  # Doit ne rien retourner.
  ```

- [ ] **Vérifier les fichiers touchés** :
  ```bash
  git diff --stat
  # Attendu : index.html, mentions-legales.html, politique-confidentialite.html,
  # robots.txt, sitemap.xml, CLAUDE.md, MIGRATION-DOMAINE.md
  ```

---

## PHASE 3 — Désactiver le `noindex` (5 min)

### 3.1 — Retirer la meta robots des 3 HTML

- [ ] **Une commande couvre les 3 fichiers** :
  ```bash
  cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman
  for f in index.html mentions-legales.html politique-confidentialite.html; do
    perl -i -0pe 's|<!--[^>]*PROVISOIRE[^>]*-->\s*\n\s*<meta name="robots" content="noindex, nofollow">\s*\n||gs' "$f"
  done
  ```

- [ ] **Vérifier qu'aucune meta noindex ne traîne sur index.html** :
  ```bash
  grep -n 'noindex' index.html && echo "⚠ noindex encore présent" || echo "✓ propre"
  ```

- [ ] **Option recommandée** : garder `mentions-legales.html` et `politique-confidentialite.html` en `noindex, follow` (pratique courante — pas de valeur SEO mais permet aux crawlers de suivre les liens internes vers la home). Remplacer dans ces 2 fichiers uniquement par :
  ```html
  <meta name="robots" content="noindex, follow">
  ```
  La home, elle, doit absolument être indexable.

### 3.2 — Réécrire le `robots.txt` en version production

- [ ] **Écraser avec la version production** :
  ```bash
  cat > /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman/robots.txt <<'EOF'
  # robots.txt — Cabinet Emmanuelle Francblu Blecher
  # Psychanalyste & Hypno-praticienne · Paris 11e République
  # Production · Last updated: [DATE-DU-JOUR]

  # ─── Crawlers de recherche classiques ───
  User-agent: *
  Allow: /

  # ─── AI crawlers : autorisés (citabilité ChatGPT, Claude, Perplexity, Gemini) ───
  User-agent: GPTBot
  Allow: /

  User-agent: ChatGPT-User
  Allow: /

  User-agent: ClaudeBot
  Allow: /

  User-agent: Claude-Web
  Allow: /

  User-agent: PerplexityBot
  Allow: /

  User-agent: Google-Extended
  Allow: /

  User-agent: Bytespider
  Allow: /

  User-agent: CCBot
  Allow: /

  User-agent: Applebot-Extended
  Allow: /

  # ─── Sitemap ───
  Sitemap: https://NOUVEAU-DOMAINE/sitemap.xml
  EOF
  ```
  Puis remplacer `NOUVEAU-DOMAINE` et `[DATE-DU-JOUR]` :
  ```bash
  sed -i '' "s|NOUVEAU-DOMAINE|$NOUVEAU_DOMAINE|g; s|\\[DATE-DU-JOUR\\]|$(date +%Y-%m-%d)|g" robots.txt
  ```

---

## PHASE 4 — Mise à jour des notes projet (2 min)

- [ ] **CLAUDE.md** : retirer la section "🔴 PRIORITÉ ABSOLUE — LE JOUR DE LA BASCULE DOMAINE" et la section "Domaine canonique (PROVISOIRE)" (le domaine n'est plus provisoire). Mettre à jour la date en bas.
- [ ] **MIGRATION-DOMAINE.md** : ajouter une note en haut → `> ✅ Migration effectuée le [date]. Domaine final : NOUVEAU-DOMAINE. Ce document devient un historique.`

---

## PHASE 5 — Tests locaux avant push (5 min)

- [ ] **Lancer le serveur local** et tester les 5 ressources :
  ```bash
  cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman
  python3 -m http.server 8765 &
  SERVER_PID=$!
  sleep 1
  for u in index.html mentions-legales.html politique-confidentialite.html robots.txt sitemap.xml; do
    echo "$u : $(curl -s -o /dev/null -w 'HTTP %{http_code}' http://localhost:8765/$u)"
  done
  kill $SERVER_PID
  ```

- [ ] **Vérifier que les URLs absolues dans le JSON-LD pointent vers le nouveau domaine** :
  ```bash
  grep -E '(href|content|"url"|"@id"|"image")' index.html | grep -iE 'http' | head -20
  ```

---

## PHASE 6 — Commit + push (3 min)

- [ ] **Stage + commit** :
  ```bash
  cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman
  git add -A
  git status
  git commit -m "Migration vers domaine custom NOUVEAU-DOMAINE

  - Domaine canonique passé de site-maman-sage.vercel.app à NOUVEAU-DOMAINE
    (canonical, og:url, JSON-LD @id/url/image, sitemap)
  - Retrait du noindex sur index.html (page publiquement indexable)
  - robots.txt remis en mode production (Allow: / + 9 AI crawlers explicites)
  - sitemap.xml URLs migrées"
  ```

- [ ] **Push** :
  ```bash
  git -c http.postBuffer=524288000 push origin main
  ```

- [ ] **Attendre 30-60s** que Vercel redéploie.

---

## PHASE 7 — Vérifications post-déploiement (10-15 min)

### 7.1 — Tests HTTP

- [ ] **HTTP 200 sur les 5 URLs publiques** :
  ```bash
  for u in / /mentions-legales.html /politique-confidentialite.html /robots.txt /sitemap.xml; do
    echo "$u : $(curl -s -o /dev/null -w 'HTTP %{http_code} · %{time_total}s' -L https://NOUVEAU-DOMAINE$u)"
  done
  ```

- [ ] **HTTPS forcé** : `curl -I http://NOUVEAU-DOMAINE/` doit retourner un 301 vers https.

- [ ] **CLAUDE.md et MIGRATION-DOMAINE.md retournent 404** (`.vercelignore` actif) :
  ```bash
  for f in CLAUDE.md MIGRATION-DOMAINE.md; do
    echo "$f : $(curl -s -o /dev/null -w 'HTTP %{http_code}' https://NOUVEAU-DOMAINE/$f)"
  done
  ```

### 7.2 — Google Rich Results Test

- [ ] Aller sur https://search.google.com/test/rich-results
- [ ] Onglet **"URL"** → saisir `https://NOUVEAU-DOMAINE/`
- [ ] **Vérifier que ces 3 entités sont détectées sans erreur** :
  - [ ] `Person` (Emmanuelle Francblu Blecher, jobTitle, alumniOf, knowsAbout)
  - [ ] `ProfessionalService` (cabinet — adresse, téléphone, services)
  - [ ] `WebSite`
- [ ] **0 erreur bloquante** ; warnings facultatifs OK

### 7.3 — Lighthouse mobile post-migration

- [ ] **Lancer Lighthouse 3 fois** :
  ```bash
  for i in 1 2 3; do
    lighthouse https://NOUVEAU-DOMAINE/ \
      --output=json --output-path=/tmp/lh-postmig-$i.json \
      --only-categories=performance,seo,accessibility,best-practices \
      --form-factor=mobile --screenEmulation.disabled \
      --chrome-flags="--headless --no-sandbox" --quiet
  done
  ```
- [ ] **Cibles attendues** (médianes) :
  - Performance ≥ 90
  - **SEO = 100** (le noindex a été retiré, donc remonte à 100)
  - Best Practices ≥ 95
  - Accessibility ≥ 95
  - **LCP < 2,5 s** ✓
  - CLS < 0,1
  - TBT < 200 ms

### 7.4 — Open Graph preview

- [ ] Tester https://www.opengraph.xyz/url/https%3A%2F%2FNOUVEAU-DOMAINE/
- [ ] Facebook debug : https://developers.facebook.com/tools/debug/?q=https://NOUVEAU-DOMAINE/ + "Scrape Again"
- [ ] LinkedIn : https://www.linkedin.com/post-inspector/inspect/https:%2F%2FNOUVEAU-DOMAINE
- [ ] **Vérifier** que l'image `og-image.jpg` 1200×630 s'affiche, titre + description OK

---

## PHASE 8 — Référencement Google + Bing (15-20 min)

### 8.1 — Google Search Console

- [ ] https://search.google.com/search-console/ → **Add property** → "URL prefix" → `https://NOUVEAU-DOMAINE/`
- [ ] Vérification : récupérer la balise meta et l'ajouter dans le `<head>` de `index.html` :
  ```html
  <meta name="google-site-verification" content="VALEUR-DONNEE-PAR-GSC">
  ```
- [ ] Push de la balise (`git add index.html && git commit -m "feat: GSC verification" && git push`)
- [ ] Attendre le redéploiement, puis cliquer "Verify"
- [ ] **Sitemaps → Add a new sitemap** → `sitemap.xml` → Submit
- [ ] **Inspection URL** → coller `https://NOUVEAU-DOMAINE/` → "Test live URL" → "Request indexing"

### 8.2 — Bing Webmaster Tools

- [ ] https://www.bing.com/webmasters → Add a site → `https://NOUVEAU-DOMAINE/`
- [ ] Import direct depuis GSC possible (1 clic)
- [ ] Soumettre `sitemap.xml`
- [ ] Bing alimente ChatGPT, Copilot, Alexa → critique pour AI visibility

### 8.3 — Google Business Profile

- [ ] https://business.google.com → **claim "Cabinet Emmanuelle Francblu Blecher"** (17 bd Jules Ferry, Paris 11)
- [ ] Catégorie primaire : **"Psychanalyste"**
- [ ] Catégorie secondaire : "Hypnothérapeute"
- [ ] Horaires : "Sur rendez-vous" (champ libre — pas d'horaires fixes)
- [ ] Téléphone : +33 6 16 98 46 78
- [ ] Site web : `https://NOUVEAU-DOMAINE/`
- [ ] Description : reprendre la meta description du site
- [ ] Photos : intérieur cabinet, extérieur, portrait
- [ ] **Récupérer le `place_id` GBP** une fois la fiche validée et l'utiliser pour remplacer l'iframe Google Maps dans `index.html` (section #localisation) — la fiche GBP avec ses avis s'affichera dans la carte.

### 8.4 — Apple Business Connect

- [ ] https://businessconnect.apple.com → claim cabinet
- [ ] NAP + URL site
- [ ] (Atteint Apple Maps + Siri)

### 8.5 — Annuaires professionnels santé mentale (optionnel mais utile)

- [ ] Doctolib (si prise de RDV en ligne souhaitée)
- [ ] Psychologue.net / PsyEnLigne (si applicable)
- [ ] Pages Jaunes (annuaire local FR)

---

## PHASE 9 — Monitoring J+1 à J+7 (passif)

### J+1

- [ ] GSC → **Pages → Why pages aren't indexed** : aucune page en "Excluded by noindex"
- [ ] GSC → **Page Indexing report** : la home doit apparaître en "Indexed" (peut prendre 24-72h)

### J+2 à J+3

- [ ] Google : `site:NOUVEAU-DOMAINE` → doit retourner au moins la home
- [ ] Test : `psychanalyste paris 11 république` ou `Emmanuelle Francblu Blecher` → vérifier la présence dans la SERP (7-14 jours)
- [ ] Vérifier qu'aucun duplicate content avec `site-maman-sage.vercel.app` : si Vercel Settings → Domains liste encore l'ancien, **ajouter une redirection 301** dans `vercel.json` :
  ```json
  {
    "redirects": [
      {
        "source": "/(.*)",
        "destination": "https://NOUVEAU-DOMAINE/$1",
        "permanent": true,
        "has": [{ "type": "host", "value": "site-maman-sage.vercel.app" }]
      }
    ]
  }
  ```

### J+7

- [ ] GSC → **Performance** : premières impressions
- [ ] CrUX (si plus de 1k visites) : Core Web Vitals terrain via https://pagespeed.web.dev/?url=https://NOUVEAU-DOMAINE/

---

## 🆘 PLAN DE ROLLBACK — si quelque chose casse

### Symptôme A : Site répond 404/500

1. Vercel → Deployments — dernier build OK ?
2. Si oui : problème DNS → vérifier les enregistrements
3. Si non : `git revert HEAD && git push` pour rollback

### Symptôme B : Google indexe encore `site-maman-sage.vercel.app` (duplicate content)

1. Ajouter la redirection 301 dans `vercel.json` (voir J+3)
2. Dans GSC ancien projet : Settings → "Remove URL prefix property"
3. Attendre 2-4 semaines pour migration des URLs indexées

### Symptôme C : LCP > 2,5 s

1. Vérifier HTTP/2 + Brotli : `curl -I -H "Accept-Encoding: br" https://NOUVEAU-DOMAINE/Bureau1-1200.webp` → `content-encoding: br`
2. Vérifier que `<link rel="preload" as="image">` est toujours dans le `<head>`
3. Tester depuis pagespeed.web.dev avec un autre point géo

### Symptôme D : Rollback complet vers la version pre-migration

```bash
cd /Users/user/Desktop/Cours/Travail_Perso/Creation_Site/Site-Maman
git checkout pre-migration-backup
git push origin pre-migration-backup:main --force-with-lease
```

⚠️ `--force-with-lease` écrase `main` mais préserve les commits intermédiaires si d'autres ont pushé. À utiliser **seulement** si certain qu'aucun autre changement n'a été poussé depuis.

Après rollback : Vercel → Settings → Domains → retirer le nouveau domaine.

---

## 📋 Récapitulatif des fichiers modifiés ce jour-là

| Fichier | Changement |
|---|---|
| `index.html` | `site-maman-sage.vercel.app` → nouveau domaine (canonical + OG + JSON-LD), retrait noindex |
| `mentions-legales.html` | Domaine + (optionnel) `noindex, follow` au lieu de `noindex, nofollow` |
| `politique-confidentialite.html` | Idem |
| `robots.txt` | Réécrit en version production (Allow: / + AI crawlers) |
| `sitemap.xml` | URLs migrées |
| `CLAUDE.md` | Sections obsolètes retirées, date mise à jour |
| `MIGRATION-DOMAINE.md` (ce fichier) | Note d'exécution en haut |
| (nouveau) `vercel.json` | Redirection 301 depuis site-maman-sage.vercel.app (si applicable) |

---

*Document créé le 2026-05-20. À utiliser le jour J et ensuite archiver (mais conserver dans le repo pour traçabilité).*
