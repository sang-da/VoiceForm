Projet de Questionnaire Vocal (Modèle)

Ce projet est un modèle complet pour créer une application web de collecte de feedback vocal. Il permet aux utilisateurs de s'identifier, de répondre à une série de questions en enregistrant leur voix, et stocke en toute sécurité les fichiers audio et les métadonnées sur Google Drive.

Il inclut également une expérience utilisateur asynchrone (l'envoi se fait en arrière-plan) et une modale de félicitations avec collecte d'email optionnelle (stockée dans un Google Sheet).

Architecture

Le projet est divisé en deux parties :

Frontend (Frontend-Template.html) : Un unique fichier HTML qui contient tout le code client (HTML, CSS Tailwind, et JavaScript). Il gère l'interface utilisateur, l'enregistrement audio dans le navigateur, et la communication avec le backend.

Backend (Backend-Template-Anonyme.gs) : Un script Google Apps Script qui sert d'API sécurisée. Il gère la création de dossiers, la réception des fichiers, et la sauvegarde des données sur Google Drive et Google Sheets.

Installation

Suivez ces étapes pour déployer votre propre version de l'application.

1. Installation du Backend (Google Apps Script)

Le backend est le cœur de l'application. Vous devez le déployer en premier pour obtenir une URL d'API.

Ouvrez le fichier Backend-Template-Anonyme.gs.

Créez votre dossier Google Drive :

Créez un dossier sur votre Google Drive (ex: "MesFeedbacksAudio").

Ouvrez-le et copiez l'ID depuis l'URL (ex: drive.google.com/drive/folders/xxxxxxxxxxxx).

Collez cet ID dans la variable PARENT_FOLDER_ID (ligne 20) du script.

Créez votre Google Sheet (Optionnel) :

Si vous souhaitez collecter les emails, créez un nouveau Google Sheet (ex: "Mailing List").

Copiez l'ID depuis l'URL (ex: docs.google.com/spreadsheets/d/yyyyyyyyyyyy/edit).

Collez cet ID dans la variable MAILING_LIST_SHEET_ID (ligne 26) du script.

Déployez le Script :

Enregistrez le script.

Cliquez sur Déployer (en haut à droite) > Nouveau déploiement.

Cliquez sur l'icône Engrenage (à côté de "Sélectionner le type") et choisissez Application Web.

Pour "Qui y a accès", sélectionnez "Tout le monde". (C'est crucial pour que le frontend puisse l'appeler).

Cliquez sur Déployer.

Autorisez les Permissions :

Google vous demandera d'autoriser le script. Cliquez sur "Examiner les autorisations".

Choisissez votre compte.

Google affichera un avertissement "Google n'a pas validé cette application". Cliquez sur "Paramètres avancés", puis sur "Accéder à [Nom de votre script] (non sécurisé)".

Cliquez sur "Autoriser" pour donner les permissions d'accès à Drive et Sheets.

Copiez l'URL de l'Application Web :

Une fois le déploiement terminé, copiez l'URL de l'application Web (elle se termine par /exec). Gardez-la précieusement.

2. Installation du Frontend (HTML)

Le frontend a besoin de savoir à quelle URL envoyer les données.

Ouvrez le fichier Frontend-Template.html.

Trouvez la section <script id="app-config"> (vers le début du fichier).

Collez votre URL :

Localisez la ligne WEBAPP_URL: "REMPLACEZ_PAR_VOTRE_URL_DE_WEBAPP_GOOGLE_SCRIPT",

Remplacez le placeholder par l'URL de votre application Web (celle que vous avez copiée à l'étape 6 du backend).

Personnalisez (Optionnel) :

Vous pouvez changer les QUESTIONS, le TITLE, et les LOGO_URL dans cette même section APP_CONFIG.

3. C'est Prêt !

Vous pouvez maintenant ouvrir le fichier Frontend-Template.html dans un navigateur. L'application est entièrement fonctionnelle et connectée à votre Google Drive.

Fonctionnement Détaillé

L'utilisateur ouvre le Frontend-Template.html.

Il remplit le formulaire de profil (Identifiant, Contexte, etc.).

Lors de la validation, le frontend appelle l'action ensureUserFolder du backend.

Le backend crée l'arborescence de dossiers (ex: /MonDossier/CHOPS_plus_CHOPi/etudiant/ID_Promo/) et renvoie l'ID unique du dossier final (userFolderDriveId).

Cet ID est stocké localement dans le navigateur.

L'utilisateur enregistre une réponse audio.

Lors de l'envoi (submitOne), la modale se ferme, la carte passe en "Envoi en cours...".

En arrière-plan (runUploadTask), le frontend appelle l'action uploadAudio avec le fichier audio (en Base64) et le userFolderDriveId.

Le backend reçoit le fichier, le décode, et le sauvegarde (Audio + JSON) directement dans le bon dossier, ce qui est très rapide.

Le backend répond ok: true.

Le frontend met à jour la carte en "Répondu" (ou "Échec" en cas d'erreur).

(Fin) Si l'utilisateur remplit le champ email, l'action saveEmail est appelée pour ajouter une ligne au Google Sheet.
