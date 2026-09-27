# Déploiement Waifly (Pterodactyl)

Egg **NodeJS**, Node **>= 20**.

## Démarrage

Commande de démarrage exacte : `npm start`
Cela exécute `node dist/index.js`.

Ne **pas** uploader `node_modules` : le panel lance `npm install` (production, sans devDependencies).

## Configuration

1. Copier `.env.example` vers `.env`.
2. Renseigner `DISCORD_TOKEN`, `CLIENT_ID`, `GUILD_ID` (optionnel : `PRIVATE_SERVER_URL`, lien du serveur privé Roblox ; sinon lien par défaut ou `/admin serveur-prive`).
3. Activer le **redémarrage automatique** (auto-restart).

## Données

SQLite persiste dans `data/` (`data/aaf.sqlite`). Monter ce dossier en volume persistant si Waifly le propose.

## Build précompilé

Ce zip inclut déjà `dist/`. En production il n’y a pas TypeScript/tsx : si `dist/` est absent après l’upload, il faut le zip qui contient le `dist/` précompilé.

## Lot A v2 — Roblox + 📡・frequence (après déploiement)
- Migration automatique au démarrage (table `roblox_links`, colonnes `live_sessions.frequencies` / `idle_closed_day`) : idempotente, rien n'est supprimé.
- Les commandes (`/staff session frequences`, `/admin roblox set|unlink`) sont réenregistrées au démarrage.
- Lancer `/setup` une fois : crée la catégorie SESSION (📡・frequence + 🛫・session-live déplacé, vocal muet pour les membres), passe 🎮・ptfs en staff, publie l'embed dans 📡・frequence et supprime l'ancien de 🎮・ptfs.
- Les membres refont leur inscription (bouton de filière) pour lier et vérifier leur compte Roblox : sans lien vérifié, aucune heure n'est comptée.
- Présence Roblox : API publique (sans cookie). Une minute compte si Roblox indique le joueur en jeu sur PTFS, ou en jeu avec le jeu masqué ; jamais pour un autre jeu visible, le site / l'appli, Studio ou hors ligne. Le joueur doit laisser « Statut en ligne » sur « Tout le monde » (Paramètres › Confidentialité et restrictions de contenu › Visibilité & serveurs privés).

## Lots B+C — paliers par heures + nouvelle carte des salons (après déploiement)
- Migrations automatiques et idempotentes au démarrage : tables `career_floors` et `palier_awards` ; chaque licence déjà délivrée (`certifications`, ex. `pilote-2`) devient un plancher (Pilote / Contrôleur = 4 h, Senior = 11 h). Aucune donnée supprimée (XP, ₳, modules, certifications restent en base).
- Au démarrage : synchronisation silencieuse des paliers (aucun MP, aucune ligne dans ✅・résultat) et une seule ligne de résumé dans 📝・logs. Les rôles de carrière déjà posés deviennent des planchers (jamais retirés).
- Commandes réenregistrées au démarrage : `/staff session open|close|status|frequences|heures`, `/staff ticket`, `/admin heures|roblox|paliers|certification|suspendre|annonce`.
- Lancer `/setup` une fois : crée les rôles de l'échelle (réutilise les rôles existants au même nom, ne crée jamais Instructeur Pilote / Instructeur ATC), applique la carte ACCUEIL · SESSION · PARCOURS · COMMUNAUTÉ · AIDE · VOCAL · STAFF · VOCAL STAFF (renommages 📂・mon-dossier → 📂・mon-parcours, 🎫・support → 🎫・aide, SUPPORT → AIDE, VOICE → VOCAL ; salons de formation masqués, jamais supprimés), supprime les catégories ANNONCES / PARTENARIATS / FORMATION / MEMBRES seulement si elles sont vides, republie règlement / inscriptions / aide / dossier et retire les anciens panneaux du bot.
- Vérifier ensuite que le rôle du bot est au-dessus des rôles de l'échelle.
- Nettoyage optionnel : `/workspace/aaf-audit/cleanup-post-deploy.mjs` (dry-run par défaut, `--apply` pour agir).

## Cut 1 — suppression des anciens salons + bouton serveur privé (26/09)
- Code : les salons 📚・formation-pilote, 📚・formation-atc, 📋・demandes, 📝・grille-évaluation, 🎙️・doc-phraséologie, 👋・bienvenue, 🎮・ptfs, 🔑・serveur-privé et les vocaux 🧑‍✈️・Formation pilote / 🎧・Formation ATC sont retirés de la carte : `/setup` ne les recrée plus et ne les touche plus (ligne « ancien(s) salon(s) retiré(s) … » tant qu'ils existent). Leurs ids stockés sont purgés de la config.
- 📡・frequence : nouveau bouton lien « Rejoindre le serveur privé » (visible par tous), édité en place au prochain rafraîchissement (démarrage, `/setup`, changement d'état). `/admin serveur-prive` change / masque / rétablit le lien.
- Vérifier après déploiement : `dist/services/private-server.js` présent ; la commande `/admin serveur-prive` apparaît ; le panneau 📡・frequence montre le 3ᵉ bouton.
- Puis seulement : `node /workspace/aaf-audit/delete-old-channels.mjs` (dry-run) puis `--apply` (crée une invitation permanente vers 📜・règlement si besoin, supprime exactement ces 10 salons par id).

## Rename 1 — noms plus clairs (26/09)

Catégories : ACCUEIL → COMMENCER ICI, SESSION → SESSION DU SOIR, PARCOURS → MA PROGRESSION, AIDE → BESOIN D'AIDE, VOCAL → DISCUSSION VOCALE (COMMUNAUTÉ, STAFF, VOCAL STAFF inchangées).
Salons : 📜・règlement → 📜・1-règlement, 📝・inscriptions → ✍️・2-inscription, 📡・frequence → 📡・infos-session, 🛫・session-live → 🛫・vocal-session, ✅・résultat → 🏅・nouveaux-brevets, 📂・mon-parcours → 📂・mes-heures, 🎫・aide → 🎫・ouvrir-un-ticket, 🔊・salon-général → 🔊・discussion, 💬・général (STAFF) → 💬・discussion-staff. Le 💬・général de COMMUNAUTÉ ne change pas.

- Déployer d'abord. Au démarrage, les clés de config sont migrées vers les nouveaux noms (`migrateRenamedConfigKeys`, mêmes ids) : tout fonctionne avant et après le renommage Discord. Les commandes slash (descriptions) sont réenregistrées.
- Vérifier après déploiement : `dist/db/purge-orphans.js` contient `migrateRenamedConfigKeys` ; `dist/constants.js` contient `📡・infos-session` et `SCOPED_CHANNEL_ALIASES`.
- Puis : `node /workspace/aaf-audit/rename-channels.mjs` (dry-run) puis `--apply` (renomme par id, raison d'audit « AAF: noms plus clairs (go Oscar 26/09) », réordonne SESSION DU SOIR : 📡・infos-session, 🛫・vocal-session, 🏅・nouveaux-brevets).
- Puis lancer `/setup` une fois : réédite en place le panneau du règlement (nouveau texte), enregistre les ids sous les nouveaux noms et applique l'ordre des salons. `/setup` seul (sans le script) renomme aussi en place grâce aux alias, sans jamais créer de doublon.

## Embed1 — nouveau panneau 📡・infos-session (26/09)

- Titres : « 🔴 Session fermée » / « 🟢 Session en cours, vos heures comptent » / « 🟠 Session en pause, heures suspendues », puis le bloc fixe « Comment participer » (mentions <#…> de 🛫・vocal-session et ✍️・2-inscription).
- Bouton « Où j’en suis » retiré du panneau (📂・mes-heures → Consulter). Rangée membre : ✈️ Rejoindre le serveur privé puis 🛫 Rejoindre le vocal ; rangée staff inchangée. Un ancien clic « Où j’en suis » répond toujours (même id que Consulter).
- Signature `v5` : le message existant est édité en place au premier tick après le démarrage (≈ 1 min), aucun nouveau message ; pas besoin de `/setup`.
- Vérifier après déploiement : `dist/services/progress-view.js` contient `liveHowToBlock` et « 🟢 Session en cours, vos heures comptent » ; `dist/services/live-sessions.js` contient `"v5"`.

## StaffPanel1 — 🎛️・gestion-session + 📊・statistiques en direct (26/09)

- Les boutons staff Ouvrir / Fermer / Fréquences quittent 📡・infos-session (signature `v6` : le panneau membre est édité en place au premier tick, il ne garde que ✈️ Rejoindre le serveur privé + 🛫 Rejoindre le vocal). Un clic sur un ancien bouton staff fonctionne toujours (mêmes ids).
- Au démarrage : 🎛️・gestion-session est créé dans STAFF (juste après 💬・discussion-staff) s'il n'existe pas, puis le panneau staff y est publié une fois et édité en place ; l'embed 📊・statistiques est publié une fois (ou un ancien message du bot au même titre est repris) puis édité toutes les 10 min et après chaque fermeture de session. Pas besoin de `/setup` (il fait la même chose sans doublon, et remet l'ordre des salons STAFF).
- Base : colonnes `staff_panel_*` / `stats_*` ajoutées à `live_sessions`, nouvelle table `live_session_history` (migration automatique, idempotente, rien n'est supprimé).
- Vérifier après déploiement : `dist/services/stats-panel.js` et `dist/services/staff-panels.js` présents ; `dist/services/live-sessions.js` contient `"v6"` et `liveStaffPanelPayload`.

## Tests1 — grades par heures + test (27/09)

- Les heures débloquent le test du palier ; le grade est donné quand le test est réussi ET les heures atteintes. 1 h : quiz 5 questions (4/5) ; 4 h : quiz 10 questions (8/10) ; 11 h et 24 h : « 🎯 Demander la validation » → message dans 🎛️・gestion-session avec « ✅ Valider » / « ❌ Pas encore » (Fondateur, Administrateur, Modérateur, permission Administrateur). 24 h entre deux essais après un échec.
- Banque de questions : `src/data/quiz-questions.json`, copiée dans `dist/data/` par `npm run build` (`scripts/copy-data.mjs`). Vérifier après déploiement : `dist/data/quiz-questions.json` et `dist/services/grade-tests-discord.js` présents.
- Base : nouvelle table `grade_tests` (migration automatique, rien n'est supprimé). Au démarrage, une seule fois : les rôles de carrière déjà portés sont enregistrés comme grades obtenus (aucune rétrogradation ; les paliers déjà enregistrés restent).
- Au démarrage : le panneau 📂・mes-heures (« 📂 Mes heures », bouton « 📂 Voir mes heures ») et le règlement sont édités en place, sans republier.
