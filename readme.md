# **Projet : Questionnaire Vocal Sécurisé avec Transcription IA (Gemini)**

Ce projet est une application web complète de "voiceform" (questionnaire vocal). Elle permet de collecter des réponses audio de la part des utilisateurs, de les stocker de manière sécurisée dans Google Drive, et d'utiliser un "worker" asynchrone pour transcrire automatiquement l'audio en texte grâce à l'API Gemini.

## **Architecture**

Le projet est divisé en trois parties distinctes pour maximiser la vitesse côté utilisateur et la sécurité :

1. **Frontend (Frontend-Template.html)**  
   * Un simple fichier HTML/CSS/JS (vanilla) qui ne nécessite aucun serveur.  
   * Il gère l'interface utilisateur, l'enregistrement audio, et l'envoi asynchrone (non bloquant) des données.  
2. **Backend A : Le Réceptionniste (Backend-Receiver-A-Template.gs)**  
   * Un script Google Apps Script déployé comme **Application Web**.  
   * Son unique rôle est d'être *extrêmement rapide*. Il reçoit la requête (métadonnées \+ fichier audio), crée les dossiers nécessaires sur Google Drive, sauvegarde les fichiers, et répond "OK" immédiatement (en \~1 seconde).  
   * Il gère aussi la collecte optionnelle d'emails pour la mailing list.  
3. **Backend B : Le Travailleur (Backend-Worker-B-Template.gs)**  
   * Un second script Google Apps Script qui n'est **pas déployé**.  
   * Il s'exécute automatiquement toutes les 10 minutes grâce à un **déclencheur (trigger)**.  
   * Son rôle est de scanner le Google Drive à la recherche de fichiers audio "non traités", de les envoyer à l'API Gemini pour transcription, de sauvegarder le fichier .txt, et d'envoyer un email de notification.

Cette architecture garantit que l'utilisateur n'attend jamais la fin de la transcription (qui peut être longue).

## **Fonctionnalités**

* **Enregistrement Vocal :** Capture audio directement depuis le navigateur.  
* **Stockage Sécurisé :** Crée un dossier par utilisateur sur Google Drive.  
* **Envoi Asynchrone :** L'interface utilisateur ne se bloque jamais. L'utilisateur peut répondre à toutes les questions rapidement.  
* **Transcription IA :** Utilise gemini-2.5-flash-preview-09-2025 pour une transcription rapide et précise.  
* **Notifications :** Envoie un email à l'administrateur avec la transcription et un lien vers le dossier de l'utilisateur.  
* **Sécurisation des Clés :** **Aucune clé API, ID de dossier ou email** n'est écrit en dur dans le code. Tout est stocké dans les **Propriétés du Script** de Google Apps Script pour une sécurité maximale.

## **Guide d'Installation (pour les étudiants)**

Suivez ces 4 parties dans l'ordre pour faire fonctionner le projet avec votre propre compte Google.

### **Partie 1 : Prérequis (Google Drive & API)**

Vous avez besoin de 4 "clés" avant de commencer.

1. **Dossier Parent (Drive) :** Créez un dossier principal sur votre Google Drive (ex: "Mes Reponses Audio"). Ouvrez-le et copiez l'ID depuis l'URL.  
   * https://drive.google.com/drive/folders/**\[...VOTRE\_ID\_PARENT...\]**  
   * Notez cet ID. C'est votre PARENT\_FOLDER\_ID.  
2. **Dossier Transcriptions (Drive) :** Créez un *autre* dossier (ex: "Mes Transcriptions").  
   * Notez son ID. C'est votre TRANSCRIPTS\_FOLDER\_ID.  
3. **Mailing List (Sheets) :** Créez un nouveau Google Sheet (ex: "Ma Mailing List").  
   * Notez son ID. C'est votre MAILING\_LIST\_SHEET\_ID.  
4. **Clé API Gemini :**  
   * Allez sur [Google AI Studio (anciennement MakerSuite)](https://aistudio.google.com/).  
   * Connectez-vous et cliquez sur "**Get API key**" \-\> "**Create API key in new project**".  
   * Copiez la longue chaîne de caractères. C'est votre GEMINI\_API\_KEY.

### **Partie 2 : Configurer le Backend A (Réceptionniste)**

1. Allez sur [script.google.com](https://script.google.com/) et créez un **Nouveau projet**.  
2. Copiez-collez le contenu de Backend-Receiver-A-Template.gs dans l'éditeur.  
3. À gauche, cliquez sur **Paramètres du projet (⚙️)**.  
4. Faites défiler jusqu'à **"Propriétés du script"** et ajoutez 3 propriétés :  
   * **Nom :** PARENT\_FOLDER\_ID | **Valeur :** \[Votre ID de dossier parent de l'étape 1.1\]  
   * **Nom :** MAILING\_LIST\_SHEET\_ID | **Valeur :** \[Votre ID de Google Sheet de l'étape 1.3\]  
   * **Nom :** APP\_ID | **Valeur :** mon-app-unique (ou un autre nom unique de votre choix)  
5. Enregistrez les propriétés.  
6. Cliquez sur **Déployer** \> **Nouveau déploiement**.  
7. Type de déploiement (⚙️) : **Application Web**.  
8. Configuration :  
   * **Description :** Mon Réceptionniste Audio  
   * **Exécuter en tant que :** Moi  
   * **Qui y a accès :** **Tout le monde** (C'est crucial pour que le HTML puisse l'appeler)  
9. Cliquez sur **Déployer**. Autorisez l'accès à vos services Google.  
10. **IMPORTANT :** Copiez l'**URL de l'application Web**. Vous en aurez besoin.

### **Partie 3 : Configurer le Backend B (Travailleur)**

1. Créez un **second projet** Google Apps Script.  
2. Copiez-collez le contenu de Backend-Worker-B-Template.gs dans l'éditeur.  
3. Allez dans **Paramètres du projet (⚙️)** \> **Propriétés du script** et ajoutez 4 propriétés :  
   * **Nom :** GEMINI\_API\_KEY | **Valeur :** \[Votre clé API Gemini de l'étape 1.4\]  
   * **Nom :** NOTIFY\_EMAIL | **Valeur :** \[Votre email où recevoir les notifications\]  
   * **Nom :** PARENT\_FOLDER\_ID | **Valeur :** \[Votre ID de dossier parent (le même qu'en 2.4)\]  
   * **Nom :** TRANSCRIPTS\_FOLDER\_ID | **Valeur :** \[Votre ID de dossier transcriptions de l'étape 1.2\]  
4. Enregistrez les propriétés.  
5. Dans l'éditeur, sélectionnez la fonction testPermissions\_ScriptB dans le menu déroulant et cliquez sur **Exécuter**. Autorisez l'accès (cela vérifie que toutes vos clés sont correctes).  
6. Sélectionnez la fonction installTrigger\_ScriptB et cliquez sur **Exécuter**. Cela démarre le "worker" (il s'exécutera toutes les 10 minutes).

### **Partie 4 : Configurer le Frontend**

1. Ouvrez le fichier Frontend-Template.html dans un éditeur de code.  
2. Trouvez la section window.APP\_CONFIG.  
3. Modifiez ces deux lignes :  
   * WEBAPP\_URL: : Collez l'**URL de votre application web** (obtenue à l'étape 2.10).  
   * APP\_ID: : Mettez le même ID unique que celui de l'étape 2.4 (ex: mon-app-unique).  
4. (Optionnel) Personnalisez les questions dans window.APP\_CONFIG.QUESTIONS ou les logos dans window.APP\_CONFIG.BRAND.  
5. Enregistrez le fichier.

Vous avez terminé \! Vous pouvez maintenant ouvrir le fichier Frontend-Template.html dans votre navigateur pour tester le projet.