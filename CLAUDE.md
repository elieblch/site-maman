# Site-Maman — Instructions projet pour Claude Code

> **Lecture obligatoire à chaque session.** Site one-page d'**Emmanuelle Francblu Blecher**, psychanalyste & hypno-praticienne à Paris 11e (République). Ce fichier référence le plugin SEO externe `claude-seo-main` qui est une **dépendance permanente** du projet.

---

## 1. Dépendance externe : `claude-seo-main`

Chemin absolu :
`/Users/user/Desktop/Cours/Travail_Perso/Creation_Site/claude-seo-main/`

C'est le plugin open-source [AgriciDaniel/claude-seo](https://github.com/AgriciDaniel/claude-seo) (Claude Code skill Tier 4) téléchargé localement. **Il n'est PAS installé comme plugin Claude Code** — il faut suivre les `SKILL.md` à la main.

### Ressources à consulter en priorité (chemins absolus)

| Besoin | Fichier |
|---|---|
| Règles Google officielles | `claude-seo-main/pdf/google-seo-reference.md` |
| On-page audit | `claude-seo-main/skills/seo-page/SKILL.md` |
| Technical audit | `claude-seo-main/skills/seo-technical/SKILL.md` |
| Schema.org (Person, ProfessionalService) | `claude-seo-main/skills/seo-schema/SKILL.md` + `schema/templates.json` |
| **Local SEO (cœur Site-Maman)** | `claude-seo-main/skills/seo-local/SKILL.md` |
| Images / alt / formats | `claude-seo-main/skills/seo-images/SKILL.md` |
| Content E-E-A-T (santé mentale) | `claude-seo-main/skills/seo-content/SKILL.md` |
| Schema validation hook | `claude-seo-main/hooks/validate-schema.py` |

### Sub-skills à activer pour Site-Maman

✅ **Prioritaires** :
`seo-local`, `seo-schema`, `seo-page`, `seo-technical`, `seo-images`, `seo-content`

❌ **Désactivées (hors scope)** :
- `seo-hreflang` — mono-langue FR
- `seo-ecommerce` — pas de boutique
- `seo-programmatic` — site one-page
- `seo-backlinks` — pas de campagne de netlinking pour l'instant
- `seo-cluster` / `seo-competitor-pages` — pas de blog/cluster
- `seo-maps` — Google Business Profile pourra être activé après mise en ligne (TODO), mais hors scope du code HTML

### Scripts utiles sans API key

- `parse_html.py <file>` — extracteur HTML pour audit on-page
- `fetch_page.py <url>` — récupération page avec rotation UA
- `validate-schema.py <file>` — vérifie JSON-LD (placeholders + types dépréciés)

---

## 2. Spécifications Site-Maman (NAP + business)

### Identité

| Champ | Valeur |
|---|---|
| **Nom légal / commercial** | Emmanuelle Francblu Blecher |
| **Métier** | Psychanalyste & praticienne certifiée en hypnose ericksonienne |
| **Schema.org primary types** | `Person` + `ProfessionalService` + `WebSite` (`@graph`) |
| **Catégorie GBP recommandée** | "Psychanalyste" (primaire) + "Hypnothérapeute" (secondaire) |
| **Slogan / positionnement** | Cabinet bienveillant, serein et confidentiel · Paris 11e République |

### NAP (Name · Address · Phone) — référence absolue

| | Valeur exacte |
|---|---|
| **Adresse** | 17 boulevard Jules Ferry, 75011 Paris, France |
| **Téléphone (E.164)** | `+33616984678` |
| **Téléphone affiché** | +33 6 16 98 46 78 |
| **Email** | efb@relations-psy.fr |
| **Coordonnées géo (approx.)** | Latitude 48.8666, Longitude 2.3681 (centre du 17 bd Jules Ferry, Paris 11e) |

⚠️ **Règle de cohérence absolue** : tout NAP affiché doit être strictement identique dans le HTML visible, le JSON-LD `ProfessionalService`/`Person`, et la future fiche Google Business Profile. Toute divergence = pénalité SEO local.

### Horaires

> Sur rendez-vous uniquement — du lundi au samedi. Visio possible.

Pas d'horaires fixes affichés (sur RDV) → on **ne renseigne PAS** d'`openingHoursSpecification` rigide dans le schema (évite le mensonge Google). On indique `availableService` + `serviceArea` + statut "sur RDV" en texte libre.

### Zone cible (local SEO)

- **Quartier** : République / Saint-Ambroise
- **Arrondissement** : Paris 11e (75011)
- **Ville** : Paris
- **Région** : Île-de-France
- **Pays** : FR
- **Métro accès** : République (lignes 3, 5, 8, 9, 11)

### Mots-clés prioritaires (intent local)

**Tête de longue traîne** (très concurrentielle)
- psychanalyste Paris · hypnose Paris · hypnose ericksonienne Paris

**Longue traîne (à privilégier)**
- psychanalyste Paris 11 · psychanalyste République · psychanalyste Paris 75011 ·
- hypnose ericksonienne Paris 11 · hypno-praticien Paris République ·
- thérapie brève Paris 11 · approche systémique Paris · cabinet psychanalyse 11e ·
- consultation psy Paris République · MBSR Paris

**Branded**
- Emmanuelle Francblu Blecher · cabinet Francblu Blecher · relations-psy

**Intent transactionnel**
- prendre rendez-vous psychanalyste Paris 11 · consulter psychanalyste République ·
- séance hypnose ericksonienne Paris · première consultation psy Paris

### Specs Schema.org pour le @graph

- **Person** : name, jobTitle, alumniOf (formations), knowsAbout, worksFor → cabinet
- **ProfessionalService** : name, image, address PostalAddress (FR), telephone, email, areaServed (Paris 11e + Île-de-France), priceRange, paymentAccepted, hasMap, sameAs (relations-psy.fr quand confirmé)
- **WebSite** : url, name, inLanguage `fr-FR`, publisher → Person

⚠️ **Ne PAS utiliser** :
- `MedicalBusiness` / `Physician` — une psychanalyste n'est pas médecin, c'est trompeur
- `FAQPage` schema — restreint gov/healthcare seulement depuis août 2023
- `HowTo` (déprécié sept 2023)
- `SpecialAnnouncement` (déprécié juillet 2025)

---

## 3. Règles automatiques de Claude pour ce projet

### Règle d'or — Audit SEO automatique à chaque modification HTML

> **À chaque modification HTML sur `index.html` (et tout nouveau fichier HTML), Claude DOIT automatiquement vérifier les implications SEO sans attendre que l'utilisateur le demande**.

Checklist auto à chaque Edit/Write HTML :
1. Le `<title>` reste 50–60 caractères avec keyword local (Paris 11 / République) ? ✓
2. Meta description reste 150–160 caractères ? ✓
3. Pas plus d'un `<h1>` (visible ou `visually-hidden`) ? ✓
4. Tout nouvel `<img>` a-t-il `alt`, `width`, `height`, `loading` approprié, format moderne ? ✓
5. Toute nouvelle section sémantique → impact schema ? (ex : avis patients → **ne PAS** ajouter aggregateRating auto-déclaré ; section tarifs → `priceSpecification`)
6. Liens externes — pas de `href="#"` placeholder ? ✓
7. Cohérence NAP — adresse, téléphone, email affichés ≡ schema JSON-LD ? ✓
8. JSON-LD présent et valide ? Si modifié, hook `validate-schema.py` doit passer ✓

### Garde-fous (E-E-A-T santé mentale = YMYL)

Le SEO de la santé mentale (YMYL — Your Money Your Life) est strict côté Google :
- **JAMAIS** de claim médical non sourcé ("guérit", "soigne", "thérapie efficace prouvée")
- **TOUJOURS** mettre en avant les **formations vérifiables** (déjà fait : Faculté Libre, A.R.C.H.E., MBSR Jon Kabat-Zinn, Palo Alto, Altrando, Paris 8) → c'est l'**Expertise** de E-E-A-T
- **JAMAIS** de faux témoignages, ni d'aggregateRating non-vérifié
- **TOUJOURS** rappeler "1ère prise de contact gratuite et sans engagement" → CTA conforme déontologie
- Préférer "accompagnement" / "consultation" plutôt que "traitement" / "soin médical"

### Single source of truth

Site **mono-page** : un seul `index.html` à la racine. Avant la bascule SEO finale :
- `index.html` = version originale (garde-fou)
- `index.seo.html` = version de travail SEO
À la bascule (Phase 5) : `index.html` → `index-legacy.html`, `index.seo.html` → `index.html`.

### Police bannie

⚠️ **Cormorant Garamond bannie** (cf. mémoire utilisateur globale). La version originale utilisait Cormorant Garamond + Jost. Remplacement appliqué dans la version SEO :
- **Titres** : `Fraunces` (variable serif éditorial, axes opsz/wonk/soft pour adoucir — adapté au registre thérapeutique)
- **Corps** : `Inter` (sans-serif lisible, neutre)
- Combo validé sur d'autres projets de l'utilisateur (cf. `reference_validated_fonts.md`).

### Domaine canonique (PROVISOIRE)

> ⚠️ **À MIGRER quand un domaine custom sera branché.**

URL provisoire actuelle : `https://site-maman.vercel.app` (preview Vercel typique pour le repo `elieblch/site-maman`).
Domaine final probable : `relations-psy.fr` (cf. email `efb@relations-psy.fr`).
Statut : URL preview. **Doit être remplacée** dès la mise en prod réelle.

**Marqueur** : un commentaire HTML `@canonical-domain` est placé en haut de `index.seo.html` (porté sur `index.html` à la bascule).

**Commande de migration en une fois** (depuis la racine du projet) :
```bash
grep -rln "site-maman.vercel.app" . \
  --include='*.html' --include='*.xml' --include='*.txt' --include='*.md' \
  | xargs sed -i '' 's|https://site-maman.vercel.app|https://NOUVEAU-DOMAINE|g'
```

**Endroits affectés** :
| Fichier | Occurrences |
|---|---|
| `index.html` | `<link rel="canonical">`, `og:url`, JSON-LD `@id` × 3, `url` × 2, `logo`, `image` × 2 (~10) |
| `sitemap.xml` | `<loc>` |
| `robots.txt` | `Sitemap:` |
| `mentions-legales.html` / `politique-confidentialite.html` | éventuelles refs |
| `CLAUDE.md` | exemples |

Voir `MIGRATION-DOMAINE.md` (généré par bloc 4) pour la procédure complète.

---

## 4. Workflow standard pour ce projet

1. **Toute modif HTML** → checklist auto + suggestion proactive des schemas concernés
2. **Ajout d'images** → recommander conversion WebP + commande exacte (`cwebp -q 75 …` pour LCP, `-q 82` pour photos sous fold)
3. **Avant tout commit** → vérifier que `validate-schema.py` passe
4. **Push vers main** → Vercel redéploie automatiquement
5. **Après mise en ligne** → recommander : (a) claim GBP, (b) inscrire Apple Business Connect + Bing Places, (c) soumettre sitemap à Search Console

---

## 5. TODOs restants (action user / cliente "Maman")

Le site est **prêt à pousser en preview** mais quelques éléments nécessitent une intervention humaine. Grep :
```bash
grep -rn "TODO\|🔴" . --include='*.html' --include='*.txt' --include='*.md'
```

### 🔴 PRIORITÉ ABSOLUE — LE JOUR DE LA BASCULE DOMAINE

**Désactiver le `noindex` provisoire** (mis pour la phase preview Vercel). Sans cette action, le site sur le vrai domaine restera **invisible à Google**. Procédure dans `MIGRATION-DOMAINE.md` :

1. **3 fichiers HTML** — retirer la balise `<meta name="robots" content="noindex, nofollow">` du `<head>` de :
   `index.html`, `mentions-legales.html`, `politique-confidentialite.html`.

2. **`robots.txt`** — remplacer le contenu provisoire (`Disallow: /`) par la version production (Allow: / + 9 AI crawlers explicites).

3. **Vérification post-bascule** :
   - Tester sur https://www.google.com/search?q=site:NOUVEAU-DOMAINE (48-72h)
   - Soumettre `sitemap.xml` dans Google Search Console
   - "Live test" sur l'URL principale dans GSC pour forcer le recrawl

### 🔴 Priorité haute — à faire avec "Maman"

1. **Domaine de production custom** — actuellement preview Vercel. Si `relations-psy.fr` est dispo et qu'elle veut s'en servir, suivre `MIGRATION-DOMAINE.md`.

2. **Fiche Google Business Profile (GBP)** — créer/claim sur https://business.google.com :
   - Catégorie primaire : "Psychanalyste"
   - Catégorie secondaire : "Hypnothérapeute"
   - Adresse : 17 boulevard Jules Ferry, 75011 Paris
   - Téléphone : +33 6 16 98 46 78
   - Site web : URL finale du domaine
   - **Récupérer le `place_id`** pour pointer l'iframe Google Maps sur la fiche GBP (et ses avis) : actuellement l'iframe cherche par adresse.

3. **Vraies URLs sociales (si compte existant)** — actuellement aucune URL sociale n'est dans le footer. Si Mme Francblu Blecher a un LinkedIn pro, un compte Doctolib, ResaLib, ou autre annuaire pro, les ajouter au schema JSON-LD champ `sameAs`.

4. **Infos légales obligatoires** — marquées `<span class="todo">À COMPLÉTER PAR LA PRATICIENNE</span>` dans `mentions-legales.html` et `politique-confidentialite.html` :
   - Statut juridique (libéral / micro-entreprise / EI)
   - N° SIRET
   - N° ADELI ou RPPS (si applicable pour hypnothérapeute)
   - Assurance professionnelle (RC Pro) : nom assureur + n° police + couverture géographique
   - Médiateur de la consommation (obligatoire pour toute activité de service à client particulier — choisir parmi les médiateurs agréés CECMC)

### 🟡 Priorité moyenne — à faire après mise en ligne réelle

5. **Vraies photos optimisées** — les photos `Bureau1.jpg`, `Bureau2.jpg`, `Exterieur.jpg` ont été converties en WebP (bloc 2). Si de meilleures photos sont disponibles plus tard :
   ```bash
   cwebp -q 75 -metadata all photo-XX.jpg -o photo-XX.webp   # LCP / hero
   cwebp -q 82 -metadata all photo-XX.jpg -o photo-XX.webp   # sous le fold
   ```

6. **Google Search Console** — vérifier le domaine + soumettre `sitemap.xml`

7. **Bing Webmaster Tools** — soumettre le sitemap (alimente ChatGPT, Copilot)

8. **Apple Business Connect** — claim sur https://businessconnect.apple.com (Apple Maps + Siri)

9. **Annuaires professionnels santé mentale** (pertinents pour psychanalyste) :
   - Doctolib (si accepte la prise de RDV en ligne)
   - PsyEnLigne / Psychologue.net (si applicable)
   - Pages Jaunes (annuaire local FR)

### 🟢 Priorité basse — v2 éventuelle

10. **Page blog / ressources** — articles E-E-A-T (qu'est-ce que la psychanalyse, hypnose ericksonienne, MBSR…) pour ranker sur des recherches informationnelles. Mais attention : santé mentale = YMYL, niveau de qualité éditorial très élevé requis.
11. **Page tarifs** dédiée — si Mme Francblu Blecher accepte d'afficher des fourchettes.
12. **Section témoignages anonymisés** — uniquement si vrais témoignages avec accord écrit ; **ne PAS** ajouter aggregateRating schema (Google ignore le self-serving review markup).

---

## 6. Référence rapide aux commandes du plugin

Si l'utilisateur demande un audit complet, suivre (en exécutant le script Python correspondant ou en suivant la `SKILL.md` à la main) :

| Tâche | Sub-skill | Action |
|---|---|---|
| Audit complet | `seo-audit` | suivre `skills/seo-audit/SKILL.md` |
| Single page | `seo-page` | suivre `skills/seo-page/SKILL.md` |
| Technical | `seo-technical` | suivre `skills/seo-technical/SKILL.md` |
| Schema | `seo-schema` | suivre `skills/seo-schema/SKILL.md` |
| Local | `seo-local` | suivre `skills/seo-local/SKILL.md` |

(Le slash command natif `/seo …` ne fonctionne PAS — le plugin n'est pas installé.)

---

*Dernière mise à jour : 2026-05-20 (création SEO complète : schema Person+ProfessionalService, OG/Twitter/canonical, favicons, WebP+preload, mentions légales, sitemap, robots, noindex preview Vercel temporaire)*
