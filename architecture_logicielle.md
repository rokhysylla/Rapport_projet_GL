Pour fournir cette analyse , je me suis appuyé sur la commande 
mvn dependency:analyze -DskiptTests,  lancée depuis la racine du projet, qui met en évidence les dépendances réellement utilisées dans le code.
mvn dependency: tree sert à afficher l'arbre des dépendances du projet. Une capture d'écran nommée dependency.png se trouve dans le répertoire images illustrant cela.

# 3.1 Utilisation de bibliothèques extérieures

## Q1 – Utilisation de bibliothèques extérieures

On arrive à un total de 17 bibliothèques extérieures pour ce projet.

Pour obtenir ce chiffre, on a épluché les 8 fichiers pom.xml en listant toutes les dépendances (même celles planquées dans le dependencyManagement). Ensuite, on a simplement nettoyé la liste pour ne garder qu'un seul exemplaire de chaque paire groupId:artifactId, histoire de ne pas compter deux fois les mêmes.

## Q2 - Bibliothèques déclarées vs réellement utilisées

Une fois qu'on a repéré nos 17 bibliothèques dans les pom.xml, on a passé un petit coup de mvn dependency:analyze pour voir ce qui servait vraiment dans le code.

### Module principal (gson)

C'est du propre : RAS ("No dependency problems found"). Ce qui est déclaré est utilisé, et inversement. C'est carré.

### Modules secondaires

Les "poids morts" (déclarés mais inutilisés) se trouve dans le module test-graal-native-image, on a des trucs comme junit-jupiter qui sont là, mais dont le code ne se sert pas.

### Verdict

 Le cœur du projet est clean, certains modules annexes sont un peu plus brouillons. Ils comptent sur des bibliothèques   ou gardent des déclarations inutiles. Pour la maintenance et la clarté du projet, il y aurait un petit coup de balai à donner.

## Q3 - Analyser les bibliothèques réellement utilisées

Que nous apprend l'utilisation de ces bibliothèques ?

En croisant les imports réellement présents dans le code avec des commandes depuis PowerShell et les diagnostics Maven (mvn dependency:analyze), on voit que ces bibliothèques ne sont pas là "pour décorer le POM" : elles correspondent à des besoins clairs du projet.

### Validation et robustesse (tests)

JUnit 4 est ultra-dominant : `import org.junit.Test;` apparaît 130 fois, `import org.junit.Before;` 55 fois, auxquels s'ajoutent `RunWith`, `Parameterized`, etc. Cela reflète un projet avec une vaste base de tests existants.

JUnit 5 apparaît, mais marginalement : `import org.junit.jupiter.api.Test;` n'apparaît que 2 fois.

Côté Maven, on voit précisément où ça coince : dans test-graal-native-image, Maven détecte :

- Used undeclared : org.junit.jupiter:junit-jupiter-api (donc JUnit 5 est réellement utilisé dans ce module)
- Unused declared : org.junit.jupiter:junit-jupiter et org.junit.platform:junit-platform-launcher (déclarés mais pas utilisées directement)

Ce que ça signifie : les tests sont très présents, avec une cohabitation de JUnit4 et JUnit5 qui ressemble à une transition partielle ou à un module de test spécifique (GraalVM).

### Qualité du code et prévention d'erreurs (Error Prone)

Dans le code, il y a des imports com.google.errorprone.annotations.* bien visibles :
CanIgnoreReturnValue 14 fois dans plusieurs fichiers différents, Keep 3, InlineMe 2…

Maven confirme et pointe un souci "administratif" :

- extras : Used undeclared error_prone_annotations
- proto : Used undeclared error_prone_annotations

Cette bibliothèque error_prone sert normalement d'annotations pour rendre le code plus sûr et plus maintenable, mais certains modules oublient de déclarer proprement la dépendance.

### Performance (Caliper)

Imports détectés : com.google.caliper.BeforeExperiment 4 fois, com.google.caliper.Param 2 fois dans le module metrics.

Maven (metrics) : Used undeclared com.google.caliper:caliper-api.
Cette partie sert à mesurer et surveiller des performances.

### écosystème (Protobuf + Jackson + Guava)

Protobuf : imports com.google.protobuf.GeneratedMessage référencées 4 fois, plus des descripteurs. Le module proto existe et Maven montre l'étape protobuf:generate-test qui implique un usage réel.

Jackson : import com.fasterxml.jackson.annotation.JsonProperty apparaît (au moins 1 fois). Maven (gson-metrics) remonte Used undeclared jackson-core et jackson-annotations.

Guava : imports Splitter (4), CaseFormat (3), helpers de test,
Maven (metrics) remonte Used undeclared guava.

Ce qui signifie que :

- Gson est le cœur
- Mais il y a des modules comme  (metrics/proto/tests) qui s'appuient sur d'autres libs : Protobuf (format binaire), Jackson (usage ponctuel, sur metrics/outillage), Guava (utilitaires et outils de test)

## Plusieurs bibliothèques pour la même chose ?

Oui, on observe deux zones de redondance, mais elles ne sont pas forcément "mauvaises".

### Plusieurs bibliothèques de tests : JUnit 4 et JUnit 5

- Énorme majorité JUnit 4, un peu de JUnit 5
- Maven : le module test-graal-native-image déclenche des alertes liées à JUnit 5 (API utilisée, mais d'autres artefacts déclarés inutilement)
- Donc oui, deux frameworks cohabitent : JUnit 4 (héritage massif) + JUnit 5 (cas spécifiques / modernisation)

### Plusieurs bibliothèques JSON : Gson et Jackson

- Gson est très présente dans le code
- Maven confirme que Jackson est réellement utilisé (au moins dans metrics via composants)
- Donc oui, deux libs JSON existent dans le repo, mais Jackson semble être utilisé en périphérique, pas concurrent du core Gson.

## Est-ce justifié ?

Globalement oui, mais avec une nuance importante : c'est justifié fonctionnellement, mais ce n'est pas toujours proprement déclaré dans les POM.

### JUnit4 + JUnit5

Justifié, mais Maven montre du ménage possible : junit-jupiter et junit-platform-launcher sont déclarés mais non utilisés.

### Gson + Jackson

Sont justifiés car Jackson sert pour des besoins spécifiques (outils, métriques, sérialisation annexe) et que Gson reste l'API principale. Ici, c'est cohérent : Jackson apparaît surtout côté metrics, pas partout.

### Error Prone / Caliper / Protobuf / Guava

Justifiés car ils correspondent à des objectifs clairs :

- fiabilité et maintenabilité (Error Prone)
- performance (Caliper)
- compatibilité formats (Protobuf)
- utilitaires/outillage (Guava)

### Point d'amélioration

Maven signale plusieurs fois Used undeclared dependencies dans les modules ou répertoires (extras, proto, metrics). Ça veut dire que le code les utilise, mais le POM n'est pas toujours aligné. Et côté tests, il y a aussi des Unused declared dependencies.




