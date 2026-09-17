# Cross scolaire – chrono et arrivées

Site d'une seule page : chronométrage, arrivées par QR code ou dossard, classement, import Excel, synchronisation en direct entre téléphones, iPad et ordinateurs.

## Principe

Une **journée** regroupe la liste des élèves et plusieurs **vagues** (5 par défaut, modifiable dans l'onglet Course). Chaque vague a son propre chrono ; le classement réunit ensuite toutes les vagues.

- **Chrono (adulte 1)** : lance le chrono, puis appuie sur « Arrivée » quand un élève franchit la ligne. L'heure est prise sur l'appareil au moment de l'appui, donc un réseau lent ne fausse pas les temps.
- **Arrivée (adulte 2)** : scanne les QR codes dans l'ordre de passage (entonnoir d'arrivée). En cas de QR perdu, tape le numéro de dossard : le nom s'affiche avant validation.
- **Classement** : dans chaque vague, le 1er temps va au 1er élève scanné, le 2e au 2e, etc. Vues disponibles :
  - **Général** : toutes les vagues réunies et triées par temps, filtrable par sexe ou par classe ;
  - **Moyennes de classe** : temps moyen, meilleur temps et rang moyen de chaque classe (aussi par sexe) ;
  - **Une vague** : pour vérifier et corriger les lignes incomplètes, surlignées.
- **Export Excel** : Général, Filles, Garçons, Moyennes classes (toutes, filles, garçons), une feuille par classe et une par vague.

Quand quelqu'un démarre une vague, tous les appareils basculent automatiquement dessus. Un élève déjà arrivé dans une autre vague est refusé. Si le fichier Excel indique la vague prévue, le site avertit quand un élève est scanné dans une autre vague.

### Lecteur de codes USB ou Bluetooth

Branchez ou appairez le lecteur, ouvrez l'onglet **Arrivée** et scannez : pas besoin de toucher l'écran. Le site reconnaît le lecteur à sa vitesse de frappe. Le dernier code lu s'affiche sous la caméra.

- Le lecteur doit terminer chaque code par « Entrée » (ou Tabulation) : c'est le réglage d'usine de la plupart des modèles.
- Réglez la langue de clavier du lecteur sur **Suisse** (ou la même que l'appareil) grâce aux codes de la notice. Si elle est fausse, le site retrouve quand même l'élève grâce au numéro de dossard en début de code, tant que l'élève est dans la liste.
- Sur iPad ou iPhone, un lecteur Bluetooth fait disparaître le clavier à l'écran : c'est normal. Pour le retrouver, éteignez le lecteur.
- Un code scanné sur un autre onglet est ignoré, avec un message : cela évite les erreurs sur l'appareil du chrono.

## Étape 1 – Mettre le site sur GitHub Pages

1. Créez un dépôt (ex. `cross-ecole`) et déposez-y `index.html`, `config.js` et ce README (bouton **Add file → Upload files**).
2. **Settings → Pages** : Source = *Deploy from a branch*, branche `main`, dossier `/ (root)`, **Save**.
3. Après une minute, le site est à `https://VOTRE-NOM.github.io/cross-ecole/`.

Le site marche déjà, mais sur un seul appareil à la fois.

## Étape 2 – Activer la synchronisation (Firebase, gratuit)

1. Sur <https://console.firebase.google.com>, **Créer un projet** (Google Analytics inutile).
2. **Build → Realtime Database → Créer une base**. Emplacement : **Belgique (europe-west1)** pour garder les données en Europe. Choisissez le *mode verrouillé*.
3. Onglet **Règles** de la base, remplacez tout par ceci puis **Publier** :

   ```json
   {
     "rules": {
       "races": {
         "$code": {
           ".read": "auth != null",
           ".write": "auth != null"
         }
       }
     }
   }
   ```

   Personne ne peut lister les courses : il faut connaître le code à 6 caractères.
4. **Build → Authentication → Commencer**, onglet *Sign-in method*, activez **Anonyme**.
5. **Paramètres du projet (roue dentée) → Vos applications → icône `</>`**, donnez un nom, puis copiez l'objet `firebaseConfig`.
6. Dans `config.js` sur GitHub (icône crayon), remplacez `null` par cet objet. Vérifiez que la ligne `databaseURL` est présente ; sinon, copiez l'adresse affichée en haut de la page Realtime Database.

La clé `apiKey` peut être publique : c'est normal pour Firebase, la protection vient des règles ci-dessus.

L'offre gratuite (Spark) permet 100 appareils connectés en même temps, largement assez.

## Le jour de la course

1. Sur l'ordinateur : **Course → Nouvelle journée**, puis **Élèves → Importer** le fichier Excel (voir `modele_eleves.xlsx`). Colonnes reconnues : Dossard, Nom, Prénom, Sexe (F/G, ou M/W), Classe, et Vague (facultative, 1 à 5). Sans colonne Dossard, les numéros sont attribués automatiquement.
2. Si besoin, renommez les vagues dans **Course → Vagues** (ex. « 7e filles »).
3. **Élèves → Imprimer les QR codes** et découpez les étiquettes.
4. Pour chaque vague : l'adulte au chrono choisit la vague et appuie sur **Démarrer**, les autres appareils suivent. Une fois tous les élèves arrivés, **Arrêter**.
5. Les autres adultes scannent le QR code affiché dans **Course** (ou saisissent le code) : ils voient tout en direct.
6. Chaque appareil garde l'écran allumé et enregistre une copie locale. Faites quand même **Télécharger une copie** à la fin.
7. Après la dernière vague : **Classement → Exporter tout en Excel**, puis **Supprimer toute la journée** pour ne pas laisser les données des élèves en ligne.

## À savoir

- La caméra exige HTTPS : c'est le cas sur GitHub Pages, pas en ouvrant le fichier depuis l'ordinateur.
- Sans réseau, les actions sont gardées en mémoire et envoyées au retour de la connexion. **Ne rechargez pas la page tant que vous êtes hors ligne**, sinon ces actions en attente sont perdues.
- Format des QR codes générés : `dossard;nom;prénom;sexe;classe`. Un QR code qui contient seulement le numéro de dossard fonctionne aussi.
- Le classement général compare des temps : il n'a de sens que si toutes les vagues courent **la même distance**.
- Les lignes incomplètes d'une vague (temps ou élève manquant) sont écartées du classement général jusqu'à correction ; un avertissement indique lesquelles.
- Évitez que deux vagues soient en course en même temps : un seul adulte à l'arrivée ne peut scanner que pour une vague à la fois.
- Une journée créée avec l'ancienne version (une seule course) est reprise automatiquement comme Vague 1.
