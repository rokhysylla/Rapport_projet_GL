# 3.1 Utilisation de bibliothèques extérieures

# 3.1 Utilisation de bibliothèques extérieures

## Q1 – Utilisation de bibliothèques extérieures

On arrive à un total de 17 bibliothèques extérieures pour ce projet.

Pour obtenir ce chiffre, on a épluché les 8 fichiers pom.xml en listant toutes les dépendances (même celles planquées dans le dependencyManagement). Ensuite, on a simplement nettoyé la liste pour ne garder qu'un seul exemplaire de chaque paire groupId:artifactId, histoire de ne pas compter deux fois les mêmes.

## Q2 - Bibliothèques déclarées vs réellement utilisées

Une fois qu'on a repéré nos 17 bibliothèques dans les pom.xml, on a passé un petit coup de mvn dependency:analyze pour voir ce qui servait vraiment dans le code.

### Module principal (gson)

C'est du propre : RAS ("No dependency problems found"). Ce qui est déclaré est utilisé, et inversement. C'est carré.

### Modules secondaires

Parmi les "invités surprises" (utilisés mais pas déclarés) , on trouve error_prone_annotations dans extras et proto. Plus gênant, dans metrics, on pioche directement dans des morceaux de Jackson et Caliper (comme jackson-core ou caliper-api) sans les avoir officiellement listés.

Les "poids morts" (déclarés mais inutilisés) se trouve dans le module test-graal-native-image, on a des trucs comme junit-jupiter qui sont là sur le papier, mais dont le code ne se sert pas.

### Verdict

Si le cœur du projet est clean, certains modules annexes sont un peu plus brouillons. Ils comptent sur des dépendances qui arrivent "par ricochet" (transitives) ou gardent des déclarations inutiles. Pour la maintenance et la clarté du projet, il y aurait un petit coup de balai à donner.

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

### Performance et benchmarks (Caliper)

Imports détectés : com.google.caliper.BeforeExperiment 4 fois, com.google.caliper.Param 2 fois.

Maven (metrics) : Used undeclared com.google.caliper:caliper-api.

Donc le projet, cela sert à mesurer et surveiller des performances.

### Interopérabilité / écosystème (Protobuf + Jackson + Guava)

Protobuf : imports com.google.protobuf.GeneratedMessage 4 fois, plus des descripteurs. Le module proto existe et Maven montre l'étape protobuf:generate-test qui implique un usage réel.

Jackson : import com.fasterxml.jackson.annotation.JsonProperty apparaît (au moins 1 fois). Maven (gson-metrics) remonte Used undeclared jackson-core et jackson-annotations.

Guava : imports Splitter (4), CaseFormat (3), helpers de test,
Maven (metrics) remonte Used undeclared guava.

Ce qui signifie que :

- Gson est le cœur
- Mais il y a des modules dits "satellites" (metrics/proto/tests) qui s'appuient sur d'autres libs : Protobuf (format binaire/interop), Jackson (usage ponctuel, probablement sur metrics/outillage), Guava (utilitaires et outils de test)

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
- compatibilité formats/intéropérabilité (Protobuf)
- utilitaires/outillage (Guava)

### Point d'amélioration

Maven signale plusieurs fois Used undeclared dependencies dans les modules ou répertoires (extras, proto, metrics). Ça veut dire que le code les utilise, mais le POM n'est pas toujours aligné. Et côté tests, il y a aussi des Unused declared dependencies.

# 3.2 Organisation en paquetages

## 3.2.1 Le nombre total de paquetages Java du projet est de 25

Ce résultat a été obtenu en parcourant automatiquement tous les fichiers *.java, en extrayant les déclarations `package ...;`, puis en comptant le nombre de noms de paquetages distincts.

## 3.2.2 Analyse des liens entre les paquetages

### La démarche technique

Pour ne pas se contenter des simples imports du code source, nous avons analysé le bytecode (le fichier JAR compilé) à l'aide de l'outil jdeps. Cette méthode est plus précise car elle révèle les dépendances réelles après compilation, y compris les liens "cachés" (transitifs). Les données brutes ont ensuite été transformées en un graphe de 83 liaisons pour visualiser les interactions.

### Qui dépend de qui ?

L'analyse montre que le projet est très interconnecté. On peut classer les paquetages selon deux critères :

**Les plus "dépendants"** (ceux qui appellent beaucoup les autres) :
- Le paquetage `internal.bind` est le champion avec 18 liens sortants. Cela s'explique par son rôle : il fait le pont entre les objets Java et le JSON, il doit donc connaître presque tout le reste du projet. 
- Le cœur du projet (`com.google.gson`) suit de près avec 15 liens.

**Les plus "sollicités"** (ceux qui sont appelés par tout le monde) :
- Le paquetage racine `com.google.gson` et le paquetage `internal` sont les deux piliers centraux. Ils servent de fondations à l'ensemble de la bibliothèque.

### L'organisation en couches

Sur le papier, Gson est organisé en deux zones distinctes :

- **L'API ou partie publique** : Ce que l'utilisateur voit et utilise (ex: `com.google.gson`)
- **La partie interne** : La mécanique complexe cachée au développeur (ex: `internal.bind`)

Dans une architecture idéale (dite "stratifiée"), la vitrine ne devrait jamais dépendre de l'arrière-boutique pour rester indépendante. Ici, ce n'est pas le cas. L'API publique appelle directement des composants internes pour fonctionner. On parle d'une **architecture semi-stratifiée** : la séparation existe dans les noms de dossiers, mais pas totalement dans le fonctionnement du code.

### Le problème des cycles (allers-retours)

L'analyse révèle des dépendances cycliques. Par exemple, le paquetage `com.google.gson` a besoin de `internal.bind`, mais `internal.bind` a aussi besoin de `com.google.gson`.

**L'organisation des paquetages traduit :**

- **un couplage fort** : Cela signifie que ces deux parties sont "soudées". Si on modifie l'une, on risque fortement de casser l'autre

- **une maintenabilité fragile** : C'est plus difficile à tester ou à isoler.

Ces cycles sont bien réels, mais ils sont limités au noyau central du projet. Ce n'est pas une "toile d'araignée" ingérable, mais cela montre que l'interface publique et l'implémentation technique de Gson sont intimement liées.

## 3.2.3 Analyse de la hiérarchie des paquetages

### Quel est le nombre de niveaux de paquetages ?

Pour répondre rigoureusement, j'ai analysé toutes les déclarations `package ...;` dans les fichiers .java du projet via PowerShell.

**Résultat :**
- Nombre minimum de niveaux : 2
- Nombre maximum de niveaux : 6
- Moyenne ≈ 3,8 niveaux

Cela signifie que la hiérarchie est relativement profonde, mais reste maîtrisée. On observe une structure typique de projet Java :

```
com.google.gson.internal.bind
```

On a donc une organisation hiérarchique cohérente, avec :
- un préfixe global (`com.google`)
- un cœur métier (`gson`)
- des sous-domaines fonctionnels (`internal`, `bind`, `reflect`, etc.)

La profondeur maximale (6 niveaux) reste raisonnable et ne révèle pas de sur-structuration excessive.

### La hiérarchie des tests suit-elle celle des sources ?

Pour répondre à cette question, j'ai comparé automatiquement :
- les packages sous `src/main/java`
- les packages sous `src/test/java`

**Résultats :**
- 16 packages côté main
- 20 packages côté test
- 12 packages communs
- 8 packages spécifiques aux tests
- 4 packages du main sans équivalent test

**Interprétation :**

La majorité des packages de test suivent la hiérarchie des sources (12 correspondances exactes). Cela montre une volonté claire d'alignement structurel : les tests sont organisés selon la même logique que le code principal.

Cependant, on observe aussi :
- Des packages spécifiques aux tests comme :
    - `com.google.gson.functional`
    - `com.google.gson.integration`
    - `com.google.gson.native_test`
    - `com.google.gson.jpms_test`

Ces packages correspondent à :
- des tests d'intégration
- des tests fonctionnels
- des tests liés à des environnements spécifiques (JPMS, GraalVM)

Il ne s'agit donc pas d'un désordre architectural, mais d'une hiérarchie parallèle justifiée par la nature des tests.

**La structure des tests est donc :**
- majoritairement alignée
- mais enrichie par des sous-hiérarchies spécialisées

### Existe-t-il des paquetages ne contenant qu'un seul sous-paquetage sans classe ?

Pour répondre à cette question sans interprétation, j'ai :
- identifié tous les packages déclarés
- reconstruit les parents intermédiaires
- vérifié lesquels :
    - ne contiennent aucune classe directe
    - et n'ont qu'un seul sous-package

**Résultat : 3 cas détectés**
- `com`
- `com.google`
- `com.google.gson.extras.examples`

**Interprétation :**

Ces packages jouent un rôle de conteneur hiérarchique :
- `com` et `com.google` sont des niveaux racines implicites
- `com.google.gson.extras.examples` sert uniquement à regrouper un sous-package

Ils ne contiennent pas de classes directement.

## 3.2.4 Analyse des noms de paquetages

### Les noms révèlent-ils l'utilisation d'un design pattern (ex : MVC) ?

#### Observation

Les principaux paquetages sont :
- `com.google.gson`
- `com.google.gson.internal`
- `com.google.gson.internal.bind`
- `com.google.gson.internal.reflect`
- `com.google.gson.stream`
- `com.google.gson.annotations`

On n'observe aucun nom de type : `controller`, `model`, `view`, `service`, `repository`, `dao`.

#### Conclusion

Le projet ne suit pas une architecture MVC. Ce résultat est cohérent avec la nature du projet : Gson est une bibliothèque de sérialisation JSON, pas une application web. On observe plutôt une architecture orientée bibliothèque technique, structurée par responsabilités internes.

### Les noms révèlent-ils un lien avec une base de données ?

On ne retrouve aucun paquetage de type : `dao`, `repository`, `entity`, `persistence`, `jdbc`, `database`.

On observe uniquement : `internal.sql`, `protobuf`, `stream`, `reflect`.

#### Interprétation

Le projet n'est pas lié à une base de données. Le paquetage `internal.sql` sert uniquement à gérer la conversion JSON ↔ SQL types (par exemple Date, Timestamp), mais il n'implémente pas une couche d'accès aux données. Il n'y a donc aucune architecture orientée persistance.

### Que nous apprennent les noms des paquetages sur l'architecture ?

#### a) Présence massive du préfixe internal

Exemples : `internal`, `internal.bind`, `internal.reflect`, `internal.sql`.

Cela indique une séparation nette entre :
- l'API publique (`com.google.gson`)
- l'implémentation interne (`internal`)

C'est un très bon indicateur de maturité architecturale, correspondant à un principe fondamental : l'encapsulation forte de l'implémentation.

#### b) Présence de bind

Le paquetage `internal.bind` est l'un des plus dépendants (18 dépendances sortantes). Le terme "bind" indique un mécanisme de binding objet ↔ JSON. Cela suggère l'utilisation implicite du pattern :
- Adapter
- Éventuellement Strategy

Car le binding nécessite de choisir dynamiquement un mécanisme de conversion.

#### c) Présence de reflect

Le paquetage `internal.reflect` indique l'utilisation de la réflexion Java (`java.lang.reflect`). Cela montre que la bibliothèque :
- inspecte dynamiquement les classes
- manipule les champs et méthodes à l'exécution

Cela confirme une architecture fortement basée sur la métaprogrammation.

#### d) Présence de stream

Le paquetage `stream` indique une API orientée lecture/écriture séquentielle. Cela suggère une séparation entre :
- API objet
- API flux bas niveau

On retrouve ici une organisation en couches techniques internes.

#### e) Présence d'annotations

La présence d'un paquetage `annotations` indique :
- une API orientée configuration déclarative
- utilisation d'annotations personnalisées (ex: `@Expose`, `@SerializedName`)

Cela renforce l'idée d'un design orienté extensibilité.

### Synthèse globale

Les noms des paquetages nous apprennent que :

- Le projet n'est pas structuré selon MVC.
- Il n'a aucun lien architectural avec une base de données.
- Il adopte une architecture de bibliothèque technique.
- Il distingue clairement API publique et implémentation interne.
- Il repose fortement sur :
    - la réflexion
    - le binding dynamique
    - les annotations
    - une organisation en sous-domaines fonctionnels

On observe une architecture modulaire, orientée responsabilité technique, et conforme aux bonnes pratiques des bibliothèques Java.

## 3.3 Répartition des classes dans les paquetages

### Approche 1 : Comptage par paquetage (via script PowerShell)

J'ai extrait les fichiers .java (hors package-info.java) et regroupé les classes selon leur déclaration `package ...;`.

Ce comptage correspond aux fichiers Java présents dans chaque paquetage. En Java, un fichier peut contenir plusieurs déclarations de classes (classes internes, etc.), donc ce résultat correspond à une approximation par fichier.

**Résultats obtenus :**

- Nombre total de classes (fichiers .java) : 27
- Nombre de paquetages concernés : 5
- Minimum (classes / paquetage) : 1
- Maximum (classes / paquetage) : 18
- Moyenne (classes / paquetage) : 5,40

On observe une forte concentration dans le paquetage `com.example` (18 classes), tandis que les autres paquetages restent plus légers.

### Approche 2 : Nombre total de classes (vision globale projet)

Pour avoir une vue plus large, j'ai également utilisé un script comptant l'ensemble des classes du projet multi-modules (sources + tests).

**Résultat :**

- Total classes détectées (main + tests) : 255
- SonarQube indique : 185 classes au niveau global du projet

La différence s'explique par le fait que :
- SonarQube applique ses propres règles d'inclusion/exclusion (modules, configuration, analyse AST)
- Le script PowerShell inclut explicitement toutes les classes src/main et src/test
- Les deux chiffres ne sont donc pas contradictoires : ils correspondent à des périmètres différents

### Conclusion

La répartition interne par paquetage montre :
- une concentration importante des classes dans un paquetage principal
- plusieurs paquetages spécialisés contenant peu de classes
- une organisation relativement centralisée

Le volume global du projet (entre 185 et 255 classes selon le périmètre) confirme qu'il s'agit d'un projet de taille significative, structuré en plusieurs modules.

### Analyse de la répartition des classes dans les paquetages

#### 1) Toutes les classes (ou presque) sont‑elles dans le même paquetage ?

D'après le comptage que j'ai réalisé par paquetage (à partir des fichiers .java et de leur déclaration package), j'observe une forte concentration des classes dans un paquetage dominant.

Dans le périmètre mesuré :

Maximum : 18 classes dans un même paquetage  
Total mesuré : 27 classes réparties sur 5 paquetages

Cela signifie qu'environ les deux tiers des classes se trouvent dans un seul paquetage. Je ne peux donc pas dire que toutes les classes sont dans le même paquetage, mais la structure est clairement très centralisée.

Je précise cependant que ce comptage correspond aux fichiers .java analysés dans ce périmètre, et qu'il s'agit donc d'une mesure structurelle par fichier.

#### 2) Des paquetages non‑feuilles contiennent‑ils des classes ?

Grâce à l'analyse de la hiérarchie, j'ai identifié plusieurs paquetages parents (par exemple `com`, `com.google`, etc.) qui ne contiennent aucune classe directement, mais uniquement des sous‑paquetages.

Cela montre qu'il existe des paquetages purement structurels, utilisés pour organiser l'arborescence.

Cependant, j'observe aussi que certains paquetages qui ont des sous‑paquetages (donc non‑feuilles) contiennent malgré tout des classes.  
La présence de classes ne dépend donc pas uniquement du fait qu'un paquetage soit feuille ou non.

#### 3) En cas de hiérarchies parallèles, les paquetages les plus chargés sont‑ils les mêmes ?

J'ai comparé les hiérarchies `src/main/java` et `src/test/java`.  
Les résultats montrent :

- 16 paquetages en main  
- 20 paquetages en test  
- 12 communs  
- 8 spécifiques aux tests  
- 4 spécifiques au main

Cela confirme l'existence de hiérarchies partiellement parallèles.

Je ne peux pas affirmer strictement, avec les mesures actuelles, que les paquetages les plus chargés en production sont aussi les plus chargés en tests. En effet, certains paquetages sont spécifiques aux tests (ex. `functional`, `integration`, `native_test`), et peuvent donc concentrer des classes qui n'existent pas en production.

Pour le démontrer quantitativement, il faudrait réaliser un comptage séparé « classes par paquetage » pour main et test, puis comparer les maxima.

### Conclusion

D'après mon analyse :

- La répartition des classes est fortement centralisée autour d'un paquetage principal.
- Certains paquetages servent uniquement à structurer l'arborescence et ne contiennent pas de classes.
- Les tests introduisent une hiérarchie partiellement parallèle, ce qui peut modifier la répartition relative des classes entre production et tests.

J'ai donc répondu en m'appuyant sur les résultats obtenus, tout en précisant les limites méthodologiques de mon comptage.

## 3.3.1 Analyse du couplage et de la cohésion de quelques paquetages clés

Pour analyser le couplage et la cohésion, je me suis concentré sur trois paquetages centraux du projet :

- `com.google.gson`
- `com.google.gson.internal`
- `com.google.gson.internal.bind`

Je me base sur l'analyse des dépendances obtenue via **jdeps**, qui m'a permis d'identifier les dépendances sortantes (outgoing) et entrantes (incoming) entre paquetages.  
Au total, j'ai identifié **83 dépendances inter‑paquetages**.

### Analyse du couplage

#### com.google.gson.internal.bind

- Dépendances sortantes : 18  
- Dépendances entrantes : 4

Il s'agit du paquetage le plus dépendant du projet. Il utilise un grand nombre d'autres paquetages (`internal`, `reflect`, `stream`, …), ce qui indique un fort couplage sortant. En revanche, relativement peu de paquetages dépendent de lui (incoming = 4).  
`internal.bind` joue un rôle d'orchestrateur interne, très connecté au reste du moteur, sans être un point d'entrée majeur pour d'autres composants : couplage élevé mais cohérent avec son rôle technique central.

#### com.google.gson.internal

- Sortantes : 16  
- Entrantes : 5

Ce paquetage présente également un couplage élevé, à la fois sortant et entrant. Il dépend de nombreux autres paquetages et est lui‑même utilisé par plusieurs composants, ce qui confirme qu'il constitue un noyau interne du système. Le risque serait qu'il devienne un « fourre‑tout », mais le couplage observé semble structurel plutôt qu'anarchique.

#### com.google.gson

- Sortantes : 15  
- Entrantes : 5

Paquetage de l'API publique. Son couplage sortant reste important (15), logique : l'API publique s'appuie sur les mécanismes internes. Son couplage entrant (5) montre qu'il est aussi utilisé par d'autres composants internes. Le couplage est modéré à élevé, cohérent avec son rôle central.

### Analyse de la cohésion

Je juge la cohésion par la responsabilité fonctionnelle des classes.

#### com.google.gson

Regroupe les classes de l'API publique (`JsonReader`, `JsonWriter`, …). Toutes visent la manipulation et la sérialisation JSON : bonne cohésion fonctionnelle.

#### com.google.gson.internal

Contient les mécanismes internes (gestion des types, réflexion, utilitaires). Malgré de nombreuses dépendances, les classes restent centrées sur le support interne de la sérialisation : cohésion correcte, plus technique que fonctionnelle.

#### com.google.gson.internal.bind

Regroupe les mécanismes de binding (adaptateurs, liaison Java ↔ JSON). Son fort couplage sortant s'explique par son rôle d'interconnexion entre sous‑systèmes. Les classes partagent une responsabilité commune (mécanisme d'adaptation) : cohésion satisfaisante malgré le couplage.

### Conclusion générale

À partir des 83 dépendances identifiées :

- `internal.bind` est le paquetage le plus couplé (sortant).
- `internal` et `gson` forment le noyau central.
- Le couplage est relativement élevé, mais cohérent avec une architecture en couches internes.
- La cohésion des trois paquetages étudiés est bonne : chacun a une responsabilité identifiable (API publique, mécanismes internes, binding).

L'architecture apparaît donc centralisée autour d'un noyau interne fort, sans désorganisation évidente dans les responsabilités des paquetages analysés.

# 3.4 Organisation des classes

Pour analyser l'organisation des classes, je me suis appuyé sur les classes compilées du projet (12 répertoires target/classes et target/test-classes), soit 264 classes uniques.

## Hiérarchie

Les indicateurs DIT/NOC montrent une hiérarchie globalement plate :

DIT min = 0, max = 2, moyenne = 0,08

NOC min = 1, max = 200, moyenne = 15,87

La profondeur d'héritage est donc faible : la plupart des classes n'utilisent pas de chaînes d'héritage longues. En revanche, l'existence d'un NOC maximal élevé indique qu'au moins une classe joue un rôle très central avec de nombreux descendants.

## Stabilité (couplage)

L'analyse jdeps du jar gson met en évidence 2414 dépendances entre classes.
Le cœur du projet dépend fortement des packages standards Java (java.lang, java.util, java.io, réflexion), ce qui est cohérent pour une bibliothèque de sérialisation. Le volume important de dépendances montre un couplage dense, normal pour une librairie centrale, mais implique que certaines classes sont particulièrement structurantes.

## Cohésion

J'ai mesuré la cohésion interne de trois paquetages représentatifs :

- com.google.gson : cohésion = 0,422
- com.google.gson.internal : cohésion = 0,20
- com.google.gson.internal.bind : cohésion = 0,171

Le paquetage principal est le plus cohésif, tandis que les packages internes dépendent davantage de composants externes (TypeAdapter, JsonReader/Writer, etc.), ce qui correspond à leur rôle d'implémentation et d'adaptation.

## Conclusion

L'architecture montre une hiérarchie peu profonde, un noyau fortement connecté, et des couches internes moins cohésives mais spécialisées. L'organisation globale est cohérente avec le rôle d'une bibliothèque de sérialisation modulaire.

