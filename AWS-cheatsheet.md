# AWS Configuration Cheat Sheet

**Note :** Ce document n'est pas destiné à remplacer les vidéos, mais à servir de feuille de triche pour les étudiants qui souhaitent rapidement passer en revue les étapes de configuration AWS ou voir facilement s'ils ont manqué une étape. Il aidera également à naviguer à travers les changements de l'interface utilisateur AWS depuis l'enregistrement du cours.

---

## Docker Compose Config Update

1. Renommez le fichier `docker-compose.yml` de développement en `docker-compose-dev.yml`.
2. Créez un fichier `docker-compose.yml` de production à la racine du projet et collez ce qui suit :

    ```yaml
    version: '3'
    services:
      web:
        build:
          context: .
          dockerfile: Dockerfile
        ports:
          - '80:80'
    ```

---

## Create EC2 IAM Instance Profile

1. Accédez à la **AWS Management Console**.
2. Recherchez **IAM** et cliquez sur le service IAM.
3. Cliquez sur **Roles** sous **Access Management** dans la barre latérale gauche.
4. Cliquez sur le bouton **Create role**.
5. Sélectionnez **AWS Service** sous **Trusted entity type**. Ensuite, sélectionnez **EC2** sous **common use cases**.
6. Recherchez **AWSElasticBeanstalk** et sélectionnez les politiques **AWSElasticBeanstalkWebTier**, **AWSElasticBeanstalkWorkerTier** et **AWSElasticBeanstalkMulticontainerDocker**. Cliquez sur le bouton **Next**.
7. Donnez au rôle le nom **aws-elasticbeanstalk-ec2-role**.
8. Cliquez sur le bouton **Create role**.

---

## Create Elastic Beanstalk Environment

1. Accédez à la **AWS Management Console**.
2. Recherchez **Elastic Beanstalk** et cliquez sur le service Elastic Beanstalk.
3. Si vous n'avez jamais utilisé Elastic Beanstalk auparavant, vous verrez une page de présentation. Cliquez sur le bouton **Create Application**. Si vous avez déjà créé des environnements et des applications Elastic Beanstalk, vous serez directement redirigé vers le tableau de bord Elastic Beanstalk. Dans ce cas, cliquez sur le bouton **Create environment**. Vous passerez alors par un flux de 6 étapes.
4. Vous devrez fournir un **Application name**, qui remplira automatiquement un **Environment Name**.
5. Faites défiler vers le bas pour trouver la section **Platform**. Vous devrez sélectionner la plateforme **Docker**. Cela sélectionnera automatiquement plusieurs options par défaut. Changez la branche de la plateforme en **Docker running on 64bit Amazon Linux 2**. La nouvelle branche de 2023 a actuellement des problèmes avec les déploiements de conteneurs uniques.
6. Faites défiler vers le bas jusqu'à la section **Presets** et assurez-vous que **free tier eligible** est sélectionné.
7. Cliquez sur le bouton **Next** pour passer à l'étape #2.
8. Vous serez dirigé vers un formulaire de configuration d'accès au service.

   Sélectionnez **Create and use new service role** et nommez-le **aws-elasticbeanstalk-service-role**. Vous devrez ensuite définir le **EC2 instance profile** sur le rôle **aws-elasticbeanstalk-ec2-role** créé précédemment (cela sera probablement pré-rempli pour vous).
   
9. Cliquez sur le bouton **Skip to Review** car les étapes 3 à 6 ne sont pas applicables.
10. Cliquez sur le bouton **Submit** et attendez que votre nouvelle application et environnement Elastic Beanstalk soient créés et lancés.
11. Cliquez sur le lien sous la coche dans **Domain**. Cela devrait ouvrir l'application dans votre navigateur et afficher un message de félicitations.

---

## Update Object Ownership of S3 Bucket

1. Accédez à la **AWS Management Console**.
2. Recherchez **S3** et cliquez sur le service S3.
3. Trouvez et cliquez sur le bucket **elasticbeanstalk** qui a été automatiquement créé avec votre environnement.
4. Cliquez sur l'onglet **Permissions**.
5. Trouvez **Object Ownership** et cliquez sur **Edit**.
6. Changez de **ACLs disabled** à **ACLs enabled**. Changez **Bucket owner Preferred** à **Object Writer**. Cochez la case reconnaissant l'avertissement.
7. Cliquez sur **Save changes**.

---

## Add AWS Configuration Details to .travis.yml File's Deploy Script

1. Définissez la région. Le code de la région peut être trouvé en cliquant sur la région dans la barre d'outils à côté de votre nom d'utilisateur.

   Par exemple : `'us-east-1'`

2. **app** doit être défini sur le **Application Name** (étape #4 de la configuration initiale ci-dessus).

   Par exemple : `'docker'`

3. **env** doit être défini sur la version en minuscules du nom de votre environnement Beanstalk.

   Par exemple : `'docker-env'`

4. Définissez **bucket_name**. Cela peut être trouvé en recherchant le service S3. Cliquez sur le lien pour le bucket **elasticbeanstalk** qui correspond à votre code de région et copiez le nom.

   Par exemple : `'elasticbeanstalk-us-east-1-923445599289'`

5. Définissez **bucket_path** sur `'docker'`.
6. Définissez **access_key_id** sur `$AWS_ACCESS_KEY`.
7. Définissez **secret_access_key** sur `$AWS_SECRET_KEY`.

---

## Create an IAM User

1. Recherchez le service **IAM Security, Identity & Compliance**.
2. Cliquez sur **Create Individual IAM Users** et cliquez sur **Manage Users**.
3. Cliquez sur **Add User**.
4. Entrez n'importe quel nom que vous souhaitez dans le champ **User Name**.

   Par exemple : `docker-react-travis-ci`

5. Cliquez sur **Next**.
6. Cliquez sur **Attach Policies Directly**.
7. Recherchez **beanstalk**.
8. Cochez la case à côté de **AdministratorAccess-AWSElasticBeanstalk**.
9. Cliquez sur **Next**.
10. Cliquez sur **Create user**.
11. Sélectionnez l'utilisateur IAM que vous venez de créer dans la liste des utilisateurs.
12. Cliquez sur **Security Credentials**.
13. Faites défiler vers le bas pour trouver **Access Keys**.
14. Cliquez sur **Create access key**.
15. Sélectionnez **Command Line Interface (CLI)**.
16. Faites défiler vers le bas et cochez la case **I understand...** puis cliquez sur **Next**.

   Copiez et/ou téléchargez l'**Access Key ID** et l'**Secret Access Key** pour les utiliser dans la configuration des variables Travis.

---

## Travis Variable Setup

1. Allez sur votre tableau de bord Travis et trouvez le projet de dépôt pour l'application sur laquelle nous travaillons.
2. Sur la page du dépôt, cliquez sur **More Options** puis sur **Settings**.
3. Créez une variable **AWS_ACCESS_KEY** et collez votre clé d'accès IAM de l'étape #13 ci-dessus.
4. Créez une variable **AWS_SECRET_KEY** et collez votre clé secrète IAM de l'étape #13 ci-dessus.

---

## Deploying App

1. Apportez une petite modification à votre fichier `src/App.js` dans le texte de salutation.
2. Dans la racine du projet, exécutez dans votre terminal :

    ```bash
    git add .
    git commit -m "testing deployment"
    git push origin main
    ```

3. Allez sur votre tableau de bord Travis et vérifiez le statut de votre build.
4. Le statut devrait éventuellement retourner un coche verte et afficher **"build passing"**.
5. Allez sur votre application AWS Elastic Beanstalk.
6. Cela devrait dire **"Elastic Beanstalk is updating your environment"**.
7. Cela devrait éventuellement montrer une coche verte sous **Health**. Vous pourrez maintenant accéder à votre application à l'URL externe fournie sous le nom de l'environnement.

---