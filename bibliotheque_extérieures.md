# 3.1 Utilisation de bibliothèques extérieures

## Q1 – Combien de bibliothèques extérieures ?

On a identifié 17 bibliothèques extérieures dans le projet.

Pour arriver à ce chiffre, on a parcouru les 8 fichiers pom.xml du projet. On a listé toutes les dépendances déclarées, y compris celles présentes dans dependencyManagement. Ensuite, on a supprimé les doublons en ne gardant qu'une seule occurrence par couple groupId:artifactId. L'objectif était de compter les bibliothèques réellement distinctes, sans surévaluer à cause des redondances entre modules.

## Q2 – Bibliothèques déclarées vs réellement utilisées

Après avoir listé les 17 dépendances déclarées, on a utilisé la commande :

```bash
mvn dependency:analyze
```

Cela permet d'identifier :

- les dépendances déclarées mais non utilisées,
- les dépendances utilisées mais non déclarées.

### Module principal (gson)

Dans le module principal, Maven indique :

**No dependency problems found**

Donc ce module est propre : toutes les dépendances déclarées sont utilisées, et aucune dépendance n'est utilisée sans être déclarée.

### Modules secondaires

Dans certains modules annexes, la situation est moins propre.

#### Dépendances utilisées mais non déclarées

error_prone_annotations est utilisée dans extras et proto mais pas déclarée explicitement.

Dans metrics, certaines classes de Jackson et Caliper sont utilisées sans déclaration directe (jackson-core, caliper-api).

Cela signifie que ces modules s'appuient sur des dépendances transitives (héritées indirectement), ce qui n'est pas une bonne pratique.

#### Dépendances déclarées mais non utilisées

Dans le module test-graal-native-image, Maven signale des dépendances déclarées mais non utilisées directement, comme :

- junit-jupiter
- junit-platform-launcher

Elles apparaissent dans le POM mais ne sont pas réellement utilisées dans le code.

### Verdict

Le cœur du projet est bien maintenu, mais certains modules secondaires sont plus approximatifs. Ils utilisent des dépendances transitives sans les déclarer explicitement, ou gardent des dépendances inutilisées. Cela nuit à la clarté et à la maintenabilité.

## Q3 – Que nous apprend l'analyse des bibliothèques utilisées ?

En croisant les imports présents dans le code et les résultats de mvn dependency:analyze, on observe que les bibliothèques ont des rôles bien identifiés.

### Tests – JUnit 4 et JUnit 5

JUnit 4 est très largement dominant :

- org.junit.Test apparaît plus de 100 fois.
- org.junit.Before, RunWith, Parameterized, etc. sont très présents.

JUnit 5 apparaît beaucoup plus marginalement (org.junit.jupiter.api.Test très peu utilisé).

Maven confirme :

Dans test-graal-native-image, JUnit 5 est utilisé mais certains artefacts sont déclarés sans être réellement nécessaires.

Conclusion : il y a une cohabitation JUnit 4 / JUnit 5, probablement liée à une transition partielle ou à des besoins spécifiques.

### Qualité du code – Error Prone

On trouve plusieurs imports com.google.errorprone.annotations.* comme :

- CanIgnoreReturnValue
- Keep
- InlineMe

Cela montre une volonté d'améliorer la robustesse et la qualité du code.

Cependant, Maven signale que dans certains modules, cette dépendance est utilisée sans être déclarée explicitement.

### Performance – Caliper

Dans le module metrics, on trouve des imports com.google.caliper.*.

Cela montre que le projet intègre des benchmarks pour mesurer les performances.

Mais Maven indique que caliper-api est utilisée sans être déclarée explicitement dans le module concerné.

### Interopérabilité – Protobuf, Jackson, Guava

Protobuf est utilisé dans le module proto, ce qui montre un besoin d'interopérabilité avec un format binaire.

Jackson est utilisé principalement dans metrics, probablement pour comparer les performances avec Gson.

Guava apparaît dans certains modules pour des utilitaires.

Ces bibliothèques ne sont donc pas redondantes par erreur : elles répondent à des objectifs différents.

### Y a-t-il redondance ?

#### JUnit 4 et JUnit 5

Oui, deux frameworks de test coexistent. Ce n'est pas incohérent, mais cela suggère une modernisation partielle. Maven montre qu'un ménage est possible.

#### Gson et Jackson

Oui, deux bibliothèques JSON sont présentes. Mais Jackson est utilisé dans des modules spécifiques (benchmarks), alors que Gson est le cœur du projet. Il ne s'agit donc pas d'un conflit fonctionnel.

## Conclusion globale

L'utilisation des bibliothèques est globalement cohérente avec les objectifs du projet :

- **Gson** : cœur fonctionnel
- **JUnit** : validation
- **Error Prone** : qualité
- **Caliper** : performance
- **Protobuf** : interopérabilité
- **Jackson** : comparaison / outillage

Le principal problème relevé par Maven ne concerne pas le choix des bibliothèques, mais leur déclaration incomplète ou inutile dans certains modules. Un alignement plus strict des POM améliorerait la maintenabilité.
