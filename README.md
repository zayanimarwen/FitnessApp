# App Sport — MVP Android (Kotlin / Compose / Room)

Suivi musculation + cardio sur 3 séances/semaine (A/B/C), progression automatique pilotée par le RPE,
suivi du tour de taille. 100 % offline, aucune donnée ne quitte le téléphone.

## Lancer
1. Android Studio Ladybug (2024.2) ou plus récent, JDK 17.
2. File > Open > dossier `AppSport`.
3. Le projet ne contient PAS `gradle-wrapper.jar` : si Android Studio le demande, laisse-le utiliser
   son Gradle, ou exécute `gradle wrapper --gradle-version 8.9` dans le dossier.
4. Sync, puis Run sur un téléphone/émulateur Android 8.0+ (API 26).
5. Tests du moteur de progression : `./gradlew test` (ou clic droit sur `app/src/test` > Run tests).

## Architecture
- `domain/` : Kotlin pur, sans Android — progression force (`ProgressionEngine`), cardio (`CardioProgression`),
  calendrier du programme, estimation calories. C'est la partie testée.
- `data/db` : Room (profil, exercices, prescriptions courantes, séances, logs par exercice, mesures corporelles).
- `data/prefs` : DataStore (date de début, semaine allégée, notifications, disclaimer).
- `data/seed` : catalogue Technogym (~29 exercices + alternatives) et programme initial A/B/C.
- `data/WorkoutRepository` : point unique d'accès aux données ; applique la progression en fin de séance.
- `ui/` : Compose + Material 3, un ViewModel par écran, DI manuelle (`AppContainer`).
- `notifications/` : WorkManager (rappel quotidien 8h) + notifications de fin de séance.

Extensions prévues : un `SyncSource`/Health Connect se branche au niveau du repository ; l'export JSON/CSV
se fait à partir des DAO sans toucher au domaine.

## Règles de progression (résumé)
- Force : double progression. Si le cran de la machine dépasse 5 % de la charge, on ajoute d'abord des
  répétitions jusqu'au haut de la fourchette, puis +1 cran et retour en bas de fourchette.
- RPE ≤ 7 et tout réussi → augmente ; 8–9 → maintien ; 10 ou 2 échecs → baisse ; 3 séances dures → allègement.
- Douleur → rien n'augmente + alternatives proposées. RPE non saisi → rien n'augmente.
- Cardio : une seule variable à la fois, en rotation ; trop dur → annulation du dernier changement.
- Semaine allégée seulement si les données le justifient (voir `WorkoutRepository.maybeScheduleDeload`).

## À vérifier toi-même
- `loadStepKg` et `defaultLoadKg` dans `ExerciseCatalog.kt` : les crans réels de TES machines.
- Les charges de départ sont volontairement basses (semaine 1 = calibration).

## Non fait dans ce MVP
- Photos de progression.
- Export/import, sync cloud, Health Connect, montre.
- RPE par série (un RPE par exercice).
- Vidéos intégrées (lien de recherche YouTube uniquement).

App Sport n'est pas un dispositif médical.
