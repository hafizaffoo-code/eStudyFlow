# eStudyFlow — MVP

Application web de suivi des démarches étudiantes, des documents, des heures travaillées et des candidatures. Les comptes sont authentifiés par Firebase et leurs données sont enregistrées dans Firestore.

## Démarrer en local

Prérequis : Node.js 20 ou plus récent et Java 21 ou plus récent (requis par les émulateurs Firebase).

```bash
npm install
npm run dev:local
```

Ouvre ensuite [http://localhost:3003](http://localhost:3003). Firebase Authentication et Firestore tournent localement ; la console des émulateurs est disponible sur [http://localhost:4000](http://localhost:4000). Pour tester sur un téléphone connecté au même Wi-Fi, ouvre `http://<adresse-IP-de-l’ordinateur>:3003`. Les émulateurs écoutent sur le réseau local et le navigateur utilise automatiquement le même nom d’hôte pour les joindre. Les comptes et les données de cette instance sont de test, locaux et effacés lorsque les émulateurs s’arrêtent.

## Firebase en production

Le mode local utilise un identifiant de projet de démonstration et ne contacte pas le projet Firebase distant. En production, l’application utilise la configuration de [`firebase-applet-config.json`](./firebase-applet-config.json) : Firebase Authentication doit activer **Email/Password** pour les comptes récupérables et **Anonymous** si l’accès invité est proposé ; il faut aussi publier [`firestore.rules`](./firestore.rules) et créer les documents `/admins/{uid}` nécessaires depuis un processus de confiance. Une inscription récupérable demande un nom d’utilisateur, une adresse e-mail réelle et un mot de passe d’au moins 6 caractères. Les nouveaux comptes se connectent avec leur adresse e-mail ; les comptes historiques peuvent utiliser leur nom d’utilisateur tant qu’ils n’ont pas associé d’adresse e-mail. Après confirmation d’une nouvelle adresse, ils se connectent avec celle-ci. Le lien « Mot de passe oublié ? » envoie un lien de réinitialisation à l’adresse associée au compte. Les comptes historiques sans adresse e-mail peuvent en ajouter une depuis leur profil après s’être connectés, puis la confirmer depuis leur boîte de réception. Une erreur de connexion ne crée pas de session invitée : l’accès invité doit être choisi explicitement.

« Continuer en invité » crée un compte Firebase anonyme ; ce n’est pas une connexion Google. Les données de cette session restent liées à cet appareil et ne peuvent pas être récupérées après déconnexion ou sur un autre appareil. Le fournisseur **Anonymous** est nécessaire uniquement si ce mode invité reste proposé.

Les offres sont lues depuis la collection Firestore `jobs` ; aucune API d’offres n’est connectée. Cette collection doit être alimentée uniquement avec des offres vérifiées et des liens de candidature autorisés. Les actualités sont récupérées côté serveur depuis les pages officielles de Service-Public.fr et de l’Assurance Maladie, avec un cache de 30 minutes ; les vues Accueil et Actualités utilisent la même source. Les sources temporairement inaccessibles sont signalées dans l’interface.

Les dossiers et statuts, heures travaillées, préférences d’affichage des notifications, candidatures enregistrées, idées envoyées et conversations IA sont associés au compte authentifié. L’adresse e-mail du compte récupérable est gérée par Firebase Authentication et incluse dans l’export du profil. Les notifications push et la génération automatique des alertes ne sont pas encore configurées ; les rappels calendrier sont exportés sous forme de fichier `.ics`. L’application ne conserve pas les données Firestore hors ligne après fermeture/rechargement : seul le shell PWA est précaché. L’export est généré depuis les données Firestore ; la suppression de compte efface les sous-collections prises en charge et demande le mot de passe pour les comptes classiques (pas pour les comptes invités anonymes).

Le parcours invité est anonyme et irrécupérable après déconnexion ou changement d’appareil. La publication bêta nécessite également une politique de confidentialité et une information sur les traitements Firebase, les actualités récupérées et le fournisseur IA.

## Assistant IA (facultatif)

Ajoute une clé Gemini **côté serveur uniquement** dans `.env` :

```dotenv
GEMINI_API_KEY=...
GEMINI_MODEL=gemini-3.1-flash-lite
```

Sans clé, l’assistant indique qu’il n’est pas configuré et ne fournit pas de réponse juridique de secours. Son endpoint exige un jeton Firebase valide et applique un quota en mémoire par compte.

## Vérification avant déploiement

1. Configurer Firebase Authentication (Email/Password requis pour les comptes récupérables ; Anonymous facultatif pour le mode invité), les règles Firestore et les domaines autorisés dans Firebase Console.
2. Ajouter la clé Gemini au `.env` du serveur si eStudy AI doit être activé ; ne pas la préfixer avec `VITE_`.
3. Alimenter `jobs` avec des offres réelles et maintenir leurs dates et URL de candidature.
4. Choisir un stockage partagé du quota IA si plusieurs instances serveur sont déployées.
5. Configurer un domaine HTTPS et la politique de confidentialité avant d’ouvrir l’application à des utilisateurs externes.

En production, générer le client avec `npm run build`, puis démarrer le serveur API et les fichiers statiques avec `npm start`.

## Vérifications

```bash
npm run lint
npm run build
```

Le projet ne dispose pas encore d’une suite de tests automatisés.
