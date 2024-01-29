# Importation Certificat SSL/TLS Sur Kubernetes

TOS sur l'importation d'un certificat SSL/TLS existant dans un cluster Kubernetes, ici l'objectif n'est pas d'apprendre à générer un certificat SSL/TLS, mais de savoir comment importer un certificat que l'on possède déjà.

Si vous souhaitez générer des certificats Let's Encrypt SSL/TLS voici deux méthodes une sous Windows et une sous Linux :

* [Win-ACME (Windows)](https://www.it-connect.fr/microsoft-exchange-server-2019-certificat-ssl-gratuit-avec-lets-encrypt/#III_Demander_un_certificat_SSL_avec_Win-ACME)
* [Cerbot (Linux)](https://antoinelounis.com/informatique/securite/creer-certificat-ssl-certbot-https/)

## Création d'un secret Kubernetes

Après avoir généré votre certificat SSL et sa clé privé, vous pouvez l'importer en créant une ressource `Secret`. Il s'agit d'un objet de kubernetes permettant de stocker des informations snesibles comme les mots de passe, les clés ssh ou les jetons d'authentification.

Cette ressource peut se créer soit en ligne de commande `$ kubectl create secret tls mon-cert-tls --key tls.key --cert tls.crt` que vous devez ensuite éditer pour ajouter les chaînes du certificat et de la clé, en soit directement avec un manifeste `mon-cert-tls.yaml`.

Dans les 2 cas, il faudra ajouter les chaînes du certificat et de la clé __encodé en base64__, en voici un exemple.

> [mon-cert-tls.yaml](mon-cert-tls.yaml)
>
> ```yaml
> apiVersion: v1
> kind: Secret
> type: kubernetes.io/tls
> metadata:
>   name: mon-cert-tls
> data:
>   tls.crt: LS0tLS1CRUdJTiBDRV...LS0tGc== # Base64-Encode
>   tls.key: LS0tLS1CRUdJTiBSU0...LS0tGc== # Base64-Encode
> ```

Voilà votre certificat est importé dans kubernetes dans une ressource `Secret`. :smiley:

## Utiliser un certificat par défaut dans Traefik

Voici un cas intéressant sur la façon d'utiliser un secret contenant un certificat SSL/TLS. Il faut avoir déjà installé Traefik et ces CRDs pour que cela fonctionne.

Ici nous allons faire un sorte que tout les `IngressRoutes` de Traefik puissent utiliser par défaut un autre certificat que celui déjà présent dans l'image de Traefik.

La ressource utilisé ici est une ressource `TLSStore` qui fournit par Traefik afin de définir le nouveau certificat par défaut.

Voici un manifeste pour utiliser le certificat importé a tout vos `IngressRoutes`.

> [default-tls-store.yaml](default-tls-store.yaml)
>
> ```yaml
> apiVersion: traefik.containo.us/v1alpha1
> kind: TLSStore
> metadata:
>  name: default
>  namespace: default
>
>spec:
>  defaultCertificate:
>    secretName: mon-cert-tls
>```

Bravo :clap:, vous avez maintenant à votre disposition un Traefik qui attribuera votre certificat à toutes les `IngressRoutes` qui n'ont pas de certificats paramétrés.
