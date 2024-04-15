# Petit Résumé des actions GitHub que j'ai pu connaitre

## Environnement de recette pour la construction d'application

### Déclencheurs d'exécution :

- Pull Request
- Déclenchement manuel

### Gestion de la concurrence :

- Groupe de concurrence : ${{ github.workflow }}-${{ github.ref }}
- Annulation en cours : activée

### Jobs :

#### Test :

- Système d'exploitation : Ubuntu (dernière version)

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

- Besoin du job "Test"
- Système d'exploitation : Ubuntu (dernière version)

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