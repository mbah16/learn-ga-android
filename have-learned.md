## Environnement de recette pour la construction d'application

### Déclencheurs d'exécution :

Les actions sont déclenchées à chaque Pull Request ou peuvent être déclenchées manuellement via un déclenchement workflow_dispatch.

### Gestion de la concurrence :

Pour garantir une exécution fluide et éviter les conflits, les actions sont regroupées en fonction du nom du workflow et de la référence Git. De plus, elles ont la capacité d'annuler les actions en cours si nécessaire pour maintenir la cohérence.

### Jobs :

#### Test :

Ce job vérifie si la Pull Request a été approuvée par un ou plusieurs examinateurs. Si elle est approuvée, le processus continue ; sinon, il échoue avec un message approprié. Ensuite, il procède à la récupération du code à partir du référentiel, à la configuration de l'environnement Java nécessaire et à la construction de l'APK de test en mode débogage.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Vérification de l'approbation
        run: |
          PR_NUMBER=$(echo "${{ github.event.pull_request.url }}" | awk -F'/' '{print $NF}')
          REVIEWERS=$(curl -s -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" "https://api.github.com/repos/${{ github.repository }}/pulls/${PR_NUMBER}/reviews")
          
          if echo "$REVIEWERS" | jq '.[] | select(.state == "APPROVED")' > /dev/null; then
            echo "La pull request a été approuvée."
          else
            echo "La pull request n'a pas été approuvée."
            exit 1
          fi

      - name: Récupération du code
        uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        name: Configuration de Java 21
        with:
          distribution: 'corretto'
          java-version: '21'

      - name: Construction de l'APK de test en mode débogage
        run: bash ./gradlew test
```

#### Construction :

Ce job dépend du job "Test" pour s'assurer que les tests ont réussi. Il procède ensuite à la récupération du code, à la configuration de l'environnement Java et à la construction du fichier APK final.

```yaml
jobs:
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Récupération du code
        uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        name: Configuration de Java 21
        with:
          distribution: 'corretto'
          java-version: '21'

      - name: Construction du fichier APK
        run: bash ./gradlew build
```