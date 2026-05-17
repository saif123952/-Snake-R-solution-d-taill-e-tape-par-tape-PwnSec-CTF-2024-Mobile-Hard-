# -Snake-R-solution-d-taill-e-tape-par-tape-PwnSec-CTF-2024-Mobile-Hard-

Introduction

Dans ce laboratoire, nous avons réalisé une analyse statique et dynamique d’une application Android vulnérable nommée Snake.apk.
L’objectif principal était de comprendre le fonctionnement interne de l’application, identifier les mécanismes de protection implémentés (anti-root, anti-emulator et anti-Frida), puis contourner ces protections grâce au patching Smali.
Après cette phase de reverse engineering, nous avons exploité une vulnérabilité de désérialisation SnakeYAML afin d’exécuter du code caché dans la classe BigBoss et récupérer le flag final de l’application.
Analyse du MainActivity avec Jadx
Commentaire

Cette étape montre l’analyse statique du fichier MainActivity à l’aide de l’outil Jadx-GUI.
Nous avons identifié la méthode isDeviceRooted() appelée au démarrage de l’application dans onCreate().
Si un environnement rooté est détecté, l’application affiche le message “Root detected! Exiting application.” puis se ferme automatiquement avec System.exit(0).
Cette analyse nous a permis de localiser précisément les protections de sécurité à contourner.
![](https://github.com/user-attachments/assets/eef22e36-67db-4646-9c5a-8876e13926da)

Décompilation de l’APK avec Apktool
Commentaire

Cette capture montre l’utilisation de Apktool pour décompiler l’APK Snake.apk.
L’outil extrait les ressources, le manifeste Android ainsi que les fichiers Smali afin de permettre la modification du code bas niveau de l’application.
Cette étape est essentielle pour effectuer le patching des mécanismes de détection.
![](https://github.com/user-attachments/assets/1ebe5352-09b5-4195-909b-50051bfaaa91)
Analyse de la classe BigBoss
Commentaire

Dans cette étape, nous avons analysé la classe BigBoss.smali.
La classe charge une bibliothèque native avec System.loadLibrary("snake") puis appelle la fonction JNI stringFromJNI().
Nous avons également identifié la chaîne spéciale "Snaaaaaaaaaaaaaake" utilisée comme condition pour déclencher l’exécution du code natif responsable de la génération du flag.
![](https://github.com/user-attachments/assets/888a9313-9732-4467-971b-c75e24c1d7ac)

Recherche des protections Root
Commentaire

Cette capture montre les différentes détections de root présentes dans l’application.
Les vérifications portent sur :

la présence du binaire su
les applications de gestion root
les fichiers système suspects
les shells rootés

Ces protections empêchent l’exécution de l’application sur des appareils compromis ou utilisés pour l’analyse de sécurité.
![](https://github.com/user-attachments/assets/da3cc3c8-e930-490e-a24c-f50bb415e659)

Analyse de la fonction isDeviceRooted()
Commentaire

Cette fonction centralise tous les mécanismes de détection de root.
Plusieurs méthodes sont appelées successivement :

checkForDangerousBinaries()
checkForRootManagementApps()
checkForWritableSystem()
checkForRootShell()

Si une seule vérification retourne true, l’application considère l’appareil comme rooté.
![](https://github.com/user-attachments/assets/a0eb3e78-8861-42cb-a5b4-e3c3533bde0a)

Structure complète de la fonction Root Detection
Commentaire

Cette capture présente le flux logique complet de la méthode isDeviceRooted().
Les instructions conditionnelles if-nez et if-eqz contrôlent le comportement de sécurité de l’application.
Cette analyse nous a permis d’identifier les instructions Smali à modifier pour forcer la fonction à retourner false.
![](https://github.com/user-attachments/assets/5256ef41-35c3-4b6d-b62c-88223be9ce7c)

Patch Smali automatique
Commentaire

Dans cette étape, une commande sed a été utilisée pour modifier automatiquement le code Smali de la fonction isDeviceRooted().
Le retour de la fonction a été forcé à 0x0, ce qui signifie false.
Ainsi, toutes les détections root ont été désactivées.
![](https://github.com/user-attachments/assets/527aafe8-9caf-4049-bcfc-a87d96fcc68c)

Vérification du patch Smali
Commentaire

Cette capture confirme que le patch Smali a été appliqué correctement.
La fonction isDeviceRooted() retourne désormais directement false grâce à :

const/4 p0, 0x0
return p0

Cette modification empêche définitivement la fermeture automatique de l’application.
![](https://github.com/user-attachments/assets/4d306547-a286-4830-a11b-ce4928c42f4b)

Recherche des détections Emulator
Commentaire

Une recherche a été effectuée afin d’identifier les mécanismes de détection d’émulateur dans les fichiers Smali.
Les mots-clés recherchés incluent :

emulator
generic
goldfish

Cette phase permet d’identifier rapidement les protections supplémentaires présentes dans l’application.
![](https://github.com/user-attachments/assets/c8dc9aab-8408-4c11-b729-511b1f567dfa)

Résultat de la recherche Smali
Commentaire

Les résultats affichent plusieurs occurrences contenant des indicateurs liés aux émulateurs Android.
Ces informations facilitent la localisation des protections anti-emulator dans le code bas niveau afin de les neutraliser si nécessaire.
![](https://github.com/user-attachments/assets/0b9592e4-8831-4a1c-97f3-be833ac7cbbc)

Vérification du flux MainActivity
Commentaire

Cette capture montre l’appel à isDeviceRooted() directement depuis onCreate().
Le flux d’exécution indique clairement que l’application quitte immédiatement si la détection retourne true.
Cette étape confirme que le patch effectué précédemment est indispensable pour continuer l’analyse.
![](https://github.com/user-attachments/assets/a6b3f153-d07a-4357-b83b-ad1cecdeba73)

Recompilation de l’APK
Commentaire

Après les modifications Smali, l’application a été recompilée avec Apktool afin de générer un nouvel APK nommé snake_patched.apk.
Cette étape reconstruit entièrement l’application avec les protections désactivées.
![](https://github.com/user-attachments/assets/48fa2511-77c6-4c00-b08f-2f6c65a0d8f2)

Génération du Keystore
Commentaire

Cette étape montre la création d’un keystore Java utilisé pour signer l’APK modifié.
La signature numérique est obligatoire sur Android pour pouvoir installer une application recompilée.
![](https://github.com/user-attachments/assets/69b1e946-0329-43b4-9aa8-94a8fd943a23)

Signature de l’APK
Commentaire

La commande jarsigner a été utilisée pour signer l’APK patché avec le keystore précédemment généré.
Cette opération garantit l’intégrité de l’application avant son installation sur l’émulateur Android.
![](https://github.com/user-attachments/assets/993c154e-3219-4351-b213-8f3e0cb2e166)

Processus complet de signature
Commentaire

Cette capture montre le processus de signature des différents composants internes de l’APK :

AndroidManifest.xml
classes.dex
resources.arsc
fichiers XML

La signature s’est terminée avec succès.
![](https://github.com/user-attachments/assets/876a0d12-8ccc-4511-8dea-f50312d37f1f)

Alignement et installation de l’APK
Commentaire

L’outil zipalign a été utilisé pour optimiser l’APK avant la signature finale avec apksigner.
L’application patchée a ensuite été installée via adb install.
![](https://github.com/user-attachments/assets/9dcf2e3b-195c-4828-846f-ee4d9aa3e4b1)

Erreur de signature
Commentaire

Une erreur INSTALL_FAILED_UPDATE_INCOMPATIBLE est apparue car l’application originale possédait une signature différente de celle de l’APK patché.
Android refuse les mises à jour dont la signature ne correspond pas à l’application déjà installée.
![](https://github.com/user-attachments/assets/0a75eddb-5c15-40cc-86a7-6e78bc60eb18)

Modification du chemin YAML
Commentaire

Cette étape montre la modification du chemin du fichier YAML dans le code Smali afin de faciliter le chargement du payload malveillant depuis un emplacement spécifique.
![](https://github.com/user-attachments/assets/7655e0e0-fbd8-4d9e-8fdd-3e93aeb6571c)

Installation finale réussie
Commentaire

Après désinstallation de l’ancienne version et réinstallation de l’APK patché signé correctement, l’installation s’est terminée avec succès.
L’application était alors prête pour l’exploitation de la vulnérabilité SnakeYAML et la récupération du flag.
![](https://github.com/user-attachments/assets/786eb143-cf21-4b0a-86b0-93702a01547e)

Conclusion

Ce laboratoire nous a permis de comprendre les principales techniques de reverse engineering Android ainsi que les mécanismes de protection utilisés dans les applications mobiles modernes.
Nous avons réalisé une analyse statique avec Jadx, effectué du patching Smali pour contourner les protections anti-root et anti-emulator, puis recompilé et signé l’APK modifié.
Enfin, grâce à l’exploitation d’une vulnérabilité de désérialisation SnakeYAML (CVE-2022-1471), nous avons réussi à déclencher l’exécution du code natif caché dans la classe BigBoss et récupérer le flag final de l’application.
