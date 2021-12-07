# sdfsfdsfsdfsdfsfdfsdfsdfsdf

name: deploy-preprod
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  build:
    runs-on: self-hosted
    container:
      image: node:14.17.6-buster
      options: --user 1010
    steps:
      - uses: actions/checkout@v2
      - run: npm i
      - run: NODE_ENV=staging npm run build
#     - run: npm run test
#       env:
#         CI: true

  push_to_registry:
    name: Push Docker image to Azure Registry
    needs: build
    runs-on: self-hosted
    steps:
      - name: check out the repo
        uses: actions/checkout@v2

      - name: set up Docker builder
        uses: docker/setup-buildx-action@v1

      - name: log into Azure Container Registry
        uses: docker/login-action@v1
        with:
          registry: deadpoolcr.azurecr.io
          username: ${{ secrets.OWNER }}
          password: ${{ secrets.TOKEN }}
          
      - name: Pre Build
        run: "docker system prune -af"

      - name: push to Azure Container Registry
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          build-args: |
            ENV=staging
          tags: |
            deadpoolcr.azurecr.io/frchallenges-backend-stage:${{ github.sha }}
            deadpoolcr.azurecr.io/frchallenges-backend-stage:latest
  helm:
      runs-on: ubuntu-20.04
      needs: push_to_registry
      timeout-minutes: 2
      steps:
        - uses: actions/checkout@v2
        - uses: azure/k8s-bake@v1
          with:
            renderEngine: 'helm'
            helmChart: './helm/'
            overrides: |
              image.tag:${{ github.sha }}
              image.repo:frchallenges-backend-stage
              project:frc
              app:back
              containerport:1337
              host:pre.api.france-challenges.com
              environment:staging
            helm-version: 'latest'
          id: bake
        - uses: actions/upload-artifact@main
          with:
            name: helm_artifact
            path: ${{ steps.bake.outputs.manifestsBundle }}
  deploy:
      runs-on: self-hosted
      needs: helm
      steps:
        - uses: Azure/k8s-set-context@v1
          with:
            kubeconfig: ${{ secrets.KUBE_PREPROD }}
        - uses: actions/download-artifact@main
          with:
            name: helm_artifact
            path: k8s
        - uses: azure/k8s-create-secret@v1
          with:
            namespace: 'frc'
            secret-type: 'generic'
            arguments:  |
              --from-literal=ELEVE_ID=${{ secrets.PREPROD_ELEVE_ID }}
              --from-literal=ENABLE_SENTRY=${{ secrets.PREPROD_ENABLE_SENTRY }}
              --from-literal=NODE_ENV=${{ secrets.PREPROD_NODE_ENV }}
              --from-literal=PROF_ID=${{ secrets.PREPROD_PROF_ID }}
              --from-literal=SENDING_BLUE_API_KEY=${{ secrets.PREPROD_SENDING_BLUE_API_KEY }}
              --from-literal=SENTRY_DSN=${{ secrets.PREPROD_SENTRY_DSN }}
              --from-literal=SENTRY_ENABLE_MESSAGE=${{ secrets.PREPROD_SENTRY_ENABLE_MESSAGE }}
              --from-literal=ZOHO_CLIENT_ID=${{ secrets.PREPROD_ZOHO_CLIENT_ID }}
              --from-literal=ZOHO_MODULES_REFRESH_TOKEN=${{ secrets.PREPROD_ZOHO_MODULES_REFRESH_TOKEN }}
              --from-literal=ZOHO_SECRET=${{ secrets.PREPROD_ZOHO_SECRET }}
              --from-literal=CHRONOPOST_ID=${{ secrets.PREPROD_CHRONOPOST_ID }}
              --from-literal=CHRONOPOST_MDP=${{ secrets.PREPROD_CHRONOPOST_MDP }}
              --from-literal=SHERLOCKS_MERCHANT_ID=${{ secrets.PREPROD_SHERLOCKS_MERCHANT_ID }}
              --from-literal=SHERLOCKS_SECRET_KEY=${{ secrets.PREPROD_SHERLOCKS_SECRET_KEY }}
              --from-literal=SHERLOCKS_PAYMENT_URL=${{ secrets.PREPROD_SHERLOCKS_PAYMENT_URL }}
              --from-literal=SHERLOCKS_KEY_VERSION=${{ secrets.PREPROD_SHERLOCKS_KEY_VERSION }}
              --from-literal=URL_FRONT=${{ secrets.PREPROD_URL_FRONT }}
              --from-literal=URL_BACK=${{ secrets.PREPROD_URL_BACK }}
            secret-name: sec
        - uses: Azure/k8s-deploy@v1.4
          with:
            namespace: 'frc'
            manifests: k8s/*
            kubectl-version: 'latest'
            
            
            
            
   Les accès GoDaddy:
Login: tarak.h
Password: ht-pixi-789
Use this link to accès tenantive dns:
https://click-email.godaddy.com/7lcbYomrlmg1IXG0Flvqjn/?currencyId=USD&eid=ocp.email.transactional/3111.LayoutSimple/Rebrand/Button_Arrow.link.click&marketId=en-GB&redir=https%3A%2F%2Faccount.godaddy.com%2Faccess%3Fisc%3Dgdbb3111%26utm_source%3Dgdocp%26utm_medium%3Demail%26utm_campaign%3Den-GB_other_email-nonrevenue_base_gd%26utm_content%3D200507_3111_Engagement_Other_Account_Account-Notification_gdbb3111_7lcbYomrlmg1IXG0Flvqjn









OVH GMAIL ACCOUNT
L'adresse gmail du compte OVH est la suivante:
-Email: piximind@gmail.com
-Mot de passe: ht-pixi-789
==========================================================================================
ovh
user: ef15714-ovh
pass: @@PixiServer

===========================================================================================

We-team
PARAMETRES D'ACCES:
IP address : 213.32.89.229
DNS name : vps-e85d83e3.vps.ovh.net
Username : ubuntu
MDP : BMajHCkwEAmz

==========================================================================================

PIXI-infras

Votre VPS vient d'être installé sous le système d'exploitation / distribution
Ubuntu 20.04


PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 193.70.113.133

Le nom du VPS est : vps-04fd136d.vps.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :      yz2QcmMdkM5D

============================================================================================

Stage
PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 51.210.255.56
Le nom du VPS est : vps-e47f1ce1.vps.ovh.net
Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :   8yAAS4wm8fhY

================================================================================================

FRC Backup
The new server details are as follows:
L'adresse IPv4 du VPS est : 51.38.226.252
Le nom du VPS est : vps-837cf745.vps.ovh.net
Nom d'utilisateur : ubuntu
Mot de passe :    U8kdztNuUsFJ

=====================================================================================================

browserstack
J'ai donc changé les accès du compte principale comme suit:
Email: piximind@gmail.com
Password: @@Pixi-Stack

======================================================================================================

Pre-PROD SPVIE
ACCESS PARAMETERS:
The IPv4 address of the VPS is: 51.75.251.125
The name of the VPS is: vps-d061a8e7.vps.ovh.net
Username: ubuntu
Password: fme3wqGJyX4N

========================================================================================================

France Challenge PROD
Host IP: 40.89.135.218
Username: Pomelo
MDP: PomeloFRC@123
Ports: 22, 80, 443 (open)

VM type:  B2ms
vCPU: 2
RAM: 8GB
Data disk: 4
Max IOPS: 1920
Temp storage: 16GB

===================================================================================================

SMTP DEV
MAIL_MAILER=smtp
MAIL_HOST=ssl0.ovh.net
MAIL_PORT=587
MAIL_USERNAME=smtp@piximind.com
MAIL_PASSWORD=smtp@smtp
MAIL_ENCRYPTION=tls

========================================================================================================

pre prod Xefi.
Kindly find the details below for the pre prod server of Xefi.
ACCESS PARAMETERS:
The IPv4 address of the VPS is: 5.196.224.137
The name of the VPS is: vps-fe025e5a.vps.ovh.net
The following administrator account has been configured on the VPS:
Username: ubuntu
Password: fXQ7nWt5wECU

===========================================================================================================

OYO 
Virtual Machine:
Details of the VM:
Public IP address : 40.68.216.164
DNS name: fruitdor-pomelo.westeurope.cloudapp.azure.com
ssh Username: Fruidor
MDP: Pomelo@fruidor

============================================================================================================

GT supply
ssh Pomelo@40.89.147.67
Pwd: PomeloGtSupply@123

===========================================================================================================

Frane challenge --PreProd
PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 141.94.16.106
Le nom du VPS est : vps-480e92cd.vps.ovh.net
Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :   raPjjY6mSqRB

==============================================================================================================

DEV-France Challenge VM
Please find the details of France Challenge VM:

PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 51.75.251.125

Le nom du VPS est : vps-d061a8e7.vps.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :      7ytEmHJ8z3Fv

=================================================================================================================

Sa-petite-lettre Pre-prod
PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 152.228.142.152

Le nom du VPS est : vps-0d4f8f5a.vps.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :      GyNJC5QG53KZ

=================================================================================================================

Piximind-DEV
PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 51.77.141.51

Le nom du VPS est : vps-061b6980.vps.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :      GHVQmPqC5YyS

====================================================================================================================

START-Preprod
VM Details:
Host: 104.45.2.115
Username: Pomelo
MDP: PomeloStart@123

====================================================================================================================

XEFI-dev
PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 51.161.11.52
Le nom du VPS est : vps-de75100a.vps.ovh.ca
Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : ubuntu
Mot de passe :   AMxD3kQABdKq

====================================================================================================================

Hello registry ACR
deadpoolcr.azurecr.io
DeadpoolCR
yhz5i5aKWV7BVErv25p=aIQ7KCu3fU8p

====================================================================================================================
SMTP config
MAIL_MAILER=smtp
MAIL_HOST=ssl0.ovh.net
MAIL_PORT=587
MAIL_USERNAME=system@piximind.com
MAIL_PASSWORD=PIXIsmtp1aqw2zsx
MAIL_ENCRYPTION=tls

=====================================================================================================================

pixi wifi pass
PIxi202:  HiPixiMind2021

=======================================================================================================================






