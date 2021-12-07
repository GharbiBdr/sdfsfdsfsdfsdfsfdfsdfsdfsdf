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
