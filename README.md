# Royal Spin — casino multijoueur (points virtuels)

Projet de 2ᵉ année génie informatique. Application web temps réel : **roulette** multijoueur, **blackjack**, **machine à sous**, classement, salon de discussion et **mode écran géant** pour projeter la partie.

> Aucun argent réel : tout se joue en points virtuels (1 000 points au départ).

## Lancer en local

```bash
npm install
npm start
```

Le terminal affiche trois adresses :

| Page | Adresse | Usage |
|------|---------|-------|
| Croupier | `http://localhost:3000/host` | lancer la roue, mode automatique, statistiques |
| Écran géant | `http://localhost:3000/tv` | à projeter : roue, classement, QR code |
| Joueurs | `http://<IP du PC>:3000` | les téléphones (même WiFi) scannent le QR code |

Si le QR code pointe vers une mauvaise adresse (carte réseau virtuelle), choisissez la bonne dans la liste déroulante de la page croupier.

## Fonctionnalités

- Roulette européenne (0-36) : numéro plein (×36), rouge/noir, pair/impair, manque/passe, douzaines et colonnes
- Roue animée (canvas), sons (WebAudio), confettis, historique, numéros chauds/froids
- Blackjack contre le croupier (blackjack paie 3 pour 2, doubler)
- Machine à sous (jackpot ×250)
- Niveaux, badges, classement en direct, salon de discussion et réactions
- Mode automatique : compte à rebours, tirage et nouveau tour sans intervention
- Sauvegarde des joueurs dans `data.json`
- Installable sur téléphone (PWA)

## Architecture

```
server.js          Express + Socket.IO. Toute la logique (paris, tirage, gains,
                   blackjack, slots) est côté serveur.
public/
  index.html       page joueur            js/player.js
  tv.html          écran géant            js/tv.js
  host.html        croupier + stats       js/host.js
  js/shared.js     roue canvas, audio, confettis
  css/casino.css   thème
```

- **Temps réel** : événements Socket.IO (`bet`, `spin`, `state`, `me`, `chat`…). L'état public est diffusé à tous, l'état privé (mises, main de blackjack) uniquement au joueur concerné.
- **Anti-triche** : le client n'envoie que des intentions. Le serveur valide chaque mise (type, montant, solde), tire le numéro avec `crypto.randomInt` et calcule les gains. Les cartes du croupier restent cachées tant que la main n'est pas terminée.
- **Droits du croupier** : seul `localhost` (ou la clé `HOST_KEY` en ligne) peut lancer la roue ou réinitialiser.
- **Persistance** : `data.json`, écrit toutes les 2 s si quelque chose a changé.

## Mettre en ligne (Render, gratuit)

1. Créer un dépôt GitHub avec ce dossier (le fichier `render.yaml` est déjà prêt).
2. Sur render.com : **New → Blueprint**, choisir le dépôt, valider.
3. Dans **Environment**, lire la valeur de `HOST_KEY` (générée automatiquement). C'est le mot de passe de la page `/host`.
4. Ouvrir `https://<votre-app>.onrender.com/host`, saisir la clé, puis `/tv` sur le projecteur. Le QR code pointe automatiquement vers l'adresse publique.

Notes : l'offre gratuite s'endort après 15 min sans visite (premier chargement lent) et le disque est éphémère, donc `data.json` est remis à zéro à chaque redéploiement.

Variables d'environnement : `PORT`, `HOST_KEY`, `PUBLIC_URL`, `DATA_FILE`.
