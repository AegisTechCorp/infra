Whitepaper : Architecture et Sécurité d'Aegis
Le Coffre-Fort Médical à "Connaissance Zéro"
1. Résumé Exécutif
La donnée de santé est la donnée la plus intime et la plus critique d'un citoyen. Pourtant, aujourd'hui, elle est éparpillée (laboratoires, hôpitaux, spécialistes) ou stockée sur des plateformes centralisées qui possèdent les clés de lecture.
Aegis introduit une nouvelle approche : une application où le citoyen centralise son dossier médical complet, mais où il est le seul détenteur de la clé d'accès. Ni Aegis, ni l'hébergeur cloud, ni un pirate interceptant le flux ne peuvent lire le contenu des dossiers.
2. Le Problème : La "Transparence" des Données Actuelles
Actuellement, lorsque vous stockez un document sur un Cloud classique (type Google Drive ou Dropbox) ou sur certaines plateformes médicales :
Le transfert est sécurisé (HTTPS).
Le stockage est chiffré sur le disque dur du serveur.
La faille : L'hébergeur possède la clé de déchiffrement. Si un administrateur système est malveillant ou si la base de données est compromise, les dossiers sont lisibles en clair.
3. Solution : Chiffrement côté client - Schéma du chiffrement - Preuve que le serveur ne possède jamais la clé

Le modèle cryptographique d’Aegis garantit que la clé permettant de déchiffrer les données n’existe qu’au seul endroit contrôlé par l’utilisateur. Le serveur ne reçoit jamais le mot de passe, ni la Master Key (MK), ni aucune clé en clair.


Création / connexion
L’utilisateur choisit un mot de passe maître (jamais transmis).
L’app dérive localement la Master Key (MK) via Argon2 ou PBKDF2 + salt local.


Chiffrement par fichier
Pour chaque fichier : l’app génère une File Key (FK).
Le fichier est chiffré localement avec la FK (AES-GCM ou ChaCha20-Poly1305).
La FK est chiffrée localement par la MK.
L’app envoie au serveur : { ciphertext, FK_chiffrée, metadata_min }.

Stockage
Le serveur stocke uniquement le ciphertext et la FK_chiffrée.
Aucune clé en clair n’est présente sur le serveur.

Consultation  / partage
Le serveur renvoie ciphertext + FK_chiffrée.
L’app local déchiffre FK_chiffrée avec MK → obtient FK → déchiffre le fichier.
Pour un partage temporaire, l’app génère une clé de lecture temporaire locale pour l’invité.

Qu'est-ce qu'on protège ?
La protection s'applique à tous les types de données, sans exception :
 Documents : PDF d'analyses sanguines, comptes-rendus opératoires.
 Imagerie : Radios, IRM, photos dermatologiques.
 Métadonnées sensibles : Liste des allergies, historique des prescriptions.
 Support Client : Les échanges avec le support technique ne contiennent jamais de données médicales en clair.
4. Parcours Utilisateur & Exemples Concrets
Pour comprendre la sécurité, suivons le parcours de Sophie, une utilisatrice d'Aegis.
Scénario A : Le dépôt d'une ordonnance (l'upload)
Sophie sort de chez son cardiologue avec une ordonnance papier.
Elle ouvre Aegis et prend l'ordonnance en photo.
Sur son téléphone (avant tout envoi) : L'application génère une clé unique aléatoire pour ce fichier. Le fichier est transformé en une liste de caractères illisibles (chiffré).
L'envoi : C'est cette liste chiffrée qui est envoyée vers les serveurs d'Aegis.
Stockage : Aegis reçoit le fichier illisible et le stocke. Aegis ne sait pas que c'est une ordonnance de cardiologie, Aegis voit juste un fichier binaire nommé XJ9-SafeBlob.
Scénario B : Le Bug (Support Client Sécurisé)
Sophie a un problème : "Je n'arrive pas à ouvrir mon scan d'IRM". Elle contacte le support.
Le canal de chat est chiffré.
Le technicien Aegis voit le ticket. Il voit les logs techniques ("Erreur timeout upload").
Ce qu'il ne voit pas : Il ne peut pas voir l'IRM. Il ne peut pas voir le nom du médecin. Il ne peut pas "prendre la main" pour voir le contenu déchiffré.
La résolution se fait à l'aveugle, garantissant qu'aucun employé d'Aegis n'accède à l'intimité médicale.
5. Architecture Technique Simplifiée
Qui a accès à quoi ?
Acteur
Données Chiffrées (Le Coffre)
Clé de Déchiffrement (La Clé)
Données en Clair (Le Contenu)
Utilisateur (Sophie)
✅ Oui
✅ Oui (Dérivée du mot de passe)
✅ Oui
Serveurs Aegis
✅ Oui
❌ Non
❌ Non
Hacker (Attaquant)
✅ Oui (S'il vole la BDD)
❌ Non
❌ Impossible



6. Analyse des Risques et Failles Potentielles (Modèle de Menace)
Aucun système n'est infaillible. Voici les vecteurs d'attaque et nos réponses :
1. La faille du "Post-it" (Facteur Humain)
Risque : L'utilisateur note son mot de passe maître sur un post-it ou utilise "123456".
Réponse : Aegis impose des mots de passe complexes et encourage l'usage de la double authentification, réduisant la saisie fréquente du mot de passe maître. Une double authentification sera également mise en place.
2. La compromission du terminal (Le téléphone piraté)
Risque : Le téléphone de Sophie est infecté par un logiciel espion qui enregistre tout ce qui s'affiche à l'écran.
Réponse : C'est la limite du chiffrement côté client. Si l'appareil qui déchiffre est vérolé, la donnée est exposée. Aegis implémente des protections anti-screenshot et détecte les appareils "rootés" ou non sécurisés pour bloquer l'application préventivement.
3. Les Métadonnées
Risque : Même si le contenu est chiffré, savoir que Sophie a uploadé un fichier de 5 Mo tous les jours à 8h00 peut donner des indices.
Réponse : Nous limitons la collecte de métadonnées au strict nécessaire pour le fonctionnement technique (taille du stockage utilisé, date de dernière connexion).



8. Conclusion
Aegis n'est pas simplement une "app de stockage". C'est un coffre-fort numérique dont nous avons jeté le double des clés. En acceptant la responsabilité de gérer sa propre sécurité (via son mot de passe maître), l'utilisateur regagne la propriété absolue de sa santé. Dans un monde où la donnée médicale vaut de l'or pour les hackers, Aegis choisit de rendre cette donnée inexploitable pour quiconque d'autre que le patient.


