# Académie Aéronautique Française — Bot Discord

Bot Discord de l’**Académie Aéronautique Française (AAF)**, école francophone pour les joueurs de **PTFS** (Pilot Training Flight Simulator, Roblox). Il gère l’accueil, l’inscription (avec vérification du compte Roblox), les sessions live et la carrière par heures : les rôles sont attribués automatiquement selon le temps validé en session (pas de cours, pas d’examen, pas d’évaluation instructeur).

Le bot ne pilote jamais le jeu : il lit seulement la présence Roblox publique (API officielle, sans cookie).

## Prérequis

- Node.js 20 ou supérieur
- Un serveur Discord (guilde) vierge ou existant
- Les intents **Server Members Intent** et **Guild Voice States** activés sur l’application (pas de Message Content Intent)

## 1. Créer l’application Discord

1. Ouvrez le portail développeur Discord et cliquez sur **New Application**.
2. Donnez-lui un nom (ex. AAF — Académie), acceptez les conditions, puis **Create**.
3. Onglet **Bot** : Reset Token, copiez le jeton. Activez **Server Members Intent** et **Guild Voice States**. N’activez pas Message Content Intent.
4. Onglet **OAuth2 → URL Generator** : scopes bot et applications.commands.
5. Permissions : Administrator (recommandé pour la commande setup) ou au minimum Manage Channels, Manage Roles, Send Messages, Embed Links, Attach Files, Read Message History, Use Slash Commands, Manage Messages, View Channels, Connect, Speak.
6. Ouvrez l’URL générée, sélectionnez votre serveur, autorisez le bot.
7. Placez le rôle du bot au-dessus des rôles qu’il doit attribuer (Paramètres du serveur → Rôles).

Pour copier les identifiants : activez le Mode développeur (Paramètres Discord → Avancés), clic droit sur le serveur → Copier l’identifiant. L’Application ID est dans General Information.

## 2. Configuration

Copiez .env.example vers .env puis renseignez DISCORD_TOKEN, CLIENT_ID et GUILD_ID.


## 3. Installation et lancement

Copiez .env.example vers .env, puis :

    npm install
    npm start

Enregistrement manuel des commandes slash : npm run register

Le bot est pret lorsqu il affiche AAF en ligne. Ensuite, en tant qu administrateur Discord, lancez la commande setup.

Setup est idempotent : serveur vide = creation complete. Deja configure = reparation sans duplication.

Les tests : npm test


## 4. Parcours membre (Lots B+C)

Les membres n'utilisent **pas** de commandes slash, seulement des boutons.

1. 📜・1-règlement → **J'accepte le règlement** → rôle Membre (sans Membre, seule la catégorie COMMENCER ICI est visible).
2. ✍️・2-inscription → Pilote / Contrôleur / Les deux, UserId Roblox, code de vérification dans le profil Roblox → rôle Élève + Élève Pilote / Élève Contrôleur.
3. Chaque soir 20:30–21:30 : en jeu (serveur privé, ATC Radio) + vocal muet 🛫・vocal-session ; état dans 📡・infos-session (boutons **Rejoindre le serveur privé** puis **Rejoindre le vocal**, et le bloc « Comment participer »). Le staff pilote la session dans 🎛️・gestion-session (Ouvrir / Fermer / Fréquences).
4. Paliers automatiques (`src/services/hours-ladders.ts`, source unique) : Novice 1 h (MP), Pilote / Contrôleur 4 h, Senior 11 h, Commandant de bord / Chef de tour 24 h ; doubles 5+5 / 11+11 / 24+24 h. À partir de 4 h et pour les doubles : MP + une ligne dans 🏅・nouveaux-brevets. Titres (2 h, 7 h, 16 h) et badges : dossier seulement.
5. 📂・mes-heures → **Consulter** (heures, palier suivant). 🎫・ouvrir-un-ticket → **Ouvrir un ticket** (fermeture auto après 48 h sans message).

Salons staff (StaffPanel1, 26/09) :
- 🎛️・gestion-session : panneau « état de la session » (titre, fréquences, présents, ouverture forcée ou non) + boutons **Ouvrir / Fermer / Fréquences**. Visible en lecture seule par les rôles autorisés au pilotage live (instructeurs, Examinateur, modération, Responsable Formation et direction) ; seul le Fondateur peut écrire. Créé au démarrage s'il manque (et par `/setup`), jamais en double.
- 📊・statistiques : embed « 📊 Statistiques de l'Académie » (membres, filières, heures, sessions, paliers, top 5 sur 7 jours, état), un seul message édité en place toutes les 10 min et après chaque session ; bouton **🔄 Actualiser** (Fondateur / administrateurs, 10 s entre deux clics). Historique des sessions : table `live_session_history` (à partir de cette version).

Commandes staff :
- `/setup` (idempotent, suit la carte de salons du plan §5)
- `/staff session open|close|status|frequences <liste>|heures @membre` (Instructeur, Modérateur et +), `/staff ticket` (modération et +)
- `/admin heures set|add|remove`, `/admin roblox set|unlink`, `/admin paliers [membre] [resync]`, `/admin certification accorder|retirer` (plancher, journalisé dans 🚨・alertes), `/admin suspendre` (bloque le crédit), `/admin annonce`, `/admin serveur-prive <lien|aucun|defaut>` (direction : lien du bouton « Rejoindre le serveur privé » de 📡・infos-session ; par défaut le lien de l'ancien salon 🔑・serveur-privé, surchargeable par la variable `PRIVATE_SERVER_URL`)

Salons retirés (Cut 1, 26/09) : 📚・formation-pilote, 📚・formation-atc, 📋・demandes, 📝・grille-évaluation, 🎙️・doc-phraséologie, 👋・bienvenue, 🎮・ptfs, 🔑・serveur-privé, vocaux Formation pilote / Formation ATC. Ils ne sont plus dans la carte (`RETIRED_CHANNEL_NAMES`) : `/setup` ne les recrée jamais et les signale s'ils existent encore. Les invitations pointent vers 📜・1-règlement.

Carte des salons (Rename 1, 26/09) :
- **COMMENCER ICI** : 📜・1-règlement, ✍️・2-inscription
- **SESSION DU SOIR** : 📡・infos-session, 🛫・vocal-session (vocal muet), 🏅・nouveaux-brevets
- **MA PROGRESSION** : 📂・mes-heures
- **COMMUNAUTÉ** : 📢・annonces, 🤝・partenariats, 💬・général
- **BESOIN D'AIDE** : 🎫・ouvrir-un-ticket (+ tickets)
- **DISCUSSION VOCALE** : 🔊・discussion
- **STAFF** : 💬・discussion-staff, 🎛️・gestion-session, 📊・statistiques, 🚨・alertes, 📝・logs · **VOCAL STAFF** : 🔒・Réunion staff

Les anciens noms (ACCUEIL, SESSION, PARCOURS, AIDE, VOCAL, 📜・règlement, 📝・inscriptions, 📡・frequence, 🛫・session-live, ✅・résultat, 📂・mon-parcours, 🎫・aide, 🔊・salon-général, 💬・général du STAFF) restent des alias (`CHANNEL_NAME_ALIASES`, `aliases` des catégories) : `/setup` renomme en place, sans doublon, et les clés de config sont migrées à la lecture (`migrateRenamedConfigKeys`).

## Heures en session live (Lot A — « vérité des heures »)

- Seul le vocal **🛫・vocal-session** compte. Une minute est créditée si, au tick de la minute : la session est **ouverte** (auto 20:30–21:30 Paris chaque soir ; avant 20:30 dès qu’un pilote et un contrôleur sont présents ; après 21:30 seulement si le staff l’a ouverte), il y a au moins 1 pilote et 1 ATC (2 personnes différentes), le membre a **Membre** + le rôle de filière, n’est ni muet serveur ni casque coupé (micro coupé OK), ni suspendu.
- Double filière : la même minute crédite les deux compteurs. Plafond en temps réel : 8 h sur 7 jours glissants (pas de plafond par jour). Segments < 3 min = 0.
- Données : `live_hours` (compteurs en secondes, crédit par minute entière), `live_days` (minutes par jour/soirée, présence à 20:30), `live_segments` (segment en cours, survit aux redémarrages), `live_hours_adjustments` (audit `/admin heures`). Migration automatique et idempotente au démarrage.
- Échelles et badges : `src/services/hours-ladders.ts` (module unique, à réutiliser en Lot B).
- Staff : `/staff session open|close|status|heures`, `/admin heures set|add|remove`.

## Tests

`npm test` : moteur d’heures, Roblox, paliers (annonces, planchers, synchronisation silencieuse), carte des salons, permissions, tickets.

## Donnees

Base SQLite dans data/aaf.sqlite.
